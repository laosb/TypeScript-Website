---
title: 高级类型
layout: docs
permalink: /docs/handbook/advanced-types.html
oneline: TypeScript 中关于类型的高级概念
---
此页面列出了一些高级的类型建模方法，这些方法可以与被包含在 TypeScript 中，并可以在全局使用的 [工具类型](/docs/handbook/utility-types.html) 共同工作。

# 类型守卫和区分类型

在值可以重叠的情况下，联合类型对于建模非常有用。  

当我们想要明确知道我们是否有 `Fish` 时会发生什么？  
在 JavaScript 中，要区分两个可能的值的一个常见用法是检查一个成员的存在。  
像前面提到的那样，你只能访问所有组成并集类型的类型都有的成员。

```ts
let pet = getSmallPet();

// 这些属性访问都会导致错误
if (pet.swim) {
  pet.swim();
} else if (pet.fly) {
  pet.fly();
}
```

为了让代码正常工作，我们需要使用类型断言：

```ts
let pet = getSmallPet();

if ((pet as Fish).swim) {
  (pet as Fish).swim();
} else if ((pet as Bird).fly) {
  (pet as Bird).fly();
}
```

## 用户定义的类型守卫

注意，我们不得不多次使用类型断言。  
当我们执行了检查，如果可以知道每个分支内的 “pet” 的类型会更好。

碰巧 TypeScript 有一个称之为 _类型守卫_ 的东西。  
类型守卫是执行运行时检查的一个表达式，它可以确保该类型在一定范围内。

### 使用类型判定

要定义类型守卫，我们只需要定义一个返回值类型为 _类型判定_ 的函数。

```ts
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

在上面的例子中，`pet is Fish` 是你的类型判定。  
类型判定的格式为 `参数 is 类型`，其中参数必须是当前函数签名中参数的名称。 

当使用某些变量调用 `isFish` 时，如果原始类型兼容，TypeScript 将会将该变量的类型 _缩小_ 到指定的类型。

```ts
// 现在调用 'swim' 和 'fly' 已经没有问题了。

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

请注意，TypeScript 不仅知道在 `if` 分支中 `pet` 的类型是 `Fish`，它也知道在 `else` 分支中，你并没有 `Fish`，因此您只能得到 `Bird`。 

### 使用 `in` 操作符

目前 `in` 操作符充当类型的缩小表达式。

对形如`n in x` 的表达式, 当然 `n` 是一个字符串字面量或是字符串类型, 并且 `x` 是并集类型, 在 "true" 分支中类型将缩小为具有可选或必选的 `n` 属性的类型, 并且 "false" 分支将缩小为具有可选或没有属性 `n` 的类型。

```ts
function move(pet: Fish | Bird) {
  if ("swim" in pet) {
    return pet.swim();
  }
  return pet.fly();
}
```

## `typeof` 类型守卫

让我们回过头来编写使用并集类型的 `padLeft`。  
我们可以是使用类型判定来编写它，如下所示：

```ts
function isNumber(x: any): x is number {
  return typeof x === "number";
}

function isString(x: any): x is string {
  return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
  if (isNumber(padding)) {
    return Array(padding + 1).join(" ") + value;
  }
  if (isString(padding)) {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

如果必须定义函数来确定类型是否为原始类型的话会非常麻烦。  
幸运的是你不需要将 `typeof x === "number"` 抽象到它自己的函数中，因为 TypeScript 会自行将其识别为类型守卫。  
这意味着我们可以内联的编写这些检查。

```ts
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

_·typeof 类型守卫·_ 可以识别两种不同的形式：`typeof v === "typename"` 和 `typeof v !== "typename"`，其中 `"typename"` 必须是 `"number"`, `"string"`, `"boolean"`, 或 `"symbol"`。

## `instanceof` 类型守卫

如果你已经阅读过 `typeof` 类型守卫，并且熟悉 JavaScript 中的 `instanceof` 操作符，那么您可能对本节的内容有所了解。

_`instanceof` 类型守卫_ 是一种使用构造函数缩小类型的方法。  
我们可以使用下面的字符串填充器举例: 

