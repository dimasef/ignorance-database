# GRASP Patterns

**GRASP** (General Responsibility Assignment Software Patterns) — a set of 9 principles for deciding _which object should be responsible for what_. Introduced by Craig Larman in _Applying UML and Patterns_ (1997).

---

## Pattern 1 — Information Expert

**Problem:**

_EN:_ Which object should handle a particular operation or own a piece of logic?

_UA:_ Який об'єкт повинен виконувати певну операцію або володіти частиною логіки?

**Description:**

_EN:_ Assign responsibility to the class that has the information needed to fulfil it. Don't move data to logic — move logic to data.

_UA:_ Призначай відповідальність класу, який має інформацію, необхідну для її виконання. Не переміщуй дані до логіки — переміщуй логіку до даних.

**Relation to SOLID / OOP:**

_EN:_

- Single Responsibility Principle — each class handles what it knows about
- Encapsulation — internal data stays inside the class

_UA:_

- Single Responsibility Principle — кожен клас обробляє те, що стосується його власних даних
- Encapsulation — внутрішні дані залишаються всередині класу

**Example:**

```tsx
// Bad: external component calculates the total
function Cart({ items }: { items: CartItem[] }) {
  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  return <div>{total}</div>;
}

// Good: CartItem knows how to calculate its own subtotal
class CartItem {
  constructor(
    public price: number,
    public quantity: number,
  ) {}
  subtotal() {
    return this.price * this.quantity;
  }
}

function Cart({ items }: { items: CartItem[] }) {
  const total = items.reduce((sum, item) => sum + item.subtotal(), 0);
  return <div>{total}</div>;
}
```

**Weaknesses:**

_EN:_

- Can lead to bloated model classes if taken too literally — every bit of logic ends up on the entity
- May conflict with separation of concerns when business logic leaks into data models

_UA:_

- Може призвести до роздутих класів моделі, якщо застосовувати буквально — вся логіка осідає на сутності
- Може конфліктувати з розділенням відповідальностей, коли бізнес-логіка просочується в моделі даних

---

## Pattern 2 — Creator

**Problem:**

_EN:_ Who should be responsible for creating an instance of a given class?

_UA:_ Хто повинен відповідати за створення екземпляра певного класу?

**Description:**

_EN:_ Assign creation responsibility to the class that aggregates, contains, closely uses, or has the initialisation data for the new object.

_UA:_ Призначай відповідальність за створення класу, який агрегує, містить, активно використовує або має дані ініціалізації нового об'єкта.

**Relation to SOLID / OOP:**

_EN:_

- Single Responsibility Principle — creation logic lives where it naturally belongs
- Dependency Inversion Principle — avoids scattered `new` calls across unrelated classes

_UA:_

- Single Responsibility Principle — логіка створення живе там, де їй природно бути
- Dependency Inversion Principle — уникає розкиданих викликів `new` по непов'язаних класах

**Example:**

```tsx
// Bad: Order creates its own line items from raw data in an unrelated place
function Checkout({ data }: { data: any }) {
  const items = data.lines.map((l: any) => new OrderLineItem(l.id, l.qty, l.price));
  // ...
}

// Good: Order is the natural creator of its own line items
class Order {
  lines: OrderLineItem[] = [];

  addLine(id: string, qty: number, price: number) {
    this.lines.push(new OrderLineItem(id, qty, price));
  }
}
```

**Weaknesses:**

_EN:_

- Can tightly couple the creator to the created class — changes to the constructor ripple upward
- Doesn't scale well when creation logic is complex; factory or builder patterns are better there

_UA:_

- Може тісно зв'язати творця зі створюваним класом — зміни конструктора поширюються вгору
- Погано масштабується, коли логіка створення складна; там краще підходять фабрика або будівельник

---

## Pattern 3 — Controller

**Problem:**

_EN:_ Who should receive and handle a system event (user action, API call, etc.) outside of the UI layer?

_UA:_ Хто повинен отримувати та обробляти системну подію (дія користувача, API-виклик тощо) поза межами шару UI?

**Description:**

_EN:_ Assign event handling to a non-UI object that represents either the overall system or a use-case scenario. The controller delegates to domain objects — it doesn't contain business logic itself.

_UA:_ Призначай обробку подій не-UI об'єкту, який представляє або всю систему, або сценарій використання. Контролер делегує доменним об'єктам — він сам не містить бізнес-логіки.

**Relation to SOLID / OOP:**

_EN:_

- Single Responsibility Principle — UI stays dumb; logic lives in a dedicated handler
- Separation of concerns — presentation and domain are decoupled

_UA:_

