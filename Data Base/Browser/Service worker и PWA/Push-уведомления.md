#### 1. Определение
Пуш-уведомления в PWA строятся на комбинации двух разных браузерных API:
*   **Push API:** Отвечает за *доставку* данных от твоего бэкенда в браузер пользователя. Даже если сайт закрыт, браузер получает сигнал и «будит» твой Service Worker.
*   **Notifications API:** Отвечает за *отображение* самого системного окошка с уведомлением (заголовок, текст, иконка, кнопки) на экране пользователя.

#### 2. Как это работает в браузере (Архитектура)

Система работает как треугольник: Твой Бэкенд ↔ Пуш-сервис браузера (FCM/Mozilla Push Service) ↔ Браузер пользователя.

**Шаг 1. Подписка (В React/Vue приложении - Main Thread)**
1. Приложение запрашивает у пользователя разрешение на показ уведомлений (`Notification.requestPermission()`).
2. Если юзер согласен, приложение обращается к `navigator.serviceWorker.ready.pushManager.subscribe()`.
3. Браузер связывается со своим пуш-сервисом (например, Google FCM для Chrome, Apple Push для Safari) и получает уникальный объект **PushSubscription** (в нем лежит эндпоинт и ключи шифрования).
4. Твое React/Vue приложение отправляет этот `PushSubscription` на твой бэкенд и сохраняет в БД, привязав к ID пользователя.

**Шаг 2. Отправка (На твоем Бэкенде - Node.js / Python / etc.)**
1. Бэкенд хочет отправить пуш (например, "Вам новое сообщение").
2. Бэкенд достает из БД `PushSubscription` этого юзера.
3. Используя специальную библиотеку (например, `web-push` для Node.js) и свои секретные VAPID-ключи (об этом ниже), бэкенд шифрует сообщение и отправляет HTTP POST запрос на URL пуш-сервиса (который лежал внутри `PushSubscription`).

**Шаг 3. Получение и показ (В Service Worker - Background Thread)**
1. Пуш-сервис доставляет сообщение на устройство юзера.
2. Браузер «просыпается», запускает твой файл `sw.js` (даже если вкладки сайта закрыты) и генерирует событие `push`.
3. Твой SW перехватывает событие, расшифровывает данные и вызывает `self.registration.showNotification()`.

#### 3. Пример использования (React/Vue + Service Worker)

**Часть 1: Приложение (React/Vue) — Подписка пользователя**
Здесь используется стандартный браузерный API, поэтому код одинаков для любого фреймворка.

```typescript
// В компоненте React или Vue (например, по клику на кнопку "Включить уведомления")

// VAPID public key - публичный ключ твоего сервера. 
// Он нужен, чтобы пуш-сервис браузера знал, что сообщения шлешь именно ты.
const PUBLIC_VAPID_KEY = 'BEl62i...твоя_строка_base64...'; 

async function subscribeUserToPush() {
  if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
    console.log('Push не поддерживается');
    return;
  }

  // 1. Проверяем регистрацию SW
  const registration = await navigator.serviceWorker.ready;

  try {
    // 2. Запрашиваем подписку у пуш-сервиса
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true, // Требование безопасности (мы обязуемся показать UI)
      applicationServerKey: urlB64ToUint8Array(PUBLIC_VAPID_KEY) // Конвертация ключа
    });

    console.log('Подписка получена:', JSON.stringify(subscription));

    // 3. Отправляем объект подписки на наш бэкенд (fetch / axios)
    await fetch('/api/subscribe', {
      method: 'POST',
      body: JSON.stringify(subscription),
      headers: { 'Content-Type': 'application/json' }
    });

  } catch (err) {
    console.error('Ошибка подписки:', err);
  }
}
```

**Часть 2: Service Worker (`sw.js`) — Перехват и показ пуша**
Этот код сработает, когда бэкенд отправит пуш.

