
### Настройка роутера, режимы history и hash

Во Vue настройка роутера происходит централизованно. В отличие от React Router, где маршруты часто объявляются декларативно прямо в JSX (особенно до появления `createBrowserRouter` в v6), во Vue принято создавать отдельный файл (обычно `router/index.js` или `router.ts`), где описывается массив объектов маршрутов.

#### Базовая настройка

**Vue Router (Vue 3):**
Для создания роутера используется функция `createRouter`, в которую передаются настройки истории и массив маршрутов.

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About }
]

const router = createRouter({
  history: createWebHistory(), // режим истории
  routes
})

export default router
```

Затем роутер подключается в главном файле `main.js`:
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)
app.use(router) // Подключаем плагин роутера
app.mount('#app')
```

**Сравнение с React (React Router v6):**
В React сейчас используется похожий подход с дата-роутерами, но синтаксис немного другой.

```javascript
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import Home from "./Home";
import About from "./About";

const router = createBrowserRouter([
  { path: "/", element: <Home /> },
  { path: "/about", element: <About /> },
]);

function App() {
  return <RouterProvider router={router} />;
}
```
*Ключевое отличие:* Во Vue мы передаем в `component` саму ссылку на компонент (объект компонента), а в React Router мы передаем React-элемент (JSX: `<Home />`). 

#### Режимы history и hash

Оба фреймворка поддерживают два основных способа управления URL-адресом в браузере.

**1. History Mode (HTML5 History API)**
Это стандартный и самый популярный режим для современных SPA (Single Page Applications). URL выглядит чисто: `yoursite.com/about`.

* **Vue Router:** `history: createWebHistory()`
* **React Router:** `createBrowserRouter` (или `<BrowserRouter>`)
* **Особенность (Важно для собесов!):** При использовании этого режима требуется настройка сервера (Nginx, Apache, и т.д.). Если пользователь напрямую зайдет на `yoursite.com/about` или обновит страницу, сервер попытается найти файл `about/index.html` и вернет ошибку 404, если не настроен fallback. Сервер должен быть сконфигурирован так, чтобы на любые URL возвращать главный `index.html`, а дальше уже JS на клиенте разберется, что показать.

**2. Hash Mode**
В этом режиме URL использует хэш-символ (якорь): `yoursite.com/#/about`.

* **Vue Router:** `history: createWebHashHistory()`
* **React Router:** `createHashRouter` (или `<HashRouter>`)
* **Особенность:** Сервер никогда не отправляет часть URL после `#`. Поэтому сервер вообще не нужно настраивать под роутинг — любой запрос всегда идет к `index.html`. 
* **Где применяется:** В основном для legacy-приложений, десктопных приложений на Electron, или если у вас вообще нет контроля над сервером (например, деплой на какой-нибудь простой статический хостинг без поддержки перенаправлений). В современной веб-разработке используется редко из-за плохой совместимости с SEO и некрасивых URL.

**(Дополнительно) Memory Mode:**
Есть еще режим для тестов и серверного рендеринга (SSR), который не меняет URL в браузере.
* **Vue:** `createMemoryHistory()`
* **React:** `createMemoryRouter` (или `<MemoryRouter>`)

---

**Итог по первой теме для собеседования:**
На собеседовании могут спросить: *"Зачем нужен History API и в чем проблема 404 ошибки при обновлении страницы?"*
Твой ответ: *"В режиме History мы используем встроенный в браузер History API (методы pushState, replaceState), чтобы менять URL без перезагрузки страницы. Проблема 404 возникает, потому что сервер не знает об этих виртуальных маршрутах. Чтобы её решить, нужно настроить сервер отдавать `index.html` на любой неизвестный запрос (catch-all route)."*

