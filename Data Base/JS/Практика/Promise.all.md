```js
function myPromiseAll(promises) {
  if (promises.length === 0) return Promise.resolve([]);

  return new Promise((resolve, reject) => {
    let results = [];
    let counter = 0;

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then((res) => {
          results[index] = res;
          counter = counter + 1;
          if (counter === promises.length) {
            resolve(results);
          }
        })
        .catch((err) => reject(err));
    });
  });
}
```