- Single Responsibility Principle — UI залишається простим; логіка живе у виділеному обробнику
- Separation of concerns — представлення та домен розділені

**Example:**

```tsx
// Bad: business logic inside a React component
function LoginForm() {
  async function handleSubmit(email: string, password: string) {
    const user = await db.find(email);
    if (!user || !bcrypt.compare(password, user.hash)) throw new Error("Invalid");
    session.set(user);
  }
  // ...
}

// Good: component delegates to a controller
class AuthController {
  async login(email: string, password: string) {
    const user = await this.userRepo.findByEmail(email);
    if (!user || !this.hasher.verify(password, user.hash)) throw new Error("Invalid");
    this.session.set(user);
  }
}

function LoginForm({ controller }: { controller: AuthController }) {
  async function handleSubmit(email: string, password: string) {
    await controller.login(email, password);
  }
  // ...
}
```

**Weaknesses:**

_EN:_

- Can become a "bloated controller" if too many use cases are funnelled through one class
- In simple apps adds an unnecessary layer of indirection

_UA:_

- Може перетворитися на "роздутий контролер", якщо через нього проходить забагато сценаріїв
- У простих застосунках додає непотрібний рівень непрямості

---

## Pattern 4 — Low Coupling

**Problem:**

_EN:_ How to reduce the impact of changes? How to keep classes independent and reusable?

_UA:_ Як зменшити вплив змін? Як зберегти класи незалежними та придатними для повторного використання?

**Description:**

_EN:_ Assign responsibilities so that dependencies between classes stay minimal. A class with low coupling can change without breaking others.

_UA:_ Призначай відповідальності так, щоб залежності між класами були мінімальними. Клас із низьким зв'язуванням може змінюватися, не ламаючи інших.

**Relation to SOLID / OOP:**

_EN:_

- Dependency Inversion Principle — depend on abstractions, not concretions
- Open/Closed Principle — low-coupled classes are easier to extend without modification

_UA:_

- Dependency Inversion Principle — залежи від абстракцій, а не від конкретних реалізацій
- Open/Closed Principle — класи з низьким зв'язуванням легше розширювати без модифікацій

**Example:**

```tsx
// Bad: component is tightly coupled to a specific fetch implementation
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then(setUser);
  }, [userId]);
  // ...
}

// Good: component depends on an abstraction
interface UserService {
  getUser(id: string): Promise<User>;
}

function UserProfile({ userId, userService }: { userId: string; userService: UserService }) {
  const [user, setUser] = useState<User | null>(null);
  useEffect(() => {
    userService.getUser(userId).then(setUser);
  }, [userId]);
  // ...
}
```

**Weaknesses:**

_EN:_

- Taken too far it produces excessive abstraction — interfaces for everything, even trivial utilities
- Low coupling between classes can hide high coupling at the module or package level

_UA:_

- У крайньому вираженні породжує надмірну абстракцію — інтерфейси для всього, навіть для тривіальних утиліт
- Низьке зв'язування між класами може приховувати високе зв'язування на рівні модулів або пакетів

---

## Pattern 5 — High Cohesion

**Problem:**

_EN:_ How to keep objects focused, understandable, and manageable?

_UA:_ Як тримати об'єкти зфокусованими, зрозумілими та керованими?

**Description:**

_EN:_ Assign responsibilities so that a class does one closely related set of things. A highly cohesive class is easy to understand, test, and reuse.

_UA:_ Призначай відповідальності так, щоб клас виконував один пов'язаний набір дій. Клас із високою згуртованістю легко розуміти, тестувати та повторно використовувати.

**Relation to SOLID / OOP:**

_EN:_

- Single Responsibility Principle — one reason to change
- Encapsulation — related behaviour and data stay together

_UA:_

- Single Responsibility Principle — одна причина для зміни
- Encapsulation — пов'язана поведінка та дані залишаються разом

**Example:**

```tsx
// Bad: one hook fetches, formats, and logs — three unrelated concerns
function useUserData(id: string) {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch(`/api/users/${id}`)
      .then((r) => r.json())
      .then((u) => {
        console.log("fetched", u); // logging
        setUser({ ...u, name: u.name.trim() }); // formatting
      });
  }, [id]);
  return user;
}

// Good: each hook does one thing
function useUser(id: string) {
  const [user, setUser] = useState<User | null>(null);
  useEffect(() => {
    userService.getUser(id).then(setUser);
  }, [id]);
  return user;
}

function formatUser(user: User): User {
  return { ...user, name: user.name.trim() };
}
```

**Weaknesses:**

_EN:_