```ts
interface Padder {
  getPaddingString(): string;
}

class SpaceRepeatingPadder implements Padder {
  constructor(private numSpaces: number) {}
  getPaddingString() {
    return Array(this.numSpaces + 1).join(" ");
  }
}

class StringPadder implements Padder {
  constructor(private value: string) {}
  getPaddingString() {
    return this.value;
  }
}

function getRandomPadder() {
  return Math.random() < 0.5
    ? new SpaceRepeatingPadder(4)
    : new StringPadder("  ");
}

// 类型是 'SpaceRepeatingPadder | StringPadder'
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
  padder; // 类型缩小为 'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
  padder; // 类型缩小为 'StringPadder'
}
```

`instanceof` 的右边必须是一个构造函数，TypeScript 将会按顺序缩小为：  
1. 函数的 `prototype` 属性的类型 (如果其类型不是 `any`)
2. 和该类型的构造器签名返回类型的并集

# 可空类型

TypeScript 有两个特殊的类型：`null` 和 `undefined`, 它们的值分别为 null 和 undefined.  
我们在 [基础类型部分](/docs/handbook/basic-types.html) 中简要提及了这些内容。  
默认情况下，类型检查器将认为 `null` 和 `undefined` 可以赋值给任何东西。  
事实上 `null` 和 `undefined` 是每种类型的有效值，这意味着即使你 _不能_ 做到禁止将他们赋值给其他类型。
`null` 的发明者 Tony Hoare 称其为他的 ["数亿美元的错误"](https://wikipedia.org/wiki/Null_pointer#History)。

`--strictNullChecks` 标志可以解决这个问题：声明变量时将不会自动包含 `null` 或 `undefined`。  
你可以使用并集类型来显式的包含他们：

```ts
let s = "foo";
s = null; // 错误, 'null' 不能赋值给 'string'
let sn: string | null = "bar";
sn = null; // ok

sn = undefined; // 错误, 'undefined' 不能赋值给 'string | null'
```

请注意，TypeScript 对 `null` 和 `undefined` 区别对待是为了符合 JavaScript 语义。  
`string | null` 与 `string | undefined` 和 `string | undefined | null` 是不同的类型。  

从 TypeScript 3.7 开始，你可以使用 [optional chaining](/docs/handbook/release-notes/typescript-3-7.html#optional-chaining) 来简化对可空类型的使用。

## 可选参数和属性

当使用 `--strictNullChecks` 时，可选参数会自动添加 `| undefined`：

```ts
function f(x: number, y?: number) {
  return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // 错误, 'null' 不能赋值给 'number | undefined'
```

可选属性也是如此：

```ts
class C {
  a: number;
  b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // 错误, 'undefined' 不能赋值给 'number'
c.b = 13;
c.b = undefined; // ok
c.b = null; // 错误, 'null' 不能赋值给 'number | undefined'
```

## 类型守卫和类型断言

由于可空类型是通过并集类型实现的，因此你需要使用类型守卫来避免 `null`。  
幸运的是，这与你在 JavaScript 中编写的代码相同：

```ts
function f(sn: string | null): string {
  if (sn == null) {
    return "default";
  } else {
    return sn;
  }
}
```

`null` 的消除在这里非常明显，当然您也可以使用更精简的运算符：

```ts
function f(sn: string | null): string {
  return sn || "default";
}
```

如果编译器不能自动消除 `null` 或 `undefined`, 你可以使用类型断言手动删除他们。  
其语法是后缀`!`，`identifier!` 将会在 identifier 的类型中去除 `null` 和 `undefined`。  

```ts
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + ".  the " + epithet; // 错误, 'name' 是可空的
  }
  name = name || "Bob";
  return postfix("great");
}

function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + ".  the " + epithet; // ok
  }
  name = name || "Bob";
  return postfix("great");
}
```

该实例中使用了嵌套的函数，因为编译器无法消除嵌套函数内部的空值（立即调用函数, IIFE除外）。  
这是因为它无法跟踪对嵌套函数的所有调用，尤其是从外部函数返回它时。  
不知道函数在哪里调用，就不知道函数执行时 `name` 的具体类型。

# 类型别名

类型别名创建类型的另一个名字。  
类型别名类似与接口，但是可以命名原始类型、并集、元组和其他必须手工定义的类型。

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
  if (typeof n === "string") {
    return n;
  } else {
    return n();
  }
}
```

别名并没有真的创建一个新的类型 - 它创建了一个指向该类型的 _名称_。  
虽然可以将原始类型的别名作为文档的一种形式，但是它并不是非常有效。

像接口一样，类型别名也可以是泛型的 - 我们可以添加一个类型参数并且在别名声明的右侧使用他们。

```ts
type Container<T> = { value: T };
```

我们同样可以在属性中指定一个指向它本身的类型别名。

```ts
type Tree<T> = {
  value: T;
  left: Tree<T>;
  right: Tree<T>;
};
```

结合交集类型，我们可以做出一些复杂的类型：

```ts
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
  name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```

但是类型别名不可以出现在其声明右侧的其他任何位置：

```ts
type Yikes = Array<Yikes>; // error
```

## 接口与类型别名的对比

正如我们所提到的，类型别名的作用有些像接口，但是还是有一些微妙的区别。  
其中一个区别是，接口会创建一个被别处使用的新名词，但是类型别名不会 &mdash; 例如，错误信息中不会使用类型别名。  
在下面的代码中，将光标悬停在 `interfaced` 上将会显示它返回 `Interface`, 但在 `aliased` 上将会显示对象字面量类型。

```ts
type Alias = { num: number };
interface Interface {
  num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```

在旧版本的 TypeScript 中，类型别名不能被其他类型所集成或实现（也不能继承或实现其他类型）。  
从 2.7 版本开始，类型别名可以通过创建一个新的交集类型来扩展，例如 `type Cat = Animal & { purrs: true }`。

像 [开闭原则](https://wikipedia.org/wiki/Open/closed_principle) 所指出的那样，你应该尽可能使用接口而不是类型别名。

另一方面，如果你不能通过接口来表达一些形状或你需要使用并集、元组类型，类型别名通常是最好的选择。

# 字符串字面量类型

字符串字面量类型允许你指定字符串必须具有的精确的值。  
在实践中，字符串字面量类型可以与并集类型，类型守卫和类型别名组合得很好。  
你可以一起使用这些特性来获得枚举式字符串的行为。

```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in") {
      // ...
    } else if (easing === "ease-out") {
    } else if (easing === "ease-in-out") {
    } else {
      // error! should not pass null or undefined.
    }
  }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

