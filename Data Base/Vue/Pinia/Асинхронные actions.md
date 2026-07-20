#### Архитектурная теория

В Redux для асинхронности нужны middleware (Thunk, Saga), потому что редьюсеры обязаны быть чистыми синхронными функциями. Во Vuex были `actions` для асинхронности и `mutations` для синхронного изменения стейта, что создавало много бойлерплейта.

В Pinia нет разницы между синхронными и асинхронными экшенами с точки зрения архитектуры.

- _Как это работает:_ Экшен в Pinia — это просто метод объекта (или функция в Setup-сторе). Если ты помечаешь его как `async`, он возвращает `Promise`. Pinia не вмешивается в Event Loop браузера.
- _Сохранение контекста:_ В Options-стиле Pinia использует `Proxy` для привязки контекста. Когда ты пишешь `await apiCall()`, а затем обращаешься к `this.user = data`, контекст `this` гарантированно указывает на инстанс стора, потому что стрелочные функции или методы объекта внутри `Proxy` сохраняют ссылку на сам `Proxy`. Тебе не нужно передавать `commit` или `dispatch` внутрь коллбэков.

#### Аналогия с MobX / React

- MobX: В старых версиях MobX асинхронные экшены были болью. После `await` контекст выполнения выходил из обертки `@action`, и прямая мутация стейта вызывала предупреждения (нужно было использовать `runInAction`). В современных версиях MobX это решается через генераторы (`flow`) или автоматическое оборачивание. В Pinia этой проблемы вообще нет. Ты просто пишешь `await` и мутируешь стейт на следующей строке — система реактивности Vue (Proxy) перехватит это изменение независимо от того, в каком микротаске оно произошло.
- React: Это похоже на обычную асинхронную функцию внутри компонента, которая вызывает `setState` после `await`. Разница лишь в том, что в Pinia это состояние глобальное.

#### Best Practices & TypeScript

- Обработка ошибок: Так как экшен возвращает обычный Promise, ты можешь обрабатывать ошибки прямо в компоненте (`try/catch` вокруг вызова экшена) или внутри самого экшена, сохраняя ошибку в стейт (например, `state.error = err.message`).
- Состояние загрузки (Loading state): Управляй флагами загрузки (`isLoading`) прямо внутри экшена.
- Отмена запросов (AbortController): Pinia не предоставляет встроенного механизма отмены запросов (как Redux Saga). Если нужно отменить предыдущий запрос при повторном вызове экшена, используй стандартный `AbortController` внутри экшена.

#### Главный "подводный камень"

Потеря реактивности при деструктуризации ответа API. Это скорее нюанс JavaScript, но он часто встречается в асинхронных экшенах. Если ты получаешь объект из API и хочешь обновить реактивный объект в стейте, нельзя просто переназначить ссылку, если ты до этого деструктурировал стейт в компоненте.
```ts
// ❌ ПЛОХО (если user - это реактивный объект, а не ref)
async fetchUser() {
  const data = await api.get()
  this.user = data // Ломает ссылку, если компонент уже подписался на старый объект
}

// ✅ ХОРОШО
async fetchUser() {
  const data = await api.get()
  Object.assign(this.user, data) // Мутируем существующий объект
  // ИЛИ используем ref в Setup-сторе: user.value = data
}
```
#### Пример кода

Покажу production-ready пример с обработкой состояний (загрузка, ошибка) и паттерном раннего возврата (чтобы не делать лишних запросов).
```ts
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { api } from '@/api' // Условный axios-клиент

// Типы для домена
interface UserProfile {
  id: string
  name: string
  email: string
}

export const useProfileStore = defineStore('profile', () => {
  // --- State ---
  const profile = ref<UserProfile | null>(null)
  const isLoading = ref(false)
  const error = ref<string | null>(null)

  // --- Async Actions ---
  async function fetchProfile(userId: string, force = false) {
    // Паттерн: избегаем лишних запросов, если данные уже есть
    if (profile.value?.id === userId && !force) {
      return profile.value
    }

    isLoading.value = true
    error.value = null

    try {
      // Обычный async/await
      const response = await api.get<UserProfile>(`/users/${userId}`)
      
      // Мутируем стейт после await. 
      // Vue 3 Proxy без проблем перехватит это изменение.
      profile.value = response.data
      
      return response.data // Можно возвращать данные для использования в компоненте
    } catch (err: any) {
      // Обработка ошибок
      error.value = err.response?.data?.message || 'Не удалось загрузить профиль'
      throw err // Пробрасываем ошибку дальше, если компонент хочет ее обработать
    } finally {
      isLoading.value = false
    }
  }

  async function updateProfile(data: Partial<UserProfile>) {
    try {
      isLoading.value = true
      const response = await api.patch(`/users/${profile.value?.id}`, data)
      
      // Обновляем только часть стейта
      if (profile.value) {
        Object.assign(profile.value, response.data)
      }
    } catch (err) {
      console.error('Update failed', err)
      throw err
    } finally {
      isLoading.value = false
    }
  }

  return {
    profile,
    isLoading,
    error,
    fetchProfile,
    updateProfile
  }
})
```

```ts
<!-- Пример использования в компоненте -->
<script setup lang="ts">
import { useProfileStore } from '@/stores/profile'

const profileStore = useProfileStore()

const loadData = async () => {
  try {
    // Ждем выполнения экшена прямо в компоненте
    await profileStore.fetchProfile('123')
  } catch (e) {
    // Ошибка уже лежит в profileStore.error, но мы можем показать toast здесь
    showToast('Ошибка загрузки')
  }
}
</script>
```