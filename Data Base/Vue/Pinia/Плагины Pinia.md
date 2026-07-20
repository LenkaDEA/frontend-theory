#### Архитектурная теория

Плагин в Pinia — это функция, которая вызывается один раз для каждого стора в момент его инициализации (когда ты первый раз вызываешь `useMyStore()`).

- _Что делает плагин:_ Он получает объект контекста (`context`), который содержит сам инстанс стора (`store`), инстанс приложения Vue (`app`) и опции, с которыми стор был создан (`options`).
- _Как работает:_ Плагин может модифицировать стор "на лету". Он может добавлять новые реактивные свойства в стейт, новые методы (экшены), оборачивать существующие методы или настраивать глобальные подписки (`$subscribe` / `$onAction`).
- _Возвращаемое значение:_ Если плагин возвращает объект, свойства этого объекта автоматически добавляются в инстанс стора (и становятся реактивными, если это `ref` или `reactive`).

#### Аналогия с MobX / React

- MobX: В MobX нет прямой концепции "плагинов для сторов", так как сторы — это просто классы. Подобное поведение достигается через наследование базового класса (BaseStore) или использование декораторов/миксинов.
- React: В Redux это классические Middleware или Store Enhancers. В контексте кастомных хуков React аналогом будет создание HOC (Higher-Order Component) для хука, либо фабрики хуков, которая оборачивает каждый твой `useStore` и добавляет туда, например, функцию `router.push`.

#### Best Practices & TypeScript

- Типизация — самое сложное в плагинах. Так как плагин динамически добавляет свойства в стор, TypeScript об этом изначально не знает. Чтобы автокомплит работал, необходимо использовать Declaration Merging (расширение интерфейсов TypeScript). Ты должен расширить интерфейс `PiniaCustomProperties`.
- Изоляция: Не добавляй в плагины тяжелую бизнес-логику. Плагины нужны для инфраструктуры: роутинг, API-клиенты, аналитика, персистентность (сохранение в LocalStorage).
- Чтение кастомных опций: Плагины могут читать кастомные поля из конфигурации стора (третий аргумент в `defineStore`), что позволяет включать/выключать поведение плагина для конкретных сторов (например, `persist: true`).

#### Главный "подводный камень"

Добавление нереактивных свойств или потеря `this`. Если ты добавляешь в стор объект (например, инстанс роутера), он не должен становиться реактивным (иначе Vue попытается проксировать весь роутер, что вызовет ошибки производительности). Для таких вещей используй `markRaw()` из Vue.

#### Пример кода

Создадим плагин, который делает две вещи:

1. Внедряет глобальный API-клиент во все сторы (чтобы не импортировать его каждый раз).
2. Добавляет поддержку кастомной опции `debounce`, которая автоматически дебаунсит все экшены стора.
```ts
// 1. Определение плагина (plugins/pinia-core-plugin.ts)
import { PiniaPluginContext } from 'pinia'
import { markRaw } from 'vue'
import { apiClient } from '@/api' // Наш axios инстанс
import { debounce } from 'lodash-es'

// Расширяем типы Pinia (Declaration Merging)
declare module 'pinia' {
  // Добавляем свойства в сам инстанс стора
  export interface PiniaCustomProperties {
    $api: typeof apiClient
  }
  // Добавляем кастомные опции в defineStore
  export interface DefineStoreOptionsBase<S, Store> {
    debounceActions?: boolean | number
  }
}

export function corePiniaPlugin({ store, options }: PiniaPluginContext) {
  // 1. Внедрение API клиента
  // Используем markRaw, чтобы Vue не пытался сделать axios реактивным
  store.$api = markRaw(apiClient)

  // 2. Чтение кастомных опций и модификация экшенов
  if (options.debounceActions) {
    const delay = typeof options.debounceActions === 'number' ? options.debounceActions : 300
    
    // Перебираем все свойства стора, ищем экшены (функции)
    // В Options API они лежат в options.actions, в Setup API они просто в объекте store
    Object.keys(store).forEach((key) => {
      const originalMethod = store[key]
      if (typeof originalMethod === 'function' && key !== '$reset' && !key.startsWith('$')) {
        // Оборачиваем оригинальный экшен в debounce
        store[key] = debounce(originalMethod.bind(store), delay)
      }
    })
  }

  // Можно вернуть объект, свойства которого добавятся в стор (альтернатива store.prop = ...)
  // return { $api: markRaw(apiClient) }
}
```

```ts
// 2. Подключение плагина в main.ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import { corePiniaPlugin } from './plugins/pinia-core-plugin'

const pinia = createPinia()
pinia.use(corePiniaPlugin) // Регистрируем глобально

const app = createApp(App)
app.use(pinia)
app.mount('#app')
```

```ts
// 3. Использование в сторе (stores/search.ts)
import { defineStore } from 'pinia'

export const useSearchStore = defineStore('search', {
  state: () => ({ query: '', results: [] }),
  
  // Включаем фичу из нашего плагина!
  debounceActions: 500, 

  actions: {
    async performSearch(newQuery: string) {
      this.query = newQuery
      // Используем внедренный $api (TypeScript знает о нем благодаря declare module)
      const response = await this.$api.get(`/search?q=${newQuery}`)
      this.results = response.data
    }
  }
})
```