你可以传入三个允许的字符串中的任何一个，但是其他任何和字符串都会导致如下错误：

```
Argument of type '"uneasy"' is not assignable to parameter of type '"ease-in" | "ease-out" | "ease-in-out"'
```
字符串字面量类型也可以使用同样的方法来区分重载：

```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
  // ... code goes here ...
}
```

# 数字字面量类型

TypeScript 同样有数字字面量类型。

```ts
function rollDice(): 1 | 2 | 3 | 4 | 5 | 6 {
  // ...
}
```
这些很少显式指定，但在缩小问题范围时很有用，可以找到 bug：

```ts
function foo(x: number) {
  if (x !== 1 || x !== 2) {
    //         ~~~~~~~
    // Operator '!==' cannot be applied to types '1' and '2'.
  }
}
```

In other words, `x` must be `1` when it gets compared to `2`, meaning that the above check is making an invalid comparison.

# Enum Member Types

As mentioned in [our section on enums](./enums.html#union-enums-and-enum-member-types), enum members have types when every member is literal-initialized.

Much of the time when we talk about "singleton types", we're referring to both enum member types as well as numeric/string literal types, though many users will use "singleton types" and "literal types" interchangeably.

# Discriminated Unions

You can combine singleton types, union types, type guards, and type aliases to build an advanced pattern called _discriminated unions_, also known as _tagged unions_ or _algebraic data types_.
Discriminated unions are useful in functional programming.
Some languages automatically discriminate unions for you; TypeScript instead builds on JavaScript patterns as they exist today.
There are three ingredients:

1. Types that have a common, singleton type property &mdash; the _discriminant_.
2. A type alias that takes the union of those types &mdash; the _union_.
3. Type guards on the common property.

```ts
interface Square {
  kind: "square";
  size: number;
}
interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}
interface Circle {
  kind: "circle";
  radius: number;
}
```

First we declare the interfaces we will union.
Each interface has a `kind` property with a different string literal type.
The `kind` property is called the _discriminant_ or _tag_.
The other properties are specific to each interface.
Notice that the interfaces are currently unrelated.
Let's put them into a union:

```ts
type Shape = Square | Rectangle | Circle;
```

Now let's use the discriminated union:

```ts
function area(s: Shape) {
  switch (s.kind) {
    case "square":
      return s.size * s.size;
    case "rectangle":
      return s.height * s.width;
    case "circle":
      return Math.PI * s.radius ** 2;
  }
}
```

## Exhaustiveness checking

We would like the compiler to tell us when we don't cover all variants of the discriminated union.
For example, if we add `Triangle` to `Shape`, we need to update `area` as well:

```ts
type Shape = Square | Rectangle | Circle | Triangle;
function area(s: Shape) {
  switch (s.kind) {
    case "square":
      return s.size * s.size;
    case "rectangle":
      return s.height * s.width;
    case "circle":
      return Math.PI * s.radius ** 2;
  }
  // should error here - we didn't handle case "triangle"
}
```

There are two ways to do this.
The first is to turn on `--strictNullChecks` and specify a return type:

```ts
function area(s: Shape): number {
  // error: returns number | undefined
  switch (s.kind) {
    case "square":
      return s.size * s.size;
    case "rectangle":
      return s.height * s.width;
    case "circle":
      return Math.PI * s.radius ** 2;
  }
}
```

Because the `switch` is no longer exhaustive, TypeScript is aware that the function could sometimes return `undefined`.
If you have an explicit return type `number`, then you will get an error that the return type is actually `number | undefined`.
However, this method is quite subtle and, besides, `--strictNullChecks` does not always work with old code.

The second method uses the `never` type that the compiler uses to check for exhaustiveness:

```ts
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}
function area(s: Shape) {
  switch (s.kind) {
    case "square":
      return s.size * s.size;
    case "rectangle":
      return s.height * s.width;
    case "circle":
      return Math.PI * s.radius ** 2;
    default:
      return assertNever(s); // error here if there are missing cases
  }
}
```

Here, `assertNever` checks that `s` is of type `never` &mdash; the type that's left after all other cases have been removed.
If you forget a case, then `s` will have a real type and you will get a type error.
This method requires you to define an extra function, but it's much more obvious when you forget it.

# Polymorphic `this` types

A polymorphic `this` type represents a type that is the _subtype_ of the containing class or interface.
This is called _F_-bounded polymorphism.
This makes hierarchical fluent interfaces much easier to express, for example.
Take a simple calculator that returns `this` after each operation:

```ts
class BasicCalculator {
  public constructor(protected value: number = 0) {}
  public currentValue(): number {
    return this.value;
  }
  public add(operand: number): this {
    this.value += operand;
    return this;
  }
  public multiply(operand: number): this {
    this.value *= operand;
    return this;
  }
  // ... other operations go here ...
}

let v = new BasicCalculator(2)
  .multiply(5)
  .add(1)
  .currentValue();
```

Since the class uses `this` types, you can extend it and the new class can use the old methods with no changes.

```ts
class ScientificCalculator extends BasicCalculator {
  public constructor(value = 0) {
    super(value);
  }
  public sin() {
    this.value = Math.sin(this.value);
    return this;
  }
  // ... other operations go here ...
}

let v = new ScientificCalculator(2)
  .multiply(5)
  .sin()
  .add(1)
  .currentValue();
```

Without `this` types, `ScientificCalculator` would not have been able to extend `BasicCalculator` and keep the fluent interface.
`multiply` would have returned `BasicCalculator`, which doesn't have the `sin` method.
However, with `this` types, `multiply` returns `this`, which is `ScientificCalculator` here.

# Index types

With index types, you can get the compiler to check code that uses dynamic property names.
For example, a common JavaScript pattern is to pick a subset of properties from an object:

```js
function pluck(o, propertyNames) {
  return propertyNames.map(n => o[n]);
}
```

Here's how you would write and use this function in TypeScript, using the **index type query** and **indexed access** operators:

```ts
function pluck<T, K extends keyof T>(o: T, propertyNames: K[]): T[K][] {
  return propertyNames.map(n => o[n]);
}

interface Car {
  manufacturer: string;
  model: string;
  year: number;
}
let taxi: Car = {
  manufacturer: "Toyota",
  model: "Camry",
  year: 2014
};

// Manufacturer and model are both of type string,
// so we can pluck them both into a typed string array
let makeAndModel: string[] = pluck(taxi, ["manufacturer", "model"]);

// If we try to pluck model and year, we get an
// array of a union type: (string | number)[]
let modelYear = pluck(taxi, ["model", "year"]);
```

The compiler checks that `manufacturer` and `model` are actually properties on `Car`.
The example introduces a couple of new type operators.
First is `keyof T`, the **index type query operator**.
For any type `T`, `keyof T` is the union of known, public property names of `T`.
For example:

```ts
let carProps: keyof Car; // the union of ("manufacturer" | "model" | "year")
```

`keyof Car` is completely interchangeable with `"manufacturer" | "model" | "year"`.
The difference is that if you add another property to `Car`, say `ownersAddress: string`, then `keyof Car` will automatically update to be `"manufacturer" | "model" | "year" | "ownersAddress"`.
And you can use `keyof` in generic contexts like `pluck`, where you can't possibly know the property names ahead of time.
That means the compiler will check that you pass the right set of property names to `pluck`:

```ts
// error, Type '"unknown"' is not assignable to type '"manufacturer" | "model" | "year"'
pluck(taxi, ["year", "unknown"]);
```

The second operator is `T[K]`, the **indexed access operator**.
Here, the type syntax reflects the expression syntax.
That means that `person["name"]` has the type `Person["name"]` &mdash; which in our example is just `string`.
However, just like index type queries, you can use `T[K]` in a generic context, which is where its real power comes to life.
You just have to make sure that the type variable `K extends keyof T`.
Here's another example with a function named `getProperty`.

```ts
function getProperty<T, K extends keyof T>(o: T, propertyName: K): T[K] {
  return o[propertyName]; // o[propertyName] is of type T[K]
}
```

In `getProperty`, `o: T` and `propertyName: K`, so that means `o[propertyName]: T[K]`.
Once you return the `T[K]` result, the compiler will instantiate the actual type of the key, so the return type of `getProperty` will vary according to which property you request.

```ts
let name: string = getProperty(taxi, "manufacturer");
let year: number = getProperty(taxi, "year");

// error, Argument of type '"unknown"' is not assignable to parameter of type '"manufacturer" | "model" | "year"'
let unknown = getProperty(taxi, "unknown");
```

## Index types and index signatures

`keyof` and `T[K]` interact with index signatures. An index signature parameter type must be 'string' or 'number'.
If you have a type with a string index signature, `keyof T` will be `string | number`
(and not just `string`, since in JavaScript you can access an object property either
by using strings (`object["42"]`) or numbers (`object[42]`)).
And `T[string]` is just the type of the index signature:

```ts
interface Dictionary<T> {
  [key: string]: T;
}
let keys: keyof Dictionary<number>; // string | number
let value: Dictionary<number>["foo"]; // number
```

If you have a type with a number index signature, `keyof T` will just be `number`.

```ts
interface Dictionary<T> {
  [key: number]: T;
}
let keys: keyof Dictionary<number>; // number
let value: Dictionary<number>["foo"]; // Error, Property 'foo' does not exist on type 'Dictionary<number>'.
let value: Dictionary<number>[42]; // number
```

# Mapped types

A common task is to take an existing type and make each of its properties optional:

```ts
interface PersonPartial {
  name?: string;
  age?: number;
}
```

Or we might want a readonly version:

```ts
interface PersonReadonly {
  readonly name: string;
  readonly age: number;
}
```

This happens often enough in JavaScript that TypeScript provides a way to create new types based on old types &mdash; **mapped types**.
In a mapped type, the new type transforms each property in the old type in the same way.
For example, you can make all properties of a type `readonly` or optional.
Here are a couple of examples:

```ts
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

And to use it:

```ts
type PersonPartial = Partial<Person>;
type ReadonlyPerson = Readonly<Person>;
```

Note that this syntax describes a type rather than a member.
If you want to add members, you can use an intersection type:

```ts
// Use this:
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
} & { newMember: boolean }

