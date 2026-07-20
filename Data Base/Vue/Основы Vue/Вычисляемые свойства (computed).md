## Вычисляемые свойства (computed)

### Концепция и Теория

**Какую задачу решает:** Когда нужно отобразить данные, которые зависят от другого состояния (например, отфильтрованный список, форматированная дата или флаг `isDisabled`), писать сложную логику прямо в `<template>` — плохая практика. `computed` позволяет вынести эту логику в отдельную переменную, которая будет автоматически пересчитываться.

**Как работает «под капотом»:** Главная суперсила `computed` — это **кэширование**.

Vue оборачивает вашу функцию-геттер в специальный эффект. При первом обращении функция выполняется, а Vue через Proxy автоматически "записывает", какие реактивные переменные (зависимости) были прочитаны внутри нее. Результат кэшируется. При последующих обращениях возвращается кэш, функция заново _не выполняется_. Перерасчет произойдет только в тот момент, когда изменится одна из отслеженных зависимостей.
### Синтаксис и База

Импортируем `computed` из `vue` и передаем в него коллбэк, возвращающий значение.

```vue
<script setup>
import { ref, computed } from 'vue';

const items = ref([1, 2, 3, 4, 5]);
const multiplier = ref(2);

// Объявление вычисляемого свойства
const multipliedItems = computed(() => {
  // Vue автоматически запомнит, что это свойство зависит от items.value и multiplier.value
  return items.value.map(item => item * multiplier.value);
});
</script>

<template>
  <div>{{ multipliedItems }}</div>
</template>
```

### Сравнение с React

В React прямым аналогом `computed` является хук `useMemo`.

|**Фича**|**React (useMemo)**|**Vue 3 (computed)**|
|---|---|---|
|**Синтаксис**|`useMemo(() => a + b, [a, b])`|`computed(() => a.value + b.value)`|
|**Зависимости**|Передаются массивом вручную|Отслеживаются автоматически|
|**Ленивость**|Вычисляется во время рендера|Вычисляется лениво (только когда к нему обращаются)|
|**Кэширование**|Сбрасывается, если изменилась хоть одна зависимость из массива|Сбрасывается, только если изменилась реальная зависимость под капотом|

**Принципиальная разница:**

В React вам нужно постоянно держать в голове массив зависимостей. Забыли добавить переменную — получили баг (stale state). Во Vue система реактивности на базе Proxy делает это за вас. Если внутри `computed` есть условие `if (isTrue.value) return a.value else return b.value`, то Vue динамически перестраивает зависимости: если `isTrue` равно `false`, изменение `a` не вызовет перерасчет! В React `useMemo` пересчитал бы значение в любом случае, так как `a` лежало бы в массиве зависимостей.

### Важные нюансы

1. **Никаких сайд-эффектов:** Внутри `computed` нельзя делать асинхронные запросы (`fetch`), менять другие реактивные переменные или мутировать DOM. Эта функция должна быть "чистой" (pure) — только брать данные, трансформировать и возвращать. Для сайд-эффектов нужен `watch`.

2. **Read-only по умолчанию:** `computed` возвращает константу. Нельзя написать `multipliedItems.value = [10, 20]`. Vue выдаст предупреждение. (Существуют `writable computed` с геттерами и сеттерами, но они используются крайне редко, например, для работы с Vuex/Pinia или сложными формами).

3. **Не забывай `.value`:** Внутри функции ты работаешь с другими `ref`, поэтому обязательно нужно писать `.value`, иначе ты будешь пытаться сложить или отфильтровать объекты Proxy, а не их значения.

### Пример для заметок: Фильтрация и поиск

Классический пример, где `computed` показывает себя лучше всего.

```vue
<script setup>
import { ref, computed } from 'vue';

const searchQuery = ref('');
const filterStatus = ref('all'); // 'all', 'active', 'inactive'

const users = ref([
  { id: 1, name: 'Анна', status: 'active' },
  { id: 2, name: 'Борис', status: 'inactive' },
  { id: 3, name: 'Виктор', status: 'active' },
]);

// Умный отфильтрованный список. Пересчитается ТОЛЬКО если изменится searchQuery, filterStatus или массив users.
const filteredUsers = computed(() => {
  let result = users.value;

  // 1. Фильтр по статусу
  if (filterStatus.value !== 'all') {
    result = result.filter(user => user.status === filterStatus.value);
  }

  // 2. Поиск по тексту
  if (searchQuery.value) {
    const lowerQuery = searchQuery.value.toLowerCase();
    result = result.filter(user => 
      user.name.toLowerCase().includes(lowerQuery)
    );
  }

  return result;
});
</script>

<template>
  <div>
    <input v-model="searchQuery" placeholder="Поиск по имени..." />
    
    <select v-model="filterStatus">
      <option value="all">Все</option>
      <option value="active">Активные</option>
      <option value="inactive">Неактивные</option>
    </select>

    <ul>
      <li v-for="user in filteredUsers" :key="user.id">
        {{ user.name }} ({{ user.status }})
      </li>
    </ul>
    
    <p v-if="filteredUsers.length === 0">Ничего не найдено</p>
  </div>
</template>
```