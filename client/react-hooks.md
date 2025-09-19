# Популярные хуки React

Хуки группированы по категориям для удобства навигации. Помните, что хуки должны вызываться на верхнем уровне компонента — никогда внутри циклов, условий или вложенных функций.

## Хуки состояния

Эти хуки помогают управлять состоянием компонента, например, отслеживать выборы пользователя или данные формы.

### useState

**Описание:**  
Этот хук позволяет компоненту запоминать простые данные, которые могут изменяться, например, то, что пользователь вводит в форму, или какая вкладка открыта. Идеально для базовых вещей без сложной логики.

**Пример:**  
Представьте простой счетчик в панели управления, где пользователи кликают, чтобы увеличить счет.

```typescript
import { useState } from 'react';

type CounterProps = {
  initialValue?: number;
};

const Counter = ({ initialValue = 0 }: CounterProps): JSX.Element => {
  const [count, setCount] = useState<number>(initialValue);

  const increment = () => setCount(count + 1);

  return (
    <div>
      <p>Текущий счет: {count}</p>
      <button onClick={increment}>Добавить очко</button>
    </div>
  );
};
```

### useReducer

**Описание:**  
Похож на `useState`, но для более сложных изменений состояния, когда несколько действий (например, добавить, удалить, редактировать) влияют на одни и те же данные. Он использует функцию "редьюсер" для определения нового состояния на основе текущего и действия.

**Пример:**  
Список задач, где можно добавлять или переключать задачи, распространенный в приложениях для продуктивности.

```typescript
import { useReducer } from 'react';

type Todo = { id: number; text: string; completed: boolean };
type Action = { type: 'ADD'; text: string } | { type: 'TOGGLE'; id: number };

const todoReducer = (state: Todo[], action: Action): Todo[] => {
  switch (action.type) {
    case 'ADD':
      return [...state, { id: Date.now(), text: action.text, completed: false }];
    case 'TOGGLE':
      return state.map(todo =>
        todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
      );
    default:
      return state;
  }
};

const TodoList = (): JSX.Element => {
  const [todos, dispatch] = useReducer(todoReducer, [] as Todo[]);

  const addTodo = (text: string) => dispatch({ type: 'ADD' as const, text });

  return (
    <div>
      <input
        type="text"
        onKeyDown={(e) => {
          if (e.key === 'Enter') addTodo(e.currentTarget.value);
        }}
        placeholder="Добавить задачу"
      />
      <ul>
        {todos.map(todo => (
          <li key={todo.id} onClick={() => dispatch({ type: 'TOGGLE' as const, id: todo.id })}>
            {todo.text} {todo.completed ? '(выполнено)' : ''}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

## Хуки контекста

Эти хуки помогают делиться данными между компонентами без передачи пропсов везде.

### useContext

**Описание:**  
Этот хук извлекает данные из общего "контекста" (как глобальное хранилище для тем или информации о пользователе) без необходимости передавать их через каждый родительский компонент. Идеально для настроек всего приложения, таких как темный режим.

**Пример:**  
Обмен предпочтениями языка пользователя в многоязычном блоге.

```typescript
import { createContext, useContext } from 'react';

type LanguageContextType = {
  language: 'en' | 'ru';
  setLanguage: (lang: 'en' | 'ru') => void;
};

const LanguageContext = createContext<LanguageContextType | undefined>(undefined);

type LanguageProviderProps = { children: React.ReactNode };

const LanguageProvider = ({ children }: LanguageProviderProps): JSX.Element => {
  // Упрощено: в реальном приложении используйте состояние здесь
  const [language, setLanguage] = React.useState<'en' | 'ru'>('en');
  return (
    <LanguageContext.Provider value={{ language, setLanguage }}>
      {children}
    </LanguageContext.Provider>
  );
};

const Greeting = (): JSX.Element => {
  const { language } = useContext(LanguageContext)!;
  return <h1>{language === 'en' ? 'Hello!' : 'Привет!'}</h1>;
};
```

## Хуки ссылок

Эти хуки создают ссылки на вещи, такие как элементы DOM или значения, которые не вызывают перерендеринг.

### useRef

**Описание:**  
Этот хук создает "ссылку" для хранения значения (часто элемента DOM, такого как поле ввода), которое сохраняется между рендерами без вызова обновления компонента. Полезно для фокусировки ввода или хранения таймеров.

**Пример:**  
Автофокус на поле ввода в строке поиска после клика по кнопке, как в поиске электронного магазина.

```typescript
import { useRef } from 'react';