// **Do not** use the following!
// This is an error!
type PartialWithNewMember<T> = {
  [P in keyof T]?: T[P];
  newMember: boolean;
}
```

Let's take a look at the simplest mapped type and its parts:

```ts
type Keys = "option1" | "option2";
type Flags = { [K in Keys]: boolean };
```

The syntax resembles the syntax for index signatures with a `for .. in` inside.
There are three parts:

1. The type variable `K`, which gets bound to each property in turn.
2. The string literal union `Keys`, which contains the names of properties to iterate over.
3. The resulting type of the property.

In this simple example, `Keys` is a hard-coded list of property names and the property type is always `boolean`, so this mapped type is equivalent to writing:

```ts
type Flags = {
  option1: boolean;
  option2: boolean;
};
```

Real applications, however, look like `Readonly` or `Partial` above.
They're based on some existing type, and they transform the properties in some way.
That's where `keyof` and indexed access types come in:

```ts
type NullablePerson = { [P in keyof Person]: Person[P] | null };
type PartialPerson = { [P in keyof Person]?: Person[P] };
```

But it's more useful to have a general version.

```ts
type Nullable<T> = { [P in keyof T]: T[P] | null };
type Partial<T> = { [P in keyof T]?: T[P] };
```

In these examples, the properties list is `keyof T` and the resulting type is some variant of `T[P]`.
This is a good template for any general use of mapped types.
That's because this kind of transformation is [homomorphic](https://wikipedia.org/wiki/Homomorphism), which means that the mapping applies only to properties of `T` and no others.
The compiler knows that it can copy all the existing property modifiers before adding any new ones.
For example, if `Person.name` was readonly, `Partial<Person>.name` would be readonly and optional.

Here's one more example, in which `T[P]` is wrapped in a `Proxy<T>` class:

```ts
type Proxy<T> = {
  get(): T;
  set(value: T): void;
};
type Proxify<T> = {
  [P in keyof T]: Proxy<T[P]>;
};
function proxify<T>(o: T): Proxify<T> {
  // ... wrap proxies ...
}
let proxyProps = proxify(props);
```

Note that `Readonly<T>` and `Partial<T>` are so useful, they are included in TypeScript's standard library along with `Pick` and `Record`:

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```