- Finding the right boundary is subjective — two developers will split cohesion differently
- Overly fine-grained cohesion creates many tiny classes that are hard to navigate

_UA:_

- Знайти правильну межу суб'єктивно — два розробники поділять згуртованість по-різному
- Надто дрібнозерниста згуртованість створює багато крихітних класів, у яких важко орієнтуватися

---

## Pattern 6 — Polymorphism

**Problem:**

_EN:_ How to handle behaviour that varies by type without a chain of `if/else` or `switch`?

_UA:_ Як обробляти поведінку, яка варіюється залежно від типу, без ланцюжка `if/else` або `switch`?

**Description:**

_EN:_ When behaviour varies based on type, assign the responsibility to the types themselves using polymorphic operations (interfaces, abstract classes, function overloads).

_UA:_ Коли поведінка варіюється залежно від типу, призначай відповідальність самим типам через поліморфні операції (інтерфейси, абстрактні класи, перевантаження функцій).

**Relation to SOLID / OOP:**

_EN:_

- Open/Closed Principle — new types can be added without changing existing code
- Liskov Substitution Principle — subtypes are substitutable for the base type

_UA:_

- Open/Closed Principle — нові типи можна додавати без зміни існуючого коду
- Liskov Substitution Principle — підтипи є взаємозамінними з базовим типом

**Example:**

```tsx
// Bad: switch on type in a component
function Notification({ type, message }: { type: string; message: string }) {
  if (type === "error") return <div className="error">{message}</div>;
  if (type === "success") return <div className="success">{message}</div>;
  return <div>{message}</div>;
}

// Good: each type knows how to render itself
interface NotificationStrategy {
  render(message: string): React.ReactNode;
}

const strategies: Record<string, NotificationStrategy> = {
  error: { render: (msg) => <div className="error">{msg}</div> },
  success: { render: (msg) => <div className="success">{msg}</div> },
  info: { render: (msg) => <div className="info">{msg}</div> },
};

function Notification({ type, message }: { type: string; message: string }) {
  return <>{(strategies[type] ?? strategies.info).render(message)}</>;
}
```

**Weaknesses:**

_EN:_

- Adds indirection — the concrete type that runs is not obvious at the call site
- Overkill for simple two-branch conditions that will never grow

_UA:_

- Додає непрямість — конкретний тип, що виконується, не очевидний у місці виклику
- Надмірне ускладнення для простих двогілкових умов, які ніколи не виростуть

---

## Pattern 7 — Pure Fabrication

**Problem:**

_EN:_ Where to put behaviour that doesn't naturally belong to any domain object, but must exist somewhere?

_UA:_ Де розмістити поведінку, яка природно не належить жодному доменному об'єкту, але десь повинна існувати?

**Description:**

_EN:_ Create an artificial class (not from the domain model) to keep domain objects cohesive. Typical examples: repositories, services, formatters, adapters.

_UA:_ Створи штучний клас (не з доменної моделі), щоб зберегти доменні об'єкти згуртованими. Типові приклади: репозиторії, сервіси, форматтери, адаптери.

**Relation to SOLID / OOP:**

_EN:_

- Single Responsibility Principle — domain objects stay clean; infrastructure concerns move elsewhere
- Dependency Inversion Principle — fabricated classes are usually behind interfaces

_UA:_

- Single Responsibility Principle — доменні об'єкти залишаються чистими; інфраструктурні турботи виносяться окремо
- Dependency Inversion Principle — сфабриковані класи зазвичай стоять за інтерфейсами

**Example:**

```tsx
// Bad: User entity saves itself — mixes domain and persistence
class User {
  constructor(
    public id: string,
    public email: string,
  ) {}
  async save() {
    await db.query("INSERT INTO users ...", [this.id, this.email]);
  }
}

// Good: repository is a pure fabrication that handles persistence
class User {
  constructor(
    public id: string,
    public email: string,
  ) {}
}

class UserRepository {
  async save(user: User) {
    await db.query("INSERT INTO users ...", [user.id, user.email]);
  }
}
```

**Weaknesses:**

_EN:_

- Can proliferate into dozens of thin service classes with no clear ownership
- Sometimes a sign that the domain model was designed too anemic from the start

_UA:_

- Може розростися до десятків тонких сервісних класів без чіткої відповідальності
- Іноді є ознакою того, що доменна модель з самого початку спроєктована надто анемічною

---

## Pattern 8 — Indirection

**Problem:**

_EN:_ How to decouple two components that shouldn't know about each other directly?

_UA:_ Як роз'єднати два компоненти, які не повинні знати один про одного безпосередньо?

**Description:**

