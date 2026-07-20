#### Архитектурная теория

Vuex создавался в эпоху Vue 2, где система реактивности строилась на геттерах и сеттерах через `Object.defineProperty`. Это накладывало жесткие ограничения: Vuex представлял собой единое глобальное дерево состояний (Single Source of Truth), а для отслеживания изменений в DevTools требовался жесткий контракт — мутации (mutations). Мутации были единственным синхронным способом изменить стейт, чтобы DevTools могли сделать "снапшот" состояния.

Pinia была написана с нуля под Vue 3 и его новую систему реактивности на базе Proxy.

- Как это работает под капотом: Когда ты создаешь стор в Pinia, под капотом вызывается стандартная функция Vue `reactive()` (или `ref()`). Стор — это просто проксированный объект.
- Почему исчезли мутации: Благодаря Proxy, Vue 3 (и Pinia) может перехватывать _любое_ обращение к свойству объекта (trap `set`). Pinia напрямую интегрируется с Vue DevTools, подписываясь на эти изменения под капотом. Необходимость в искусственной прослойке в виде мутаций отпала — теперь DevTools и так "видит", когда и где изменилось свойство. Поэтому в Pinia остались только `actions`, которые могут быть как синхронными, так и асинхронными.

#### Аналогия с MobX / React

- MobX: Pinia — это практически брат-близнец MobX в мире Vue. В Vuex (как и в Redux) мы диспатчили экшены, которые вызывали редьюсеры/мутации. В Pinia мы просто объявляем `state` (аналог `makeAutoObservable` или `@observable`) и мутируем его напрямую внутри `actions` (аналог `@action` в MobX).
- React: Если Vuex был похож на монолитный Redux с его `combineReducers`, то архитектура Pinia больше напоминает атомарный подход (как Jotai/Zustand) или использование множества независимых React Context. Каждый стор в Pinia изолирован и инициализируется только в момент первого вызова (lazy evaluation).

#### Best Practices & TypeScript

- Типизация из коробки: Главная боль Vuex была в TypeScript. Приходилось писать сложные интерфейсы, обертки и использовать "магические" строки (`dispatch('module/actionName')`), которые ломали автокомплит. В Pinia типы выводятся автоматически (Type Inference). Стор возвращает типизированный объект, и ты вызываешь методы как обычные функции класса.
- Плоская структура (Flat Architecture): Не пытайся воссоздать единое дерево стейта. Создавай маленькие, сфокусированные сторы (`useAuthStore`, `useCartStore`, `useUiStore`).

#### Главный "подводный камень"

Потеря реактивности при деструктуризации. Так как стор Pinia — это реактивный объект (Proxy), ты не можешь просто деструктурировать его, как обычный объект в JS. Если ты напишешь `const { count } = useCounterStore()`, ты вытащишь примитивное значение, и оно перестанет обновляться в UI (в MobX похожая проблема решается через observer-HOC, но в Vue деструктуризация Proxy ломает связь). _Решение:_ Использовать утилиту `storeToRefs` (аналог `toRefs` из Vue) для стейта и геттеров. Экшены можно деструктурировать напрямую, так как они привязаны к контексту стора.

#### Пример кода
```ts
// stores/useUserStore.ts
import { defineStore } from 'pinia'

// Типизация стейта (опционально, но рекомендуется для сложных объектов)
interface UserState {
  id: string | null
  name: string
  roles: string[]
}

export const useUserStore = defineStore('user', {
  // State: стрелочная функция, возвращающая объект (аналог data в компонентах)
  state: (): UserState => ({
    id: null,
    name: '',
    roles: [],
  }),

  // Actions: напрямую мутируем стейт (никаких mutations!)
  actions: {
    async fetchUser(userId: string) {
      // Асинхронный код прямо здесь
      const response = await api.getUser(userId)
      
      // Прямая мутация стейта (Proxy перехватит это для DevTools)
      this.id = response.id
      this.name = response.name
      this.roles = response.roles
    },
    
    logout() {
      this.$reset() // Встроенный метод Pinia для сброса стейта к initial state
    }
  }
})
```

```ts
<!-- components/UserProfile.vue -->
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/useUserStore'

const userStore = useUserStore()

// ПРАВИЛЬНО: сохраняем реактивность стейта через storeToRefs
const { name, roles } = storeToRefs(userStore)

// ПРАВИЛЬНО: экшены можно деструктурировать напрямую
const { fetchUser, logout } = userStore 

// Вызов экшена (строгая типизация аргументов)
fetchUser('user-123')
</script>

<template>
  <div>
    <!-- name и roles реактивно обновятся после fetchUser -->
    <h1>{{ name }}</h1>
    <button @click="logout">Выйти</button>
  </div>
</template>
```