`Readonly`, `Partial` and `Pick` are homomorphic whereas `Record` is not.
One clue that `Record` is not homomorphic is that it doesn't take an input type to copy properties from:

```ts
type ThreeStringProps = Record<"prop1" | "prop2" | "prop3", string>;
```

Non-homomorphic types are essentially creating new properties, so they can't copy property modifiers from anywhere.

## Inference from mapped types

Now that you know how to wrap the properties of a type, the next thing you'll want to do is unwrap them.
Fortunately, that's pretty easy:

```ts
function unproxify<T>(t: Proxify<T>): T {
  let result = {} as T;
  for (const k in t) {
    result[k] = t[k].get();
  }
  return result;
}

let originalProps = unproxify(proxyProps);
```

Note that this unwrapping inference only works on homomorphic mapped types.
If the mapped type is not homomorphic you'll have to give an explicit type parameter to your unwrapping function.

# Conditional Types

TypeScript 2.8 introduces _conditional types_ which add the ability to express non-uniform type mappings.
A conditional type selects one of two possible types based on a condition expressed as a type relationship test:

```ts
T extends U ? X : Y
```

The type above means when `T` is assignable to `U` the type is `X`, otherwise the type is `Y`.

A conditional type `T extends U ? X : Y` is either _resolved_ to `X` or `Y`, or _deferred_ because the condition depends on one or more type variables.
When `T` or `U` contains type variables, whether to resolve to `X` or `Y`, or to defer, is determined by whether or not the type system has enough information to conclude that `T` is always assignable to `U`.