_EN:_ Introduce an intermediate object to mediate between them. The intermediary absorbs the coupling so neither side depends on the other directly.

_UA:_ Введи проміжний об'єкт для посередництва між ними. Посередник поглинає зв'язування, щоб жодна зі сторін не залежала від іншої безпосередньо.

**Relation to SOLID / OOP:**

_EN:_

- Dependency Inversion Principle — both sides depend on the intermediary's abstraction
- Open/Closed Principle — either side can change independently

_UA:_

- Dependency Inversion Principle — обидві сторони залежать від абстракції посередника
- Open/Closed Principle — будь-яка зі сторін може змінюватися незалежно

**Example:**

```tsx
// Bad: UI component calls the API directly — tightly coupled to fetch details
function ProductCard({ id }: { id: string }) {
  const [product, setProduct] = useState<Product | null>(null);
  useEffect(() => {
    fetch(`https://api.example.com/v2/products/${id}`)
      .then((r) => r.json())
      .then(setProduct);
  }, [id]);
  // ...
}

// Good: a service layer is the indirection point
class ProductService {
  async getById(id: string): Promise<Product> {
    const res = await fetch(`https://api.example.com/v2/products/${id}`);
    return res.json();
  }
}

function ProductCard({ id, service }: { id: string; service: ProductService }) {
  const [product, setProduct] = useState<Product | null>(null);
  useEffect(() => {
    service.getById(id).then(setProduct);
  }, [id]);
  // ...
}
```

**Weaknesses:**

_EN:_

- Each indirection layer adds complexity and a new place for bugs
- Too many mediators make call traces hard to follow

_UA:_

- Кожен рівень непрямості додає складність та нове місце для помилок
- Занадто багато посередників робить трасування викликів важким

---

## Pattern 9 — Protected Variations

**Problem:**

_EN:_ How to design a system so that changes in one part don't cascade through everything else?

_UA:_ Як проєктувати систему так, щоб зміни в одній частині не поширювалися на всі інші?

**Description:**

_EN:_ Identify points of instability (external APIs, third-party libs, changing requirements) and wrap them behind a stable interface. Variation is hidden; the rest of the system only sees the interface.

_UA:_ Визнач точки нестабільності (зовнішні API, сторонні бібліотеки, мінливі вимоги) і обгорни їх за стабільним інтерфейсом. Варіативність прихована; решта системи бачить лише інтерфейс.

**Relation to SOLID / OOP:**

_EN:_

- Open/Closed Principle — stable interface; implementations change freely behind it
- Dependency Inversion Principle — depend on the stable abstraction, not the volatile concretion

_UA:_

- Open/Closed Principle — стабільний інтерфейс; реалізації вільно змінюються за ним
- Dependency Inversion Principle — залежи від стабільної абстракції, а не від нестабільної конкретики

**Example:**

```tsx
// Bad: analytics provider used directly — swapping it touches every call site
function SignupButton() {
  function handleClick() {
    window.mixpanel.track("signup_clicked", { source: "hero" });
  }
  return <button onClick={handleClick}>Sign up</button>;
}

// Good: analytics is wrapped — only the adapter changes when the provider does
interface Analytics {
  track(event: string, props?: Record<string, unknown>): void;
}

class MixpanelAnalytics implements Analytics {
  track(event: string, props?: Record<string, unknown>) {
    window.mixpanel.track(event, props);
  }
}

function SignupButton({ analytics }: { analytics: Analytics }) {
  function handleClick() {
    analytics.track("signup_clicked", { source: "hero" });
  }
  return <button onClick={handleClick}>Sign up</button>;
}
```

**Weaknesses:**

_EN:_

- Wrapping every volatile dependency upfront is over-engineering — the variation point has to be real and likely to change
- The abstraction itself can become a point of instability if designed poorly

_UA:_

- Обгортання кожної нестабільної залежності заздалегідь є надмірним проєктуванням — точка варіативності має бути реальною та ймовірною
- Сама абстракція може стати точкою нестабільності, якщо спроєктована погано

---

## Summary

| Pattern              | Core question                                |
| -------------------- | -------------------------------------------- |
| Information Expert   | Who has the data to do this?                 |
| Creator              | Who should create this object?               |
| Controller           | Who handles this system event?               |
| Low Coupling         | How do we minimise dependencies?             |
| High Cohesion        | How do we keep classes focused?              |
| Polymorphism         | How do we vary behaviour by type?            |
| Pure Fabrication     | Where does this infrastructure concern live? |
| Indirection          | How do we decouple two things?               |
| Protected Variations | How do we isolate points of change?          |
