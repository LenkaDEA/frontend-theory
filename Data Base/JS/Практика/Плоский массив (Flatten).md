Напиши функцию `flatten(arr)`, которая принимает массив с любым уровнем вложенности и возвращает плоский массив (аналог `Array.prototype.flat(Infinity)`).

# Ответ
```js
function flatten(arr) {
  let result = [];
  if (arr.length === 0) return result;

  arr.forEach((item) => {
    if (Array.isArray(item)) {
      result.push(...flatten(item));
    } else {
      result.push(item);
    }
  });

  return result;
}

console.log(flatten([1, [2, [3, 4], 5], 6]));

```