As an example of some types that are immediately resolved, we can take a look at the following example:

```ts
declare function f<T extends boolean>(x: T): T extends true ? string : number;

// Type is 'string | number'
let x = f(Math.random() < 0.5);
```

Another example would be the `TypeName` type alias, which uses nested conditional types:

```ts
type TypeName<T> = T extends string
  ? "string"
  : T extends number
  ? "number"
  : T extends boolean
  ? "boolean"
  : T extends undefined
  ? "undefined"
  : T extends Function
  ? "function"
  : "object";

type T0 = TypeName<string>; // "string"
type T1 = TypeName<"a">; // "string"
type T2 = TypeName<true>; // "boolean"
type T3 = TypeName<() => void>; // "function"
type T4 = TypeName<string[]>; // "object"
```

But as an example of a place where conditional types are deferred - where they stick around instead of picking a branch - would be in the following:

```ts
interface Foo {
  propA: boolean;
  propB: boolean;
}

declare function f<T>(x: T): T extends Foo ? string : number;

function foo<U>(x: U) {
  // Has type 'U extends Foo ? string : number'
  let a = f(x);

  // This assignment is allowed though!
  let b: string | number = a;
}
```

In the above, the variable `a` has a conditional type that hasn't yet chosen a branch.
When another piece of code ends up calling `foo`, it will substitute in `U` with some other type, and TypeScript will re-evaluate the conditional type, deciding whether it can actually pick a branch.

