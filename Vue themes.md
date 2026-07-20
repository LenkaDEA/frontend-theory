[Песочница](https://play.vuejs.org/)
# Основы
- [Компонентный подход](Data%20Base/Vue/Основы%20Vue/Компонентный%20подход.md), умные и глупые компоненты
- [Options API vs Composition API](Data%20Base/Vue/Основы%20Vue/Options%20API%20vs%20Composition%20API.md)
- [Жизненный цикл компонента](Data%20Base/Vue/Основы%20Vue/Жизненный%20цикл%20компонента.md)(onMounted, onUpdated, onUnmounted и др.)
- [template-синтаксис](Data%20Base/Vue/Основы%20Vue/template-синтаксис.md): директивы v-if, v-show, v-for, v-bind, v-on, v-model
- [Вычисляемые свойства (computed)](Data%20Base/Vue/Основы%20Vue/Вычисляемые%20свойства%20(computed).md)
- [Наблюдатели (watch, watchEffect)](Data%20Base/Vue/Основы%20Vue/Наблюдатели%20(watch,%20watchEffect).md)
- [Реактивность](Data%20Base/Vue/Основы%20Vue/Реактивность.md): ref, reactive, shallowRef, shallowReactive, toRef, toRefs
- [Пропсы и эмиты](Data%20Base/Vue/Основы%20Vue/Пропсы%20и%20эмиты.md) (defineProps, defineEmits)
- [Слоты](Data%20Base/Vue/Основы%20Vue/Слоты.md) (default, именованные, scoped slots)
- [Provide Inject](Data%20Base/Vue/Основы%20Vue/Provide%20Inject.md)
- [Teleport](Data%20Base/Vue/Основы%20Vue/Teleport.md)
- [Suspense](Data%20Base/Vue/Основы%20Vue/Suspense.md)
- [Динамические компоненты](Data%20Base/Vue/Основы%20Vue/Динамические%20компоненты.md) (component :is)
- [Переходы и анимации](Data%20Base/Vue/Основы%20Vue/Переходы%20и%20анимации.md) (Transition, TransitionGroup)

# Composables
- [Что такое composable и зачем нужен](Data%20Base/Vue/Composables/Что%20такое%20composable%20и%20зачем%20нужен.md)
- [Написание переиспользуемых composable-функций](Data%20Base/Vue/Composables/Написание%20переиспользуемых%20composable-функций.md)
- [Отличие от миксинов](Data%20Base/Vue/Composables/Отличие%20от%20миксинов.md)

# Оптимизация
- [Директивы v-once и v-memo](Data%20Base/Vue/Оптимизация/Директивы%20v-once%20и%20v-memo.md)
- [shallowRef и shallowReactive](Data%20Base/Vue/Оптимизация/shallowRef%20и%20shallowReactive.md)
- [markRaw](Data%20Base/Vue/Оптимизация/markRaw.md)
- [Когда использовать computed вместо метода](Data%20Base/Vue/Оптимизация/Когда%20использовать%20computed%20вместо%20метода.md)

# Pinia
- [Принцип работы, отличие от Vuex](Data%20Base/Vue/Pinia/Принцип%20работы,%20отличие%20от%20Vuex.md)
- [defineStore Options-стиль vs Setup-стиль](Data%20Base/Vue/Pinia/defineStore%20Options-стиль%20vs%20Setup-стиль.md)
- [State, Getters, Actions](Data%20Base/Vue/Pinia/State,%20Getters,%20Actions.md)
- [Асинхронные actions](Data%20Base/Vue/Pinia/Асинхронные%20actions.md)
- [Подписки (`store.$subscribe, store.$onAction`)](Data%20Base/Vue/Pinia/Подписки%20(`store.$subscribe,%20store.$onAction`).md)
- [Плагины Pinia](Data%20Base/Vue/Pinia/Плагины%20Pinia.md)
- [Разбиение стора на модули](Data%20Base/Vue/Pinia/Разбиение%20стора%20на%20модули.md)


# Vue Router
- [[Настройка роутера, режимы history и hash]]
- [Динамические маршруты и параметры (id)](Data%20Base/Vue/Vue%20Router/Динамические%20маршруты%20и%20параметры%20(id).md)
- [[Вложенные маршруты (nested routes)]]
- [[Программная навигация (router.push, router.replace)]]
- [Navigation guards (beforeEach, beforeEnter, beforeRouteLeave)](Data%20Base/Vue/Vue%20Router/Navigation%20guards%20(beforeEach,%20beforeEnter,%20beforeRouteLeave).md)
- [Lazy loading маршрутов](Data%20Base/Vue/Vue%20Router/Lazy%20loading%20маршрутов.md)
- [Именованные маршруты и именованные views](Data%20Base/Vue/Vue%20Router/Именованные%20маршруты%20и%20именованные%20views.md)
- [router-view, router-link](Data%20Base/Vue/Vue%20Router/router-view,%20router-link.md)
- [Передача пропсов через маршруты](Data%20Base/Vue/Vue%20Router/Передача%20пропсов%20через%20маршруты.md)
- [Активные классы](Data%20Base/Vue/Vue%20Router/Активные%20классы.md)

# Шаблонные refs и взаимодействие с DOM
- [ref для доступа к DOM-элементам и дочерним компонентам](Data%20Base/Vue/Шаблонные%20refs%20и%20взаимодействие%20с%20DOM/ref%20для%20доступа%20к%20DOM-элементам%20и%20дочерним%20компонентам.md)
- [defineExpose](Data%20Base/Vue/Шаблонные%20refs%20и%20взаимодействие%20с%20DOM/defineExpose.md)
- [useTemplateRef (Vue 3.5+)](Data%20Base/Vue/Шаблонные%20refs%20и%20взаимодействие%20с%20DOM/useTemplateRef%20(Vue%203.5+).md)

# Работа с формами
- [v-model на кастомных компонентах](Data%20Base/Vue/Работа%20с%20формами/v-model%20на%20кастомных%20компонентах.md)
- [defineModel (Vue 3.4+)](Data%20Base/Vue/Работа%20с%20формами/defineModel%20(Vue%203.4+).md)
- [Валидация форм](Data%20Base/Vue/Работа%20с%20формами/Валидация%20форм.md)
# Производительность и архитектура
- [Разделение кода (code splitting) через динамические импорты](Data%20Base/Vue/Производительность%20и%20архитектура/Разделение%20кода%20(code%20splitting)%20через%20динамические%20импорты.md)
- [defineAsyncComponent](Data%20Base/Vue/Производительность%20и%20архитектура/defineAsyncComponent.md)
- [SSR (базовое понимание)](Data%20Base/Vue/Производительность%20и%20архитектура/SSR%20(базовое%20понимание).md)
