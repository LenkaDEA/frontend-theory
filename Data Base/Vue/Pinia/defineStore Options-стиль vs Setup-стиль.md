#### Архитектурная теория

Pinia предоставляет два способа описания стора, которые под капотом компилируются в один и тот же реактивный объект.

1. Options-стиль (Options Store): Традиционный подход, похожий на Vuex и Options API во Vue. Ты передаешь объект с полями `state`, `getters` и `actions`.
    - _Как работает:_ Pinia берет этот объект, оборачивает `state` в `reactive()`, `getters` в `computed()`, а `actions` привязывает к контексту (`this`).
2. Setup-стиль (Setup Store): Современный подход, использующий синтаксис Vue Composition API. Вместо объекта конфигурации ты передаешь функцию (похожую на функцию `setup` в компонентах Vue).
    - _Как работает:_ Ты используешь стандартные реактивные примитивы Vue: `ref()` или `reactive()` для стейта, `computed()` для геттеров и обычные функции для экшенов. В конце функции ты возвращаешь объект с тем, что должно быть доступно снаружи (публичный API стора).

Ментальная связь с Vue Composables: Setup-стор — это, по сути, глобальный Composable (аналог глобального кастомного хука). Обычный Composable (`useSomething`) создает новое состояние при каждом вызове внутри компонента. Setup-стор в Pinia вызывает эту функцию _только один раз_ при первом обращении, а затем кэширует результат и возвращает один и тот же инстанс всем подписчикам.

#### Аналогия с MobX / React

- Options-стиль похож на классический классовый подход в MobX (где есть свойства класса и методы) или на создание стора через `makeAutoObservable(this)`.
- Setup-стиль — это 1 в 1 кастомный хук в React (например, `useAuth`), который использует `useState`/`useMemo`/`useCallback` и возвращает объект с данными и функциями. Только в React для того, чтобы сделать этот хук глобальным, тебе пришлось бы оборачивать его в Context API. В Pinia `defineStore` делает эту "глобализацию" за тебя из коробки.

#### Best Practices & TypeScript

- Когда какой стиль предпочесть?
    - _Options-стиль:_ Идеален для простых сторов. Он более декларативный, строгий и имеет встроенный метод `$reset()` (который сбрасывает стейт к первоначальному, так как Pinia знает начальную функцию `state`).
    - _Setup-стиль:_ Стандарт де-факто для сложных приложений. Позволяет использовать внутри стора другие Composables (например, `useRouter`, `useFetch`, `useLocalStorage`), использовать `watch` для сайд-эффектов и гибко управлять приватными/публичными методами (не возвращая их из функции).
- Типизация: В Setup-стиле TypeScript работает еще лучше, так как это просто обычный JavaScript/TypeScript код без магии контекста `this`. Типы выводятся автоматически из возвращаемого объекта.

#### Главный "подводный камень"

Потеря метода `$reset()` и SSR. В Setup-сторах метод `store.$reset()` не работает из коробки. Pinia не знает, каким было начальное состояние, потому что стейт создается через обычные `ref()`. Если тебе нужен сброс состояния в Setup-сторе, тебе придется написать функцию `reset` вручную. Также в Setup-сторах нужно быть осторожным с глобальными сайд-эффектами (например, глобальными `watch`), чтобы не вызвать утечки памяти при Server-Side Rendering (SSR).

#### Пример кода

Сравним оба подхода на примере одного и того же стора корзины.

1. Options-стиль:
```ts
import { defineStore } from 'pinia'

export const useCartOptionsStore = defineStore('cart-options', {
  state: () => ({
    items: [] as string[],
    isSyncing: false
  }),
  getters: {
    itemsCount: (state) => state.items.length,
    // Доступ к другому геттеру через this
    isEmpty(): boolean {
      return this.itemsCount === 0
    }
  },
  actions: {
    addItem(item: string) {
      this.items.push(item)
    }
  }
})
```

2. Setup-стиль (Production-ready подход):
```ts
import { defineStore } from 'pinia'
import { ref, computed, watch } from 'vue'
// Можно использовать другие composables прямо внутри стора!
import { useLocalStorage } from '@vueuse/core' 

export const useCartSetupStore = defineStore('cart-setup', () => {
  // --- State (ref / reactive) ---
  // Используем useLocalStorage вместо обычного ref для авто-сохранения
  const items = useLocalStorage<string[]>('cart-items', [])
  const isSyncing = ref(false)

  // Приватный стейт (не возвращаем его наружу)
  const _internalId = ref(Math.random())

  // --- Getters (computed) ---
  const itemsCount = computed(() => items.value.length)
  const isEmpty = computed(() => itemsCount.value === 0)

  // --- Actions (обычные функции) ---
  function addItem(item: string) {
    items.value.push(item)
  }

  // Ручная реализация $reset, так как встроенный не работает в Setup-стиле
  function $reset() {
    items.value = []
    isSyncing.value = false
  }

  // --- Сайд-эффекты (watch) ---
  watch(items, (newItems) => {
    console.log('Корзина изменилась:', newItems.length)
  }, { deep: true })

  // --- Public API ---
  // Возвращаем только то, что должно быть доступно компонентам
  return {
    items,
    isSyncing,
    itemsCount,
    isEmpty,
    addItem,
    $reset
  }
})
```