const SearchBar = (): JSX.Element => {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Поиск продуктов..." />
      <button onClick={focusInput}>Фокус на поиск</button>
    </div>
  );
};
```

## Хуки эффектов

Эти хуки запускают побочные эффекты, такие как получение данных или настройка подписок, после рендеров.

### useEffect

**Описание:**  
Этот хук запускает код после рендера компонента, например, получение данных из API или прослушивание событий. Это как "сделай это после рисования экрана" — очистка происходит при размонтировании компонента. Необходим для всего, что связано с внешним миром, например, загрузка профилей пользователей.

**Пример:**  
Получение и отображение профиля пользователя из API при монтировании компонента, как в социальном приложении.

```typescript
import { useState, useEffect } from 'react';

type User = {
  name: string;
  email: string;
};

type UserProfileProps = { userId: string };

const UserProfile = ({ userId }: UserProfileProps): JSX.Element => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    const fetchUser = async () => {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      setUser(data);
    };
    fetchUser();
  }, [userId]); // Перезапуск, если userId изменится

  return <div>{user ? `Имя: ${user.name}` : 'Загрузка...'}</div>;
};
```

## Хуки производительности

Эти хуки оптимизируют, кэшируя или приоритизируя обновления, чтобы избежать замедлений.

### useMemo

**Описание:**  
Этот хук запоминает результат тяжелого вычисления (например, фильтрация большого списка) и пересчитывает только если входные данные изменились.

**Пример:**  
Фильтрация списка продуктов по категории в онлайн-магазине, чтобы избежать пересчета на каждый keystroke.

```typescript
import { useMemo, useState } from 'react';

type Product = {
  id: number;
  name: string;
  category: string;
};

type ProductListProps = { products: Product[]; filter: string };

const ProductList = ({ products, filter }: ProductListProps): JSX.Element => {
  const filteredProducts = useMemo(() => {
    return products.filter(p => p.category.includes(filter));
  }, [products, filter]);

  return (
    <ul>
      {filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
};
```

### useCallback

**Описание:**  
Этот хук кэширует функцию, чтобы она не создавалась заново при каждом рендере, что помогает при передаче функций дочерним компонентам (предотвращает ненужные перерендеры).

**Пример:**  
Передача обработчика меморизованному дочернему компоненту в списке форм, как редактирование нескольких элементов.

```typescript
import { useCallback, useState } from 'react';
import { memo } from 'react';

type ItemEditorProps = { onUpdate: (id: number, value: string) => void };

const ItemEditor = memo(({ onUpdate }: ItemEditorProps): JSX.Element => (
  <input onChange={(e) => onUpdate(1, e.target.value)} />
));

const Parent = (): JSX.Element => {
  const [value, setValue] = useState('');

  const handleUpdate = useCallback((id: number, newValue: string) => {
    setValue(newValue);
  }, []);

  return <ItemEditor onUpdate={handleUpdate} />;
};
```

### useTransition

**Описание:**  
Этот хук помечает обновления как "низкоприоритетные", чтобы срочные (как ввод) не блокировались. Возвращает флаг для ожидающих переходов. Отлично для плавного UI, как поиск с сохранением отзывчивости ввода.

**Пример:**  
Обновление списка поиска без замораживания поля ввода в строке реального времени поиска.

```typescript
import { useState, useTransition } from 'react';

const Search = (): JSX.Element => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<string[]>([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newQuery = e.target.value;
    setQuery(newQuery);
    startTransition(() => {
      // Симуляция поиска
      setResults(newQuery ? [`Результат для ${newQuery}`] : []);
    });
  };

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Поиск...</p>}
      <ul>{results.map(r => <li key={r}>{r}</li>)}</ul>
    </div>
  );
};
```

## Новые хуки в React 19

Эти хуки фокусируются на формах, асинхронных операциях и оптимистичном UI для лучшего пользовательского опыта.

### use

**Описание:**  
Хук читает ресурсы, такие как промисы (для получения данных) или контекст во время рендера, приостанавливая до готовности. Может использоваться условно, в отличие от других хуков. Это более простой способ получения данных без эффектов — отлично для загрузки комментариев или данных пользователя без boilerplate.

**Пример:**  
Получение и отображение комментариев в посте блога, с приостановкой и запасным вариантом.

```typescript
import { use } from 'react';
import { Suspense } from 'react';

