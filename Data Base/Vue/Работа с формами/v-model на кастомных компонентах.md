#### 📝 Определение
`v-model` — это встроенная директива Vue для создания **двустороннего связывания данных** (two-way data binding) между родительским и дочерним компонентом. 

Под капотом это просто **синтаксический сахар**. Vue автоматически разворачивает `v-model` в передачу пропса и прослушивание события обновления этого пропса.

#### ⚖️ Сравнение с React
В React нет двустороннего связывания. Чтобы создать "управляемый компонент" (controlled component), ты всегда вручную передаешь состояние (`value`) и функцию для его изменения (`onChange`). 

В Vue `v-model` делает ровно то же самое, но прячет эту рутину от разработчика.

**React (Ментальная модель):**
```jsx
// Родитель
<CustomInput value={text} onChange={(e) => setText(e.target.value)} />
```

**Vue (То же самое под капотом):**
```html
<!-- Родитель: то, как мы пишем -->
<CustomInput v-model="text" />

<!-- Родитель: во что это превращается под капотом во Vue 3 -->
<CustomInput 
  :modelValue="text" 
  @update:modelValue="newValue => text = newValue" 
/>
```

#### 🛠 Как использовать (Код)

Во Vue 3 по умолчанию `v-model` на кастомном компоненте ожидает, что дочерний компонент принимает пропс `modelValue` и эмитит (вызывает) событие `update:modelValue`.

**Дочерний компонент (`CustomInput.vue`):**
```html
<script setup>
// 1. Объявляем пропс, который приходит от v-model
defineProps({
  modelValue: String
})

// 2. Объявляем событие, которое будем вызывать для обновления
defineEmits(['update:modelValue'])
</script>

<template>
  <!-- Привязываем значение к пропсу и слушаем нативный инпут -->
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

**Родительский компонент:**
```html
<script setup>
import { ref } from 'vue'
import CustomInput from './CustomInput.vue'

const searchText = ref('')
</script>

<template>
  <CustomInput v-model="searchText" />
  <p>Вы ищете: {{ searchText }}</p>
</template>
```

#### ⚠️ Особенности и ловушки (Pitfalls) для собеседования

1. **Ловушка мутации пропса (Самая частая ошибка новичков):**
   В React ты знаешь, что пропсы менять нельзя (read-only). Во Vue то же самое! 
   *Частая ошибка:* написать в дочернем компоненте `<input v-model="modelValue" />`. Это вызовет предупреждение в консоли, так как дочерний компонент попытается напрямую изменить пропс `modelValue`, переданный родителем. 
   *Решение:* Использовать связку `:value` + `@input` (как в примере выше) или использовать `computed` свойство с геттером и сеттером.

2. **Несколько `v-model` на одном компоненте:**
   В отличие от React, где ты просто передаешь несколько пар `value/onChange` (например, `title/setTitle`, `content/setContent`), во Vue 3 можно использовать именованные `v-model`.
   *Синтаксис:* `<CustomForm v-model:title="title" v-model:content="content" />`.
   Внутри компонента это будут пропсы `title` и `content`, и события `update:title` и `update:content`.

3. **Разница между Vue 2 и Vue 3 (Могут спросить на собеседовании):**
   Во Vue 2 дефолтный пропс для `v-model` назывался `value`, а событие — `input`. Во Vue 3 их переименовали в `modelValue` и `update:modelValue` для большей консистентности.