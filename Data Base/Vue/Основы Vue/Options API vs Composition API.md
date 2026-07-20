## Options API vs Composition API

### Концепция и Теория

**Options API** — это классический подход (Vue 2), где компонент описывается как объект с четко разделенными полями: `data`, `methods`, `computed`, `watch`.

**Composition API** — подход Vue 3, где логика группируется не по «типу опции», а по «фиче» (функциональности).

**Зачем это нужно?**

1. **Логическая переиспользуемость:** В Options API код одной фичи (например, логика корзины) размазан по всем опциям (дата, методы, хуки). В Composition API всё, что относится к корзине, лежит в одной функции или файле.

2. **TypeScript:** Composition API гораздо лучше типизируется «из коробки».

3. **Масштабируемость:** Огромные компоненты в Options API превращаются в «спагетти», где нужно постоянно скроллить вверх-вниз.

**Как это работает «под капотом»:**

На самом деле, в Vue 3 Options API — это просто синтаксическая обертка. При компиляции Vue «под капотом» превращает ваши опции в те же реактивные функции Composition API.

---
### Синтаксис и База

В Composition API (особенно в `<script setup>`) мы просто объявляем переменные и функции.

**Options API (как было):**

```JavaScript
export default {
  data() { return { count: 0 } },
  methods: {
    increment() { this.count++ }
  }
}
```

**Composition API (как сейчас):**

```JavaScript
<script setup>
import { ref } from 'vue';

const count = ref(0); // Реактивное состояние
const increment = () => count.value++; // Обычная функция
</script>
```

---

### Сравнение с React

Здесь тебе будет проще всего, так как Composition API концептуально очень близок к React.

|**Vue (Composition API)**|**React (Hooks)**|**В чем разница**|
|---|---|---|
|`ref` / `reactive`|`useState`|Vue-состояние мутируемо (`count.value++`). Не нужно вызывать `setCount`.|
|`computed`|`useMemo`|Vue автоматически вычисляет зависимости. Не нужно передавать массив зависимостей `[a, b]`.|
|`watch`|`useEffect` (частично)|`watch` следит только за изменением данных, не смешивая это с жизненным циклом.|
|`composable` (useMyHook)|`custom hook`|Vue-хук выполняется **один раз** при создании. В нем нет проблем с "замыканиями" (stale closures).|

**Главное отличие:** В React хуки вызываются при **каждом** рендере. В Vue функция `setup` вызывается **один раз**. Поэтому во Vue можно использовать `if` или циклы вокруг `ref` и других хуков (хотя это и плохая практика), в то время как в React это строго запрещено правилами хуков.

_Если ты работала с MobX, то реактивность Vue тебе напомнит именно его: ты просто меняешь свойство объекта, а система сама знает, что перерисовать._

---

### Важные нюансы

1. **Прощай, `this`:** В Composition API (внутри `<script setup>`) слова `this` больше нет. Совсем. К пропсам, методам и состоянию мы обращаемся напрямую по имени переменной.

2. **Ref Unwrapping:** В `<script>` мы пишем `count.value`, но в `<template>` Vue сам «распаковывает» реф, и мы пишем просто `{{ count }}`. Это часто путает новичков.

3. **Mixins vs Composables:** В Options API для переиспользования кода использовались `mixins` (которые создавали конфликты имен). В Composition API мы используем `composables` (обычные функции), как кастомные хуки в React.


---

### Пример для заметок: Сравнение подходов

**Задача: Счетчик с удвоенным значением.**

**Options API:**

```JavaScript
// Код разбит по категориям
export default {
  data() {
    return { count: 0 }
  },
  computed: {
    double() { return this.count * 2 }
  },
  methods: {
    inc() { this.count++ }
  }
}
```

**Composition API (`<script setup>`):**

```JavaScript
<script setup>
import { ref, computed } from 'vue';

// Код сгруппирован по смыслу
const count = ref(0);
const double = computed(() => count.value * 2);
const inc = () => count.value++;
</script>

<template>
  <button @click="inc">{{ count }} (double: {{ double }})</button>
</template>
```
