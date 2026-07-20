### Концепция и Теория

Если в React мы используем JSX (JavaScript, который умеет рендерить HTML), то во Vue мы используем **валидный HTML, расширенный специальными атрибутами** — директивами.

**Как это работает «под капотом»:**

Vue парсит ваш `<template>` и ищет атрибуты, начинающиеся с префикса `v-`. Для компилятора это инструкции: «Ага, здесь нужно привязать слушатель событий», «Здесь нужно создать цикл», «А здесь нужно следить за переменной и обновлять атрибут». На выходе компилятор превращает всю эту разметку с директивами в обычные JavaScript-функции рендера (аналогично тому, как транспилируется JSX).
### Синтаксис и База

Директивы во Vue — это основа управления разметкой. У большинства частых директив есть удобные сокращения.

- **`v-bind` (сокращение `:`):** Привязывает JS-переменную к HTML-атрибуту.
    
    - _Полностью:_ `<img v-bind:src="imageUrl" />`
        
    - _Кратко:_ `<img :src="imageUrl" />`

- **`v-on` (сокращение `@`):** Вешает слушатель событий.
    
    - _Полностью:_ `<button v-on:click="handleClick">`
        
    - _Кратко:_ `<button @click="handleClick">`

- **`v-if` / `v-else-if` / `v-else`:** Условный рендеринг. Если условие ложно, элемент физически удаляется из DOM.

- **`v-show`:** Тоже скрывает элемент, но не удаляет из DOM, а просто добавляет CSS-свойство `display: none`.

- **`v-for`:** Рендеринг списков (цикл).

- **`v-model`:** «Магия» двустороннего связывания данных для полей ввода.

### Сравнение с React

В React вся логика шаблона строится на чистом JavaScript (тернарные операторы, `map`, логические `&&`). Во Vue для этого есть декларативные директивы.

|**Задача**|**React (JSX)**|**Vue 3 (Template)**|
|---|---|---|
|**Интерполяция**|`<div>{text}</div>`|`<div>{{ text }}</div>`|
|**Условие (IF)**|`{isTrue && <div>Да</div>}`|`<div v-if="isTrue">Да</div>`|
|**Условие (IF/ELSE)**|`{isTrue ? <span>А</span> : <span>Б</span>}`|`<span v-if="isTrue">А</span><span v-else>Б</span>`|
|**Циклы**|`{items.map(i => <li key={i.id}>{i.name}</li>)}`|`<li v-for="i in items" :key="i.id">{{ i.name }}</li>`|
|**Обработка событий**|`<button onClick={handleClick}>`|`<button @click="handleClick">`|
|**Инпуты (стейт)**|`<input value={val} onChange={e => setVal(e.target.value)} />`|`<input v-model="val" />`|

**Принципиальная разница:**

`v-model` — это синтаксический сахар. В React тебе нужно вручную прописывать `value` и обрабатывать `onChange`. Во Vue директива `v-model` делает это под капотом: она сама привязывает `value` к переменной и сама обновляет эту переменную при событии `input`.

### Важные нюансы

1. **`v-if` против `v-show`:**
    
    - Используй `v-if`, если условие меняется редко (переключение вкладок, показ модалки). Перерисовка DOM стоит дорого.
        
    - Используй `v-show`, если элемент нужно переключать очень часто (например, выпадающее меню при наведении).

2. **Никогда не используй `v-if` и `v-for` на одном элементе.** Это самая частая ошибка. Приоритет директив таков, что `v-if` выполнится раньше `v-for` и не получит доступ к переменной из цикла. _Решение:_ обернуть цикл в невидимый тег `<template v-for="...">` и внутри него уже использовать `v-if`.

3. **`:key` в `v-for`:** Как и в React, Vue нуждается в уникальных ключах для элементов списка, чтобы алгоритм Virtual DOM мог эффективно переиспользовать узлы, а не перерисовывать весь список при изменении порядка.
### Пример для заметок

Комплексный пример, объединяющий основные директивы:

``` js
<script setup>
import { ref } from 'vue';

const newTask = ref('');
const tasks = ref([
  { id: 1, title: 'Изучить Vue', isDone: true },
  { id: 2, title: 'Понять директивы', isDone: false },
]);
const showCompleted = ref(true);

const addTask = () => {
  if (newTask.value.trim() === '') return;
  
  tasks.value.push({
    id: Date.now(),
    title: newTask.value,
    isDone: false
  });
  
  // Очищаем инпут
  newTask.value = '';
};
</script>

<template>
  <div>
    <h2>Мои задачи</h2>
    
    <input 
      v-model="newTask" 
      @keyup.enter="addTask" 
      placeholder="Новая задача..." 
    />
    <button @click="addTask">Добавить</button>

    <br><br>
    
    <label>
      <input type="checkbox" v-model="showCompleted" />
      Показывать выполненные
    </label>

    <ul>
      <li 
        v-for="task in tasks" 
        :key="task.id"
        v-show="showCompleted || !task.isDone"
      >
        <span :class="{ 'done-text': task.isDone }">
          {{ task.title }}
        </span>
        
        <button @click="task.isDone = !task.isDone">
          <span v-if="task.isDone">Отменить</span>
          <span v-else>Завершить</span>
        </button>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.done-text {
  text-decoration: line-through;
  color: gray;
}
</style>
```