In the meantime, we can assign a conditional type to any other target type as long as each branch of the conditional is assignable to that target.
So in our example above we were able to assign `U extends Foo ? string : number` to `string | number` since no matter what the conditional evaluates to, it's known to be either `string` or `number`.

## Distributive conditional types

Conditional types in which the checked type is a naked type parameter are called _distributive conditional types_.
Distributive conditional types are automatically distributed over union types during instantiation.
For example, an instantiation of `T extends U ? X : Y` with the type argument `A | B | C` for `T` is resolved as `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`.

### Example

```ts
type T10 = TypeName<string | (() => void)>; // "string" | "function"
type T12 = TypeName<string | string[] | undefined>; // "string" | "object" | "undefined"
type T11 = TypeName<string[] | number[]>; // "object"
```

In instantiations of a distributive conditional type `T extends U ? X : Y`, references to `T` within the conditional type are resolved to individual constituents of the union type (i.e. `T` refers to the individual constituents _after_ the conditional type is distributed over the union type).
Furthermore, references to `T` within `X` have an additional type parameter constraint `U` (i.e. `T` is considered assignable to `U` within `X`).

### Example

```ts
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T20 = Boxed<string>; // BoxedValue<string>;
type T21 = Boxed<number[]>; // BoxedArray<number>;
type T22 = Boxed<string | number[]>; // BoxedValue<string> | BoxedArray<number>;
```

Notice that `T` has the additional constraint `any[]` within the true branch of `Boxed<T>` and it is therefore possible to refer to the element type of the array as `T[number]`. Also, notice how the conditional type is distributed over the union type in the last example.

The distributive property of conditional types can conveniently be used to _filter_ union types:

```ts
type Diff<T, U> = T extends U ? never : T; // Remove types from T that are assignable to U
type Filter<T, U> = T extends U ? T : never; // Remove types from T that are not assignable to U

type T30 = Diff<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "b" | "d"
type T31 = Filter<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "a" | "c"
type T32 = Diff<string | number | (() => void), Function>; // string | number
type T33 = Filter<string | number | (() => void), Function>; // () => void

type NonNullable<T> = Diff<T, null | undefined>; // Remove null and undefined from T

type T34 = NonNullable<string | number | undefined>; // string | number
type T35 = NonNullable<string | string[] | null | undefined>; // string | string[]

function f1<T>(x: T, y: NonNullable<T>) {
  x = y; // Ok
  y = x; // Error
}

function f2<T extends string | undefined>(x: T, y: NonNullable<T>) {
  x = y; // Ok
  y = x; // Error
  let s1: string = x; // Error
  let s2: string = y; // Ok
}
```

Conditional types are particularly useful when combined with mapped types:

```ts
type FunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? K : never;
}[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = {
  [K in keyof T]: T[K] extends Function ? never : K;
}[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface Part {
  id: number;
  name: string;
  subparts: Part[];
  updatePart(newName: string): void;
}

type T40 = FunctionPropertyNames<Part>; // "updatePart"
type T41 = NonFunctionPropertyNames<Part>; // "id" | "name" | "subparts"
type T42 = FunctionProperties<Part>; // { updatePart(newName: string): void }
type T43 = NonFunctionProperties<Part>; // { id: number, name: string, subparts: Part[] }
```

Similar to union and intersection types, conditional types are not permitted to reference themselves recursively.
For example the following is an error.

### Example

```ts
type ElementType<T> = T extends any[] ? ElementType<T[number]> : T; // Error
```

## Type inference in conditional types

