**Дженерики** — это инструмент, который позволяет создавать переиспользуемые компоненты (функции, классы, интерфейсы), способные работать с **любыми типами данных**, сохраняя при этом строгую типизацию.

**Простая аналогия:** 
Обычная функция принимает *значения* через аргументы. Дженерик принимает *типы* через параметры типов (в угловых скобках `< >`). Это своеобразная «переменная для типа».

#### Главный вопрос на собеседовании: «Зачем нужны дженерики, если есть `any`?»
Если мы используем `any`, мы отключаем проверку типов и теряем информацию о том, что возвращает функция. Дженерики позволяют функции работать с любым типом, **запоминая** этот тип.

```typescript
// ПЛОХО: Использование any
function identityAny(arg: any): any {
  return arg;
}
const result1 = identityAny(42); // Тип result1 — any. Мы потеряли информацию, что это число.

// ХОРОШО: Использование дженерика
function identity<T>(arg: T): T {
  return arg;
}
const result2 = identity(42); // TypeScript понимает, что result2 — это number!
const result3 = identity("hello"); // result3 — это string!
```
*Примечание: `T` — это просто общепринятое сокращение от слова Type. Можно использовать любое имя, например `<ItemType>`.*

---
### Базовое использование дженериков

#### В функциях
Дженерики часто используются для работы с массивами или преобразования данных.

```typescript
// Функция возвращает первый элемент массива любого типа
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const numbers = getFirstElement([1, 2, 3]); // Тип: number | undefined
const strings = getFirstElement(["a", "b"]); // Тип: string | undefined
```

#### В интерфейсах и типах (Type Aliases)
Позволяют создавать гибкие структуры данных.

```typescript
interface Box<T> {
  content: T;
  isEmpty: boolean;
}

const stringBox: Box<string> = { content: "Посылка", isEmpty: false };
const numberBox: Box<number> = { content: 100, isEmpty: false };
```

#### В классах
```typescript
class DataStorage<T> {
  private data: T[] = [];

  addItem(item: T) {
    this.data.push(item);
  }

  getItems(): T[] {
    return [...this.data];
  }
}

const textStorage = new DataStorage<string>();
textStorage.addItem("Привет");
// textStorage.addItem(10); // Ошибка TypeScript! Ожидается строка.
```

---
### Продвинутые концепции

#### Несколько параметров типа
Вы можете передавать столько типов, сколько нужно. Обычно их называют `T`, `U`, `V`, `K` (Key), `V` (Value).

```typescript
function mergeObjects<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const merged = mergeObjects({ name: "Ivan" }, { age: 25 });
// Тип merged: { name: string } & { age: number }
```

#### Ограничения дженериков (Constraints - `extends`)
Иногда нам нужно, чтобы дженерик принимал не *абсолютно любой* тип, а только тот, который имеет определенные свойства. Для этого используется ключевое слово `extends`.

```typescript
interface HasLength {
  length: number;
}

// T обязан иметь свойство length
function logLength<T extends HasLength>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("Hello"); // Работает (у строк есть length)
logLength([1, 2, 3]); // Работает (у массивов есть length)
// logLength(42); // ОШИБКА! У чисел нет свойства length
```

#### Использование параметров типа в ограничениях (`keyof`)
Это классический пример с собеседований. Как написать функцию, которая безопасно достает значение из объекта по ключу?

```typescript
// K обязано быть одним из ключей объекта T
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

const user = { name: "Alice", age: 30 };

const userName = getProperty(user, "name"); // Работает, тип string
// const invalid = getProperty(user, "email"); // ОШИБКА! "email" нет в ключах user
```

#### Значения по умолчанию
Как и у обычных аргументов функций, у дженериков могут быть типы по умолчанию.

```typescript
interface ApiResponse<Data = null> {
  status: number;
  data: Data;
}

const emptyRes: ApiResponse = { status: 200, data: null }; // Data по умолчанию null
const userRes: ApiResponse<{ name: string }> = { status: 200, data: { name: "Bob" } };
```

---

### Реальные примеры из Frontend-разработки

На практике во фронтенде дженерики чаще всего встречаются в двух местах: при работе с API и в переиспользуемых UI-компонентах (например, в React).

#### Пример 1: Типизация ответов от сервера (Axios / Fetch)
Обычно бэкенд возвращает данные в единой обертке. Дженерики идеально подходят для типизации поля `data`.

```typescript
// Единый интерфейс для всех ответов сервера
interface ServerResponse<T> {
  success: boolean;
  message: string;
  payload: T;
}

interface User {
  id: number;
  username: string;
}

interface Article {
  title: string;
  content: string;
}

// Используем:
async function fetchUser(): Promise<ServerResponse<User>> {
  const response = await fetch('/api/user');
  return response.json(); 
  // Теперь TS знает, что response.payload — это объект User
}
```

#### Пример 2: Переиспользуемые React-компоненты
Представьте, что вы пишете компонент `Select` (выпадающий список) или `Table`. Опции могут быть строками, числами или сложными объектами.

```tsx
import React from 'react';

// Пропсы компонента принимают дженерик T
interface SelectProps<T> {
  options: T[];
  value: T;
  onChange: (value: T) => void;
  renderOption: (option: T) => React.ReactNode;
}

// Компонент-дженерик
function Select<T>(props: SelectProps<T>) {
  return (
    <div>
      {props.options.map((option, index) => (
        <div key={index} onClick={() => props.onChange(option)}>
          {props.renderOption(option)}
        </div>
      ))}
    </div>
  );
}

// Использование:
const App = () => {
  const users = [{ id: 1, name: 'Anna' }, { id: 2, name: 'Boris' }];
  
  return (
    <Select
      options={users}
      value={users[0]}
      onChange={(user) => console.log(user.id)} // TS знает, что user имеет тип {id, name}
      renderOption={(user) => <span>{user.name}</span>}
    />
  );
};
```

---

### Чек-лист для повторения перед собеседованием:
1. **Что это?** Способ передавать типы как параметры.
2. **Зачем?** Переиспользование кода + строгая типизация (в отличие от `any`).
3. **Синтаксис:** `<T>` после имени функции/класса/интерфейса.
4. **Ограничения:** `extends` (например, `<T extends HTMLElement>`).
5. **Связь типов:** Использование `keyof` (`<T, K extends keyof T>`).
6. **Утилиты TS:** Помните, что встроенные утилиты типа `Partial<T>`, `Pick<T, K>`, `ReturnType<T>` — это всё дженерики "под капотом".