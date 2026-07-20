#### 1. Определение
В контексте Service Worker кэширование — это процесс перехвата сетевых запросов (событие `fetch`) и принятия решения: откуда отдать ответ (из сети, из локального Cache API или скомбинировать эти подходы).

Существует **5 классических стратегий кэширования**. Разные типы данных требуют разных стратегий (например, для CSS/JS мы хотим кэш, для API — свежие данные).

#### 2. Как это работает в браузере (Cache API)
В браузере есть специальное хранилище — **Cache Storage API**.
*   Оно доступно как из окна (`window.caches`), так и из воркера (`self.caches`).
*   В нем хранятся пары: **Запрос (Request)** -> **Ответ (Response)**.
*   Важно: это *не* HTTP-кэш браузера. Cache API полностью управляется программно через JS. Мы сами решаем, что туда положить и когда оттуда удалить.

#### 3. 5 Классических стратегий кэширования (и их реализация)

Давай разберем их все внутри события `fetch`.

```typescript
// Вспомогательный код внутри sw.ts
sw.addEventListener('fetch', (event) => {
  // event.respondWith() - это способ перехватить запрос 
  // и подсунуть браузеру свой ответ (Response)
  event.respondWith(
    // Сюда мы передаем Promise, который вернет Response,
    // используя одну из стратегий ниже.
  );
});
```

##### Стратегия 1: Cache Only (Только Кэш)
Мы всегда ищем ответ только в кэше. Если там его нет — запрос падает.
*   **Где использовать:** Для App Shell (критических файлов, которые мы уже закэшировали на этапе `install` - логотипы, шрифты).
```typescript
async function cacheOnly(request: Request): Promise<Response> {
  const response = await caches.match(request);
  if (response) return response;
  // Если нет в кэше - отдаем ошибку
  return new Response('File not found in cache', { status: 404 }); 
}
```

##### Стратегия 2: Network Only (Только Сеть)
SW просто пропускает запрос в сеть. Мы вообще не трогаем кэш.
*   **Где использовать:** POST/PUT/DELETE запросы, аналитика (Google Analytics), запросы, которые нельзя кэшировать по соображениям безопасности.
```typescript
async function networkOnly(request: Request): Promise<Response> {
  // Просто проксируем запрос
  return fetch(request); 
}
```

##### Стратегия 3: Cache First, falling back to Network (Сначала Кэш, потом Сеть)
Ищем в кэше. Если есть — мгновенно отдаем. Если нет — идем в сеть, возвращаем ответ юзеру и **заодно сохраняем копию в кэш** на будущее.
*   **Где использовать:** Картинки, аватарки, статические ассеты (CSS, JS) — всё, что редко меняется и грузится динамически.
```typescript
async function cacheFirst(request: Request): Promise<Response> {
  // 1. Ищем в кэше
  const cachedResponse = await caches.match(request);
  if (cachedResponse) return cachedResponse;

  // 2. Если нет в кэше - идем в сеть
  try {
    const networkResponse = await fetch(request);
    
    // 3. Важно: Ответы можно прочитать только один раз (поток). 
    // Поэтому мы клонируем ответ: оригинал отдаем браузеру, клон - в кэш.
    const cache = await caches.open('dynamic-cache');
    cache.put(request, networkResponse.clone());
    
    return networkResponse;
  } catch (error) {
    // Сеть недоступна и в кэше нет
    return new Response('Offline', { status: 503 });
  }
}
```

##### Стратегия 4: Network First, falling back to Cache (Сначала Сеть, потом Кэш)
Пытаемся получить самые свежие данные из сети. Если успешно — сохраняем/обновляем в кэше и отдаем юзеру. Если сеть упала (оффлайн) — достаем из кэша.
*   **Где использовать:** API-запросы, ленты новостей, статьи, список сообщений. Данные должны быть свежими, но оффлайн важнее белого экрана.
```typescript
async function networkFirst(request: Request): Promise<Response> {
  try {
    // 1. Пробуем сходить в сеть
    const networkResponse = await fetch(request);
    
    // 2. Если успех - обновляем кэш
    const cache = await caches.open('api-cache');
    cache.put(request, networkResponse.clone());
    
    return networkResponse;
  } catch (error) {
    // 3. Сеть упала - достаем из кэша
    const cachedResponse = await caches.match(request);
    if (cachedResponse) return cachedResponse;
    
    throw error;
  }
}
```

