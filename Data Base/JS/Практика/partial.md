Напишем функцию `partial(fn, ...presetArgs)`. Это реализация паттерна «частичное применение». Она делает то же самое, что и `bind` в плане аргументов, но **без** жесткой привязки контекста `this` (контекст должен пробрасываться естественным образом). Функция просто запоминает часть аргументов, а остальные принимает позже.

**Пример:**

```JavaScript
function add3(a, b, c) {
  return a + b + c;
}

const add5 = partial(add3, 2, 3); // заранее передали 2 и 3
console.log(add5(4)); // 9 (то есть 2 + 3 + 4)
```

# Ответ
```js
function partial(fn, ...PortalArgs) {
  return function (...args) {
    return fn.apply(this, [...PortalArgs, ...args]);
  };
}
```