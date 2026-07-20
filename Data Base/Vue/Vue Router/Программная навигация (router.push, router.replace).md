Для управления навигацией во Vue используется инстанс роутера. В Composition API (Vue 3) мы получаем его через хук `useRouter()`.

#### Основные методы навигации

**1. `router.push()`**
Это самый частый метод. Он переходит на новый URL и **добавляет** запись в историю браузера (History API). Пользователь сможет нажать кнопку "Назад" в браузере и вернуться на предыдущую страницу.

**Vue (`useRouter`):**
```html
<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

const goToProfile = () => {
  // Вариант 1: передача строки
  router.push('/users/123')
  
  // Вариант 2: передача объекта (чаще используется на практике)
  router.push({ path: '/users/123' })
  
  // Вариант 3: передача объекта с параметрами запроса (query)
  // URL будет: /users/123?tab=profile
  router.push({ path: '/users/123', query: { tab: 'profile' } })
}
</script>
```

**Сравнение с React (`useNavigate`):**
В React v6 для этого используется функция `navigate`.
```javascript
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  
  const goToProfile = () => {
    // В React мы чаще передаем строку
    navigate('/users/123?tab=profile');
  };
}
```

**2. `router.replace()`**
Работает точно так же, как `push`, но **не добавляет** новую запись в историю браузера. Он заменяет текущую запись. 
*Когда использовать?* Например, если пользователь успешно залогинился, мы редиректим его на дашборд через `replace()`. Если он нажмет "Назад", он не попадет снова на страницу логина (ведь он уже авторизован), а вернется туда, откуда пришел до логина.

**Vue:**
```javascript
router.replace('/dashboard')
// или
router.replace({ path: '/dashboard' })
```

**React:**
```javascript
navigate('/dashboard', { replace: true });
```
*(Во Vue API более явное: разные методы `push` и `replace`, а в React это опция `replace: true` внутри метода `navigate`).*

**3. `router.go(n)` / `router.back()` / `router.forward()`**
Это методы для перемещения по истории браузера (аналог `window.history.go(n)`).
```javascript
router.go(1)   // На одну страницу вперед (аналог forward)
router.go(-1)  // На одну страницу назад (аналог back)
router.back()  // Читаемый синоним для go(-1)
```

В React (v6):
```javascript
navigate(1);  // Вперед
navigate(-1); // Назад
```

---

#### 🚨 Важный нюанс для собеседований: Именованные маршруты в push/replace

Здесь Vue выигрывает в удобстве у React. 
Во Vue роутере мы можем задавать маршрутам имена (об этом подробнее поговорим в теме про именованные маршруты, но использовать это мы можем уже сейчас).

Представь, что у тебя есть маршрут:
```javascript
{ path: '/user/:userId/settings/:tabId', name: 'user-settings', component: Settings }
```

Если ты хочешь перейти на него программно:
**Плохо (hardcode строки):**
```javascript
router.push(`/user/${user.id}/settings/${currentTab}`)
```

**Хорошо (использование имени и параметров):**
```javascript
router.push({ 
  name: 'user-settings', 
  params: { userId: user.id, tabId: currentTab } 
})
```
Почему второй вариант лучше? Если завтра бэкендеры попросят поменять URL с `/user/...` на `/profile/...`, тебе придется искать по всему коду строки `/user/`. А если ты используешь `name`, тебе достаточно изменить только поле `path` в конфигурации роутера, и все `router.push({ name: ... })` продолжат работать, так как они ссылаются на имя, а не на URL!

*(В React Router исторически с именованными маршрутами всё сложно, там обычно используют паттерн с функциями-генераторами путей вроде `generatePath()` или кастомные константы).*

---

#### Промисы и навигация

Еще одна особенность Vue Router: методы `router.push` и `router.replace` возвращают **Promise**.
Это значит, что мы можем дождаться завершения навигации:

```javascript
const goToDashboard = async () => {
  try {
    await router.push('/dashboard')
    console.log('Навигация успешно завершена!')
  } catch (error) {
    // Сработает, например, если навигация была отменена Navigation Guard'ом
    console.error('Не удалось перейти:', error)
  }
}
```

---

**Резюме для собеса по этой теме:**
1. Разница между `push` (добавляет в историю) и `replace` (перезаписывает текущий URL в истории).
2. Разделение: `useRouter()` — для действий (push, replace), `useRoute()` — для чтения данных текущего пути.
3. Удобство использования объекта `{ name: '...', params: {...} }` вместо конкатенации строк.
4. `router.push/replace` возвращают Promise (полезно для отлова отмены навигации).