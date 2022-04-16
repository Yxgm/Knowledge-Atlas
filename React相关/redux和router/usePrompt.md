```ts
 //要实现劫持跳转的page页面
import { usePrompt } from "../../hooks/usePrompt";
import DialogBox from "../../components/Dailog";
import { Auth } from "../../auth";

export default function Account() {
  const [showDialog, setShowDialog] = useState<boolean>(false)
  const [showPrompt, confirmNavigation, cancelNavigation] =  usePrompt(showDialog)
 
  const handleChange = (e) => setShowDialog(true)//有输入的情况下，将showDialog置true，但并不是展示dialog
  
  return (
    <Auth>
      <div className="Account">
          <DialogBox
            showDialog={showPrompt}
            confirmNavigation={confirmNavigation}
            cancelNavigation={cancelNavigation}
          />
        <input onChange={handleChange}/>
      </div>
    </Auth>
  );
}

//钩子实现
import {UNSAFE_NavigationContext,useLocation,useNavigate} from 'react-router';
import type { History, Transition } from 'history';

export function usePrompt(when: boolean): (boolean | (() => void))[] {
  const navigate = useNavigate();
  const location = useLocation();
  const [showPrompt, setShowPrompt] = useState(false);
  const [lastLocation, setLastLocation] = useState<any>(null);
  const [confirmedNavigation, setConfirmedNavigation] = useState(false);

  const cancelNavigation = useCallback(() => {
    setShowPrompt(false);
  }, []);

  const handleBlockedNavigation = useCallback(
    (nextLocation) => {
      if (!confirmedNavigation &&nextLocation.location.pathname !== location.pathname) {
        setShowPrompt(true); //当跳走的时候，展示dialog
        setLastLocation(nextLocation);//存储下一次跳转的path
      }
    },
    [confirmedNavigation]
  );

  const confirmNavigation = useCallback(() => {
    setShowPrompt(false); //确认跳转后，dialog关闭
    setConfirmedNavigation(true);//触发确认跳转函数
  }, []);
//监听是否触发跳转函数，变true的时候，navigate跳转页面
  useEffect(() => {
    if (confirmedNavigation && lastLocation) {
      navigate(lastLocation.location.pathname);
    }
  }, [confirmedNavigation, lastLocation]);

  const navigator = useContext(UNSAFE_NavigationContext).navigator as History;

  useEffect(() => {
    if (!when) return;
    const unblock = navigator.block((tx: Transition) => {
      const autoUnblockingTx = {
        ...tx,
        retry() {
          unblock();
          tx.retry();
        }
      };
      handleBlockedNavigation(autoUnblockingTx);
    });

    return unblock;
  }, [navigator, handleBlockedNavigation, when]);

  return [showPrompt, confirmNavigation, cancelNavigation];
}

```

