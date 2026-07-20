Реализуй функцию `deepClone(obj)`, которая создает полную копию переданного объекта, включая вложенные объекты и массивы.
# Ответ

```js
function deepClone(obj) {
  if (typeof obj !== "object" || obj === null) return obj;

  let newObj;
  if (Array.isArray(obj)) {
    newObj = [];
  } else {
    newObj = {};
  }

  for (let [key, value] of Object.entries(obj)) {
    if (typeof value === "object" && value !== null) {
      newObj[key] = deepClone(value);
    } else {
      newObj[key] = value;
    }
  }

  return newObj;
}

```