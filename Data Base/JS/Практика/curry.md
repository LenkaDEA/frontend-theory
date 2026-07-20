```js
function curry(fn) {
  return function curried(...args) {
    // Если аргументов собрано достаточно, вызываем оригинал
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      // Иначе возвращаем новую функцию, которая дособирает аргументы
      return function(...args2) {
        return curried.apply(this, [...args, ...args2]);
      }
    }
  };
}
```