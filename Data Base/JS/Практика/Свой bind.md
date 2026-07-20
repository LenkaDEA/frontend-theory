Напиши полифил для метода `bind` — `Function.prototype.myBind`. Функция должна уметь привязывать контекст и частично применять аргументы.

# Ответ
```js 
Function.prototype.myBind = function (context, ...bindArgs) {
  let fn = this;
  return function (...args) {
    return fn.apply(context, [...bindArgs, ...args]);
  };
};

```