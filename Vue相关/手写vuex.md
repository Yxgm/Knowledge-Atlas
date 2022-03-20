```js
let Vue = require('./vue');

Vue.prototype.$store = new Store();
//一句话总结：就是把this当成闭包环境变量，在commit或者dispatch中使用。
function Store(option = {}) {
  let store = this; //保存实例

  this.state = new Vue({
    data() {
      return { ...option.state };
    }
  });

  this.mutationsMap = Object.create(null);
  this.actionsMap = Object.create(null);

  Object.keys(option.mutations).forEach(
    (fnName) => (this.mutationsMap[fnName] = option.mutations[fnName])
  );

  Object.keys(option.actions).forEach(
    (fnName) => (this.actionsMap[fnName] = option.actions[fnName])
  );

  const commit = this.commit;
  this.commit = (fnName, payload) => commit(store, fnName, payload);

  const dispatch = this.dispatch;
  this.dispatch = (fnName, payload) => dispatch(store, fnName, payload);
}

Store.prototype.commit = function (thisArg, fnName, payload) {
  let fn = store.mutationsMap[fnName];
  fn.call(thisArg, store.state, payload);
};

Store.prototype.dispatch = function (thisArg, fnName, payload) {
  let fn = store.actionsMap[fnName];
  fn.call(thisArg, store, payload);
};

const store = new Store({
  state: {
    count: 0
  },
  mutations: {
    setNumber(state, number) {
      state.count = number;
    }
  },
  actions: {
    async asyncSetNumber({ commit, dispatch }, payload) {
      const result = await new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve(100);
        }, 500);
      });
      commit('setNumber', result);
    }
  }
});

// store.commit('setNumber', 10);

store.dispatch('asyncSetNumber', '参数');

setTimeout(() => {
  console.log(store.state.count);
}, 2000);

modules.Vue = Vue;

```