```typescript
/// <reference lib="webworker" />
const sw = self as unknown as ServiceWorkerGlobalScope;

// 1. Слушаем событие прихода пуша
sw.addEventListener('push', (event) => {
  // Проверяем, есть ли полезная нагрузка (payload)
  let data = { title: 'Новое уведомление', body: 'У вас новое сообщение' };
  
  if (event.data) {
    data = event.data.json(); // Бэкенд должен слать JSON
  }

  // Опции оформления окошка уведомления
  const options: NotificationOptions = {
    body: data.body,
    icon: '/icons/icon-192x192.png', // Иконка рядом с текстом
    badge: '/icons/badge.png',       // Маленькая монохромная иконка (для Android статус-бара)
    vibrate: [200, 100, 200],        // Паттерн вибрации
    data: { url: '/messages/123' },  // Скрытые данные, пригодятся при клике
    actions: [                       // Кнопки в самом уведомлении
      { action: 'open', title: 'Открыть' },
      { action: 'close', title: 'Закрыть' }
    ]
  };

  // Показываем уведомление (waitUntil важен, чтобы SW не умер до показа!)
  event.waitUntil(
    sw.registration.showNotification(data.title, options)
  );
});

// 2. Слушаем клик пользователя по уведомлению
sw.addEventListener('notificationclick', (event) => {
  const notification = event.notification;
  const action = event.action; // Какую кнопку нажали

  notification.close(); // Сразу закрываем всплывающее окошко

  if (action === 'close') {
    return; // Просто закрыли
  }

  // Если нажали на само уведомление или кнопку 'open'
  // Хотим открыть вкладку браузера или сфокусироваться на существующей
  const targetUrl = notification.data.url;

  event.waitUntil(
    sw.clients.matchAll({ type: 'window' }).then((clientsArr) => {
      // Ищем, есть ли уже открытая вкладка с этим урлом
      const hadWindowToFocus = clientsArr.some((windowClient) => {
        if (windowClient.url === targetUrl) {
          windowClient.focus(); // Фокусируемся на ней
          return true;
        }
        return false;
      });

      // Если вкладки нет - открываем новую
      if (!hadWindowToFocus) {
        sw.clients.openWindow(targetUrl);
      }
    })
  );
});
```

#### 4. Тонкие моменты и нюансы (Где ошибаются новички)

1.  **VAPID ключи (Секретность):**
    *   *Нюанс:* VAPID (Voluntary Application Server Identification) — это стандарт. Ты генерируешь пару ключей (публичный и приватный) один раз. Публичный кладется во фронтенд (React/Vue), приватный — **строго на бэкенд**.
    *   *Ошибка:* Случайно закоммитить приватный ключ во фронтенд.
2.  **`userVisibleOnly: true` (Тихие пуши запрещены):**
    *   *Нюанс:* При вызове `subscribe` мы обязаны передавать этот флаг. Браузеры (особенно Chrome) жестко требуют: **на каждый пришедший пуш (событие `push` в SW) ты ОБЯЗАН показать UI-уведомление (`showNotification`)**.
    *   *Ошибка:* Попытаться использовать пуши для "тихой" фоновой синхронизации данных (например, обновить кэш без показа уведомления). Если ты поймал событие `push` и не вызвал `showNotification`, браузер накажет тебя и сам покажет юзеру дефолтное системное сообщение: "Этот сайт обновился в фоне" (что бесит юзеров).
3.  **Не работает на iOS (в старых версиях):**
    *   *Нюанс:* Apple долго сопротивлялась веб-пушам.
    *   Полноценная поддержка Web Push API в iOS (Safari) появилась **только в iOS 16.4** (весна 2023 года). До этого пуши там не работали вообще.
    *   *Важно для собеседования:* В iOS веб-пуши работают **ТОЛЬКО если юзер добавил PWA на домашний экран (Add to Home Screen)**. В обычном браузере Safari (без установки) пуши запросить нельзя.
4.  **`event.waitUntil()` при клике:**
    *   *Ошибка:* Забыть `waitUntil` в событии `notificationclick`.
    *   *Нюанс:* Поиск открытых окон (`clients.matchAll`) или открытие нового (`clients.openWindow`) — это асинхронные операции. Если не обернуть их в `waitUntil`, браузер может убить процесс SW до того, как окно откроется, и клик уйдет в никуда.