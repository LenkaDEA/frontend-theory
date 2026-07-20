### Динамические маршруты и параметры (:id)

#### Как задаются маршруты

И во Vue Router, и в React Router синтаксис определения динамических сегментов **одинаковый** — используется двоеточие `:`.

**Vue Router:**
```javascript
const routes = [
  // Динамический сегмент начинается с двоеточия
  { path: '/users/:id', component: UserComponent }
]
```

#### Как получить доступ к параметрам

Здесь кроется важное концептуальное отличие в хуках, которое нужно запомнить. 

*   **В React** функционал разбит на несколько узких хуков: `useParams()` для параметров, `useLocation()` для текущего пути/query, `useNavigate()` для переходов.
*   **Во Vue** все данные о *текущем маршруте* (params, query, path, hash) лежат в одном объекте, который возвращает хук `useRoute()`. А методы для *управления навигацией* (push, replace) — в хуке `useRouter()`. 

**Vue (Composition API `setup`):**
```html
<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()
// Получаем id из URL (например, /users/123 -> '123')
console.log(route.params.id) 
</script>
```

**React:**
```javascript
import { useParams } from 'react-router-dom'

function UserComponent() {
  const { id } = useParams()
  console.log(id)
}
```

---

#### 🚨 ГЛАВНЫЙ ВОПРОС НА СОБЕСЕДОВАНИЯХ: Реактивность параметров

Это самая частая тема, на которой ловят новичков во Vue, особенно при переходе с других фреймворков.

**Сценарий:** Пользователь находится на странице `/users/1` и кликает на ссылку, ведущую на `/users/2`. 
Что произойдет с компонентом `UserComponent`?

**Ответ:** Vue для оптимизации производительности **не уничтожает и не создает компонент заново**. Он переиспользует тот же самый экземпляр компонента. 
Из-за этого хуки жизненного цикла, такие как `onMounted` (в React аналог `useEffect` с пустым массивом зависимостей `[]`), **не сработают повторно**.

**Как реагировать на изменение параметра?**

**В React** ты бы просто добавил `id` в массив зависимостей `useEffect`:
```javascript
// React
const { id } = useParams();

useEffect(() => {
  fetchUserData(id); // Сработает при монтировании и при каждом изменении id
}, [id]); 
```

**Во Vue** есть два основных способа это решить:

**Способ 1: Использовать `watch` (Самый частый подход в Composition API)**
Мы следим за изменениями `route.params.id`:
```html
<script setup>
import { useRoute } from 'vue-router'
import { watch, onMounted } from 'vue'

const route = useRoute()

// Функция загрузки данных
const fetchUserData = async (id) => { /* ... */ }

// Вызовется при первом открытии страницы
onMounted(() => {
  fetchUserData(route.params.id)
})

// Вызовется, когда параметр поменяется (с /users/1 на /users/2)
watch(
  () => route.params.id,
  (newId, oldId) => {
    fetchUserData(newId)
  }
)
</script>
```
*Лайфхак:* Во Vue у `watch` есть опция `immediate: true`, которая позволяет объединить `onMounted` и `watch` в один блок (аналог того самого `useEffect`):
```javascript
watch(
  () => route.params.id,
  (newId) => { fetchUserData(newId) },
  { immediate: true } // Сработает сразу при инициализации и потом при изменениях
)
```

**Способ 2: Навигационный хук (in-component guard)**
Во Vue есть встроенный хук специально для этой ситуации — `onBeforeRouteUpdate`:
```javascript
import { onBeforeRouteUpdate } from 'vue-router'

onBeforeRouteUpdate(async (to, from) => {
  // to - куда идем, from - откуда
  if (to.params.id !== from.params.id) {
    await fetchUserData(to.params.id)
  }
})
```

---

#### Бонус: Роуты 404 (Catch-all / Not Found)

Чтобы перехватить любой URL, которого нет в ваших маршрутах (показать страницу 404), синтаксис немного отличается.

**React Router:**
```javascript
{ path: '*', element: <NotFound /> }
```

**Vue Router (начиная с 4-й версии):**
Раньше во Vue тоже была звездочка `*`, но в новой версии нужно использовать специальный синтаксис регулярных выражений:
```javascript
{ 
  // Совпадет со всем и положит путь в параметр pathMatch
  path: '/:pathMatch(.*)*', 
  component: NotFound 
}
```

---

**Резюме для собеса по этой теме:**
1. Разница между `useRoute` (данные текущего URL: params, query) и `useRouter` (объект управления навигацией).
2. Понимание, что при изменении динамического параметра компонент **переиспользуется**.
3. Знание, как обновить данные при изменении URL (через `watch` на `route.params` или хук `onBeforeRouteUpdate`).