Within the `extends` clause of a conditional type, it is now possible to have `infer` declarations that introduce a type variable to be inferred.
Such inferred type variables may be referenced in the true branch of the conditional type.
It is possible to have multiple `infer` locations for the same type variable.

For example, the following extracts the return type of a function type:

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

Conditional types can be nested to form a sequence of pattern matches that are evaluated in order:

```ts
type Unpacked<T> = T extends (infer U)[]
  ? U
  : T extends (...args: any[]) => infer U
  ? U
  : T extends Promise<infer U>
  ? U
  : T;

type T0 = Unpacked<string>; // string
type T1 = Unpacked<string[]>; // string
type T2 = Unpacked<() => string>; // string
type T3 = Unpacked<Promise<string>>; // string
type T4 = Unpacked<Promise<string>[]>; // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>; // string
```

The following example demonstrates how multiple candidates for the same type variable in co-variant positions causes a union type to be inferred:

```ts
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;
type T10 = Foo<{ a: string; b: string }>; // string
type T11 = Foo<{ a: string; b: number }>; // string | number
```

Likewise, multiple candidates for the same type variable in contra-variant positions causes an intersection type to be inferred:

```ts
type Bar<T> = T extends { a: (x: infer U) => void; b: (x: infer U) => void }
  ? U
  : never;
type T20 = Bar<{ a: (x: string) => void; b: (x: string) => void }>; // string
type T21 = Bar<{ a: (x: string) => void; b: (x: number) => void }>; // string & number
```

When inferring from a type with multiple call signatures (such as the type of an overloaded function), inferences are made from the _last_ signature (which, presumably, is the most permissive catch-all case).
It is not possible to perform overload resolution based on a list of argument types.

```ts
declare function foo(x: string): number;
declare function foo(x: number): string;
declare function foo(x: string | number): string | number;
type T30 = ReturnType<typeof foo>; // string | number
```

It is not possible to use `infer` declarations in constraint clauses for regular type parameters:

```ts
type ReturnType<T extends (...args: any[]) => infer R> = R; // Error, not supported
```

However, much the same effect can be obtained by erasing the type variables in the constraint and instead specifying a conditional type:

```ts
type AnyFunction = (...args: any[]) => any;
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R
  ? R
  : any;
```

## Predefined conditional types

TypeScript 2.8 adds several predefined conditional types to `lib.d.ts`:

- `Exclude<T, U>` -- Exclude from `T` those types that are assignable to `U`.
- `Extract<T, U>` -- Extract from `T` those types that are assignable to `U`.
- `NonNullable<T>` -- Exclude `null` and `undefined` from `T`.
- `ReturnType<T>` -- Obtain the return type of a function type.
- `InstanceType<T>` -- Obtain the instance type of a constructor function type.

### Example

```ts
type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "b" | "d"
type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">; // "a" | "c"

type T02 = Exclude<string | number | (() => void), Function>; // string | number
type T03 = Extract<string | number | (() => void), Function>; // () => void

type T04 = NonNullable<string | number | undefined>; // string | number
type T05 = NonNullable<(() => string) | string[] | null | undefined>; // (() => string) | string[]

function f1(s: string) {
  return { a: 1, b: s };
}

class C {
  x = 0;
  y = 0;
}

type T10 = ReturnType<() => string>; // string
type T11 = ReturnType<(s: string) => void>; // void
type T12 = ReturnType<<T>() => T>; // {}
type T13 = ReturnType<<T extends U, U extends number[]>() => T>; // number[]
type T14 = ReturnType<typeof f1>; // { a: number, b: string }
type T15 = ReturnType<any>; // any
type T16 = ReturnType<never>; // never
type T17 = ReturnType<string>; // Error
type T18 = ReturnType<Function>; // Error

type T20 = InstanceType<typeof C>; // C
type T21 = InstanceType<any>; // any
type T22 = InstanceType<never>; // never
type T23 = InstanceType<string>; // Error
type T24 = InstanceType<Function>; // Error
```

> Note: The `Exclude` type is a proper implementation of the `Diff` type suggested [here](https://github.com/Microsoft/TypeScript/issues/12215#issuecomment-307871458). We've used the name `Exclude` to avoid breaking existing code that defines a `Diff`, plus we feel that name better conveys the semantics of the type.