##### Стратегия 5: Stale-While-Revalidate (Устаревшее, пока обновляется)
Сложная, но **самая популярная стратегия для PWA**. Мы мгновенно отдаем данные из кэша (если есть), чтобы юзер не ждал. И *одновременно* в фоне делаем запрос в сеть. Когда сеть ответит — мы тихо обновляем кэш свежими данными. При следующем заходе юзер увидит эти свежие данные.
*   **Где использовать:** Аватарки юзеров, некритичные дашборды, ресурсы, где скорость важнее 100% актуальности в данную секунду.
```typescript
async function staleWhileRevalidate(request: Request): Promise<Response> {
  const cache = await caches.open('swr-cache');
  const cachedResponse = await caches.match(request);

  // Делаем сетевой запрос в фоне
  const networkFetch = fetch(request).then(response => {
    cache.put(request, response.clone());
    return response;
  });

  // Отдаем кэш сразу, либо ждем сеть, если кэша нет
  return cachedResponse || networkFetch;
}
```

#### 4. Как это выглядит в React / Vue проектах (Использование Workbox)

Писать весь этот код руками (с обработкой ошибок, клонированием респонсов, очисткой старых кэшей) — очень муторно и чревато багами. В 99% коммерческой разработки (как на React, так и на Vue) используется библиотека **Workbox** (от Google). `vite-plugin-pwa` использует её под капотом.

В конфигурации `vite.config.ts` мы просто указываем пути (RegEx) и стратегию:

```typescript
// vite.config.ts (React или Vue)
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  plugins: [
    VitePWA({
      // Стратегии для runtime (когда приложение уже работает)
      workbox: {
        runtimeCaching: [
          {
            // Для картинок юзаем Cache First
            urlPattern: /\.(?:png|jpg|jpeg|svg)$/,
            handler: 'CacheFirst',
            options: {
              cacheName: 'images-cache',
              expiration: {
                maxEntries: 50, // Храним не больше 50 картинок (чистится само!)
                maxAgeSeconds: 30 * 24 * 60 * 60, // 30 дней
              },
            },
          },
          {
            // Для API-запросов юзаем Network First
            urlPattern: /^https:\/\/api\.mysite\.com\/.*/i,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'api-cache',
              networkTimeoutSeconds: 5, // Если сервер тупит 5 сек - отдаем кэш
            },
          }
        ]
      }
    })
  ]
})
```

#### 5. Тонкие моменты и нюансы (Где ошибаются новички)

1.  **Response.clone() — Попытка прочитать ответ дважды:**
    *   *Ошибка:* Вызвать `cache.put(req, response)`, а потом вернуть этот же `return response;`.
    *   *Нюанс:* Объект `Response` в JS — это поток (stream). Его можно прочитать только один раз. Если ты положил его в кэш, он "считан". Если вернуть его браузеру — будет ошибка потока. *Правило:* Всегда делай `response.clone()` при сохранении в кэш.
2.  **Забивание памяти (Кэш бесконечен):**
    *   *Ошибка:* Кэшировать все подряд в `Cache First` безлимитно.
    *   *Нюанс:* Кэш не резиновый (лимит зависит от браузера и свободного места на диске пользователя, часто ~50MB - 1GB). Если писать свой код, надо руками писать логику удаления старых ключей. *Решение:* Использовать Workbox, который сам умеет делать `maxEntries` (оставлять только X последних файлов).
3.  **Кэширование POST запросов:**
    *   *Ошибка:* Пытаться положить в кэш запрос типа POST/PUT/DELETE.
    *   *Нюанс:* `Cache API` поддерживает сохранение только `GET` запросов (или HEAD). Если попытаться закэшировать POST запрос (например, отправку формы), API выкинет ошибку.
4.  **Разные запросы, один URL (CORS и Query-параметры):**
    *   *Нюанс:* По умолчанию кэш различает запросы строго по полному URL (включая query параметры `?user=1`). Если нужно, чтобы в кэше `?user=1` и `?user=2` считались одним ресурсом (например, если это просто токен отслеживания), при поиске в кэше нужно использовать опции: `caches.match(request, { ignoreSearch: true })`.