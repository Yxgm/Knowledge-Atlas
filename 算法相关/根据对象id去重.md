```js
function foo(array, key) {
  let res = [];
  let list = [];
  array.forEach((item) => {
    res.forEach((item1) => {
      list.push(item1[key]);
    });
    if (!list.includes(item[key])) {
      res.push(item);
    }
  });
  return res;
}


// js数组对象根据id去重;
const removeDuplicationById = (target, key) => {
  let result = {};
  return target.reduce((arr, item) => {
    item[key] in result ? '' : (result[item[key]] = true) && arr.push(item);
    return arr;
  }, []);
};

// const removeDuplicationById = (target, key) => {
//   const result = target.reduce((arr, item) => {
//     arr.has(item[key]) ? '' : arr.set(item[key], item);
//     return arr;
//   }, new Map());
//   return result;
//   return [...result.values()];
// };

var arr = [
  {
    key: '01',
    value: '乐乐'
  },
  {
    key: '02',
    value: '博博'
  },
  {
    key: '03',
    value: '淘淘'
  },
  {
    key: '04',
    value: '哈哈'
  },
  {
    key: '01',
    value: '乐乐'
  }
];

const res = foo(arr, 'key');
console.log(res);

```

