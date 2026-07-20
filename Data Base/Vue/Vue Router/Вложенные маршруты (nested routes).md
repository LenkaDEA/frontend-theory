#### Настройка в конфигурации роутера

Чтобы объявить вложенные маршруты, во Vue используется свойство `children`. Оно принимает массив точно таких же объектов конфигурации, как и основной.

**Vue Router:**
```javascript
const routes = [
  {
    path: '/dashboard',
    component: DashboardLayout, // Компонент-обертка (например, Layout с Sidebar)
    children: [
      {
        // Путь будет /dashboard (так как путь пустой)
        // Это маршрут по умолчанию для dashboard
        path: '', 
        component: DashboardHome 
      },
      {
        // Путь будет /dashboard/settings
        path: 'settings', 
        component: DashboardSettings 
      },
      {
        // Путь будет /dashboard/users/:id
        path: 'users/:id', 
        component: UserProfile 
      }
    ]
  }
]
```
*Важный нюанс синтаксиса Vue:* Обрати внимание, что у дочерних путей **нет слеша `/` в начале** (пишем `'settings'`, а не `'/settings'`). Если написать `'/settings'`, Vue будет считать это абсолютным путем, и вложенность сломается (хотя компонент отрендерится во вложенном `router-view`, URL будет просто `/settings`, а не `/dashboard/settings`).

**Сравнение с React Router v6:**
Конфигурация очень похожа, только используется ключ `children` и `index` для маршрута по умолчанию:
```javascript
const router = createBrowserRouter([
  {
    path: "/dashboard",
    element: <DashboardLayout />,
    children: [
      { index: true, element: <DashboardHome /> }, // Маршрут по умолчанию
      { path: "settings", element: <DashboardSettings /> }
    ],
  },
]);
```

---

#### Как это отображается в компоненте (Layout)

Чтобы дочерний компонент отрендерился внутри родительского, в родительском компоненте нужно указать "точку входа". 

**Vue Router:**
Во Vue для этого используется специальный встроенный компонент `<router-view />`.

```vue
<!-- DashboardLayout.vue -->
<template>
  <div class="dashboard-layout">
    <!-- Боковое меню, которое всегда на экране -->
    <aside class="sidebar">
      <router-link to="/dashboard">Главная</router-link>
      <router-link to="/dashboard/settings">Настройки</router-link>
    </aside>
    <main class="content">
    <!-- Сюда будут динамически подставляться DashboardHome, DashboardSettings и т.д. -->
      <router-view />
    </main>
  </div>
</template>
```

**React Router:**
В React для абсолютно той же цели используется компонент `<Outlet />`.

```javascript
// DashboardLayout.jsx
import { Outlet, Link } from "react-router-dom";

function DashboardLayout() {
  return (
    <div className="dashboard-layout">
      <aside className="sidebar">
        <Link to="/dashboard">Главная</Link>
        <Link to="/dashboard/settings">Настройки</Link>
      </aside>
      
      <main className="content">
        {/* Здесь будут рендериться дочерние элементы */}
        <Outlet /> 
      </main>
    </div>
  );
}
```

---

#### Многоуровневая вложенность (Нюанс для собесов)

На собеседованиях иногда спрашивают: *"Можно ли делать вложенность бесконечной?"*
Да, свойство `children` можно вкладывать друг в друга сколько угодно раз. Главное правило: **на каждый уровень вложенности в конфигурации должен быть свой `<router-view />` в соответствующем компоненте.**

Если у тебя: `App -> DashboardLayout -> User