const fetchComments = async (postId: string): Promise<string[]> => {
  // Симуляция API
  return [`Комментарий 1 для поста ${postId}`, `Комментарий 2`];
};

type CommentsProps = { postId: string };

const Comments = ({ postId }: CommentsProps): JSX.Element => {
  const comments = use(fetchComments(postId));
  return (
    <ul>
      {comments.map((comment, index) => <li key={index}>{comment}</li>)}
    </ul>
  );
};

type PostPageProps = { postId: string };

const PostPage = ({ postId }: PostPageProps): JSX.Element => (
  <Suspense fallback={<div>Загрузка комментариев...</div>}>
    <Comments postId={postId} />
  </Suspense>
);
```

### useActionState

**Описание:**  
Хук упрощает обработку форм, оборачивая функцию действия, отслеживая состояния ожидания, ошибки и результаты. Возвращает состояние, привязанное действие и флаг ожидания. Идеально для серверных действий в формах, таких как обновление профиля без ручного управления состоянием.

**Пример:**  
Форма для обновления имени пользователя, показывающая ошибки и отключающая во время отправки.

```typescript
import { useActionState } from 'react';

const updateName = async (_: string | null, formData: FormData): Promise<string | null> => {
  const name = formData.get('name') as string;
  // Симуляция вызова API
  if (name.length < 2) return 'Имя слишком короткое';
  // Успех: вернуть null
  return null;
};

const NameForm = (): JSX.Element => {
  const [error, submitAction, isPending] = useActionState(updateName, null);

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Обновление...' : 'Обновить имя'}
      </button>
      {error && <p style={{ color: 'red' }}>{error}</p>}
    </form>
  );
};
```

### useFormStatus

**Описание:**  
Хук читает статус ближайшей родительской формы (например, ожидание отправки) без передачи пропсов. Возвращает ожидание и данные. Идеально для кастомных кнопок или спиннеров в компонентах форм, таких как отключение кнопки отправки во время загрузки.

**Пример:**  
Кнопка отправки, которая отключается и показывает прогресс на основе статуса формы в форме загрузки файла.

```typescript
import { useFormStatus } from 'react-dom';

const SubmitButton = (): JSX.Element => {
  const { pending, data } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? `Загрузка ${data?.get('file')?.name || ''}...` : 'Загрузить файл'}
    </button>
  );
};

const UploadForm = (): JSX.Element => (
  <form action={async (formData: FormData) => {
    // Симуляция загрузки
    await new Promise(resolve => setTimeout(resolve, 2000));
  }}>
    <input type="file" name="file" />
    <SubmitButton />
  </form>
);
```

### useOptimistic

**Описание:**  
Хук позволяет мгновенно показывать "оптимистичное" обновление UI (например, пометить кнопку лайка активной), пока реальный асинхронный запрос обрабатывается. Если провалится, вернется назад. Увеличивает воспринимаемую скорость в приложениях, таких как социальные ленты, где лайки появляются сразу.

**Пример:**  
Оптимистичное обновление счетчика лайков на посте во время отправки на сервер.

```typescript
import { useOptimistic, useState } from 'react';

const likePost = async (currentLikes: number): Promise<number> => {
  // Симуляция API
  await new Promise(resolve => setTimeout(resolve, 1000));
  return currentLikes + 1;
};

const LikeButton = (): JSX.Element => {
  const [likes, setLikes] = useState(0);
  const [optimisticLikes, setOptimisticLikes] = useOptimistic(likes);

  const handleLike = async () => {
    setOptimisticLikes(likes + 1);
    try {
      const newLikes = await likePost(likes);
      setLikes(newLikes);
    } catch {
      // Автоматически возвращается
    }
  };

  return (
    <button onClick={handleLike} disabled={likes !== optimisticLikes}>
      Лайки: {optimisticLikes} {likes !== optimisticLikes ? '(обновление...)' : ''}
    </button>
  );
};
```
