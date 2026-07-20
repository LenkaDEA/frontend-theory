**Utility Types (Утилитарные типы)** — это встроенные в TypeScript дженерики, которые позволяют трансформировать уже существующие типы, чтобы не писать их заново (принцип DRY — Don't Repeat Yourself).

На собеседованиях часто проверяют не только знание этих утилит, но и понимание разницы между ними (например, чем `Omit` отличается от `Exclude`).

Давай разберем запрошенные тобой утилиты, а также добавим несколько других, которые обязательно нужно знать.

---

### 1. Работа со свойствами объектов (Интерфейсов)

Эти утилиты используются, когда у вас есть объектный тип (интерфейс), и вы хотите создать на его основе новый.

#### 🔹 `Partial<T>`
Делает **все** свойства типа `T` необязательными (добавляет `?`).
*Частый юзкейс:* Обновление данных (PATCH-запросы), когда мы передаем только часть полей объекта.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// Теперь все поля необязательные: { id?: number; name?: string; email?: string; }
type UserUpdate = Partial<User>;

const updateUser = (id: number, data: UserUpdate) => {
  // data может содержать только name, только email или всё вместе
};

updateUser(1, { name: "Ivan" }); // Ошибки нет, email и id не требуются
```

#### 🔹 `Pick<T, K>`
Создает новый тип, **выбирая** только указанные свойства `K` из типа `T`.
*Частый юзкейс:* Создание "превью" объекта или урезанной версии для UI.

```typescript
interface Article {
  id: number;
  title: string;
  content: string;
  authorId: number;
  createdAt: Date;
}

// Берем только title и content
type ArticlePreview = Pick<Article, "title" | "content">;

const preview: ArticlePreview = {
  title: "Как выучить TS",
  content: "Просто пишите код..."
  // id или createdAt сюда добавить нельзя
};
```

#### 🔹 `Omit<T, K>`
Противоположность `Pick`. Создает новый тип, **исключая** указанные свойства `K` из типа `T`.
*Частый юзкейс:* Создание типа для формы создания сущности, где ID и дата генерируются на сервере.

```typescript
interface User {
  id: number;
  name: string;
  role: string;
  createdAt: Date;
}

// Исключаем id и createdAt
type UserCreateDTO = Omit<User, "id" | "createdAt">;

const newUser: UserCreateDTO = {
  name: "Alice",
  role: "admin"
};
```

---

### 2. Работа со структурами данных (Словари)

#### 🔹 `Record<K, T>`
Создает объектный тип, где ключами являются `K`, а значениями — `T`.
*Частый юзкейс:* Создание словарей (map), объектов конфигурации.

```typescript
type UserRole = "admin" | "user" | "guest";

// Ключи — это UserRole, значения — булевы (есть доступ или нет)
type RolePermissions = Record<UserRole, boolean>;

const permissions: RolePermissions = {
  admin: true,
  user: true,
  guest: false,
};

// Другой пример: словарь пользователей по их ID (строка)
const usersById: Record<string, User> = {
  "uuid-1": { id: 1, name: "Bob", /* ... */ },
  "uuid-2": { id: 2, name: "Alice", /* ... */ }
};
```

---

### 3. Работа с Union типами (Объединениями)

⚠️ **Важный вопрос с собеседования:** *"Чем `Omit` отличается от `Exclude`?"*
**Ответ:** `Omit` работает с ключами **объектов** (интерфейсов), а `Exclude` работает с **Union-типами** (объединениями `A | B | C`).

#### 🔹 `Exclude<T, U>`
Исключает из Union-типа `T` те типы, которые совместимы с `U`.

```typescript
type Status = "idle" | "loading" | "success" | "error";

// Убираем "loading" и "idle"
type FinishedStatus = Exclude<Status, "loading" | "idle">; 
// Результат: "success" | "error"
```

#### 🔹 `Extract<T, U>` (Дополнительно, пара к Exclude)
Противоположность `Exclude`. Оставляет в Union-типе `T` **только** те типы, которые совместимы с `U`.

```typescript
type MixedTypes = string | number | boolean | string[];

// Оставляем только строки и массивы строк
type OnlyStrings = Extract<MixedTypes, string | string[]>;
// Результат: string | string[]
```

---

### 4. Работа с промисами и функциями

#### 🔹 `Awaited<T>`
"Распаковывает" тип из `Promise`. Имитирует работу оператора `await` на уровне типов.
*Частый юзкейс:* Получение типа данных, которые возвращает асинхронная функция (часто используется вместе с `ReturnType`).

```typescript
// Допустим, у нас есть тип промиса
type PromiseString = Promise<string>;

// Awaited достанет то, что внутри
type Result = Awaited<PromiseString>; // Результат: string

// Работает даже с вложенными промисами
type DeepPromise = Promise<Promise<number>>;
type DeepResult = Awaited<DeepPromise>; // Результат: number
```

#### 🔹 `ReturnType<T>` (Обязательно нужно знать!)
Достает тип, который **возвращает** функция `T`.
*Частый юзкейс:* Когда вы используете стороннюю библиотеку, которая экспортирует функцию, но не экспортирует тип её результата.

```typescript
function createUser(name: string, age: number) {
  return {
    id: Math.random(),
    name,
    age,
    isActive: true
  };
}

// typeof createUser берет тип самой функции
// ReturnType достает из нее то, что она возвращает
type NewUser = ReturnType<typeof createUser>;
/*
Результат: {
  id: number;
  name: string;
  age: number;
  isActive: boolean;
}
*/
```

---

### 5. Еще пара важных утилит для собеседования

#### 🔹 `Required<T>`
Противоположность `Partial`. Делает все свойства объекта **обязательными** (убирает `?`).

```typescript
interface Props {
  title?: string;
  count?: number;
}

type StrictProps = Required<Props>;
// Результат: { title: string; count: number; }
```

#### 🔹 `NonNullable<T>`
Удаляет `null` и `undefined` из Union-типа.

```typescript
type PossibleValue = string | number | null | undefined;

type DefiniteValue = NonNullable<PossibleValue>;
// Результат: string | number
```

---

### 💡 Шпаргалка для собеседования:
*   Нужно сделать все свойства необязательными? 👉 `Partial`
*   Нужно собрать объект из пары свойств другого объекта? 👉 `Pick`
*   Нужно выкинуть пару свойств из объекта? 👉 `Omit`
*   Нужно типизировать объект-словарь? 👉 `Record`
*   Нужно выкинуть часть из типа `A | B | C`? 👉 `Exclude`
*   Нужно узнать, что возвращает функция? 👉 `ReturnType`
*   Нужно узнать, что лежит внутри Promise? 👉 `Awaited`