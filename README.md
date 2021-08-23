# A glossary of TypeScript

## Motivation

Once upon a time, there was a developer that had an issue using the TypeScript language. He wanted to share his issue with the community to get help, but he didn't know what words to use that would best describe his problem. He struggled to find the appropriate title and description as he didn't know how to name "this behavior" or "that kind of type mechanism".

This story encouraged me to start writing _a glossary of TypeScript_. Hopefully it will help you if you have to look for an issue, write an issue, or communicate with other TypeScript developers.

Disclaimer: it was me, I was the developer that struggled. I still struggle though, but this means I still have things to learn, which is great!

You may not agree with some definitions or examples in this document. Please let me know by opening an issue. You can also contact me on [Twitter](https://twitter.com/Haaress). Only together can we improve this glossary!

## Table of content

- [Ambient declaration](#ambient-declaration)
- [Conditional type](#conditional-type)
- [Diagnostic message](#diagnostic-message)
- [Discriminated union](#discriminated-union)
- [Distributive conditional type](#distributive-conditional-type) (contains information regarding _naked types_)
- [Indirect type narrowing](#indirect-type-narrowing)
- [IntelliSense](#intellisense)
- [Intersection type](#intersection-type)
- [Intrinsic type](#intrinsic-type)
- [Literal type](#literal-type)
- [Mapped type](#mapped-type)
- [Type alias](#type-alias)
- [Type assertion](#type-assertion)
- [Type check](#type-check)
- [Type guard](#type-guard)
- [Type inference](#type-inference)
- [Type narrowing](#type-narrowing)
- [Type predicate](#type-predicate)
- [Union type](#union-type)

## Glossary

### Ambient declaration

An ambient declaration lets the developer tell the compiler that a global variable/function exists, and which type it has: "I know you can't find it but trust me, this variable/function exists at runtime, and it has _this_ type". The `declare` keyword lets you write such declaration.

##### Example

```ts
doSomething(42) // TS error, "Cannot find name 'doSomething'"
```

```ts
// no implementation is provided here, because it's available at runtime.
declare function doSomething(n: number): void

doSomething(42) // no TS error
```

### Conditional type

Conditional types were introduced in [TypeScript v2.8](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#conditional-types). They can be used to make _non-uniform type mapping_, meaning they can be considered as _functions for types_, like [mapped types](#mapped-type) but more powerful.

TypeScript provides several conditional types for the developers, such as `Exclude`, `NonNullable`, `Parameters` or `ReturnType`.

##### Example

```ts
// { "strict": true }

const isDefined = <A>(value: A | null | undefined): value is NonNullable<A> => value !== null && value !== undefined

const button = document.getElementById('my-button') // HTMLElement | null
if (isDefined(button)) {
  console.log(`Button text: ${button.innerText}`)
} else {
  console.log('Button is not defined')
}
```

In this example we are using the conditional type `NonNullable` in _strict mode_ as a [type guard](#type-guard) to make sure the value is defined in the "true" branch of our condition.

##### Example

```ts
type SyncOperationName = 'getViewportSize' | 'getElementSize'
type AsyncOperationName = 'loadUsers' | 'sendData'
type OperationName = SyncOperationName | AsyncOperationName

type Operation<A extends OperationName> = A extends SyncOperationName
  ? { type: 'syncOperation'; name: A }
  : { type: 'asyncOperation'; name: A; timeout: number }
```

Here we are mapping `A` to a completely different type `Operation<A>`, which depends on the effective type of `A`:

- If `A` is a `SyncOperationName` then `Operation<A>` is `{ type: 'syncOperation', name: A }`.
- Otherwise, since `A` is a `OperationName`, `OperationName` is a [union type](#union-type) and the first element of that union type has already been checked with `A extends SyncOperationName`, then `A` must be a `AsyncOperationName` in the "else" branch of this ternary expression. Therefore, `Operation<A>` becomes `{ type: 'asyncOperation', name: A, timeout: number }`.

This is a very powerful feature that allows building complex types, especially when using [type inference in conditional types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#type-inference-in-conditional-types).

You can check the [official documentation](https://www.typescriptlang.org/docs/handbook/advanced-types.html#conditional-types) for more information about conditional types.

### Diagnostic message

When TypeScript detects an error/warning in your program, its _checker_ (an internal component of the language) generates a diagnostic message.

##### Example

```ts
// { "strict": true }

const value: string = undefined
```

```txt
Type 'undefined' is not assignable to type 'string'. (2322)
```

The `2322` number is an ID that could be used to reference the type of diagnostic message when creating an issue for example. More specifically, the `2322` ID [relates to the `Type '{0}' is not assignable to type '{1}'.` diagnostic](https://github.com/microsoft/TypeScript/blob/master/src/compiler/diagnosticMessages.json#L1285-L1288). You can also check the [wiki](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines#diagnostic-message-codes) for more information about these messages.

### Discriminated union

Discriminated unions were introduced in [TypeScript 2.0](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#tagged-union-types) as "tagged union types".

##### Example

```ts
interface Square {
  kind: 'square'
  size: number
}

interface Rectangle {
  kind: 'rectangle'
  width: number
  height: number
}

interface Circle {
  kind: 'circle'
  radius: number
}

type Shape = Square | Rectangle | Circle
```

Here, `Shape` is the discriminated union, where the discriminant - also known as _singleton property_ or _tag_ - is the `kind` property.

You can also check the official [documentation section on discriminated unions](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions).

### Distributive conditional type

This type, introduced in [TypeScript 2.8](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#distributive-conditional-types), is a conditional type whose type parameter is used as a _naked type_.

A naked type parameter is a type parameter that is not "wrapped" by another type (such as an `Array`, a `Promise`, or a tuple) in the conditional part `TypeParameter extends X ? Y : Z` of the conditional type definition.

##### Example

```ts
type ValidValue = 'a' | 'b' | 'c'

type MyDistributiveConditionalType<TypeParameter> = TypeParameter extends ValidValue ? 'yes' : 'no'

type T0 = MyDistributiveConditionalType<'a' | 'b' | 'd'>
// T0: MyDistributiveConditionalType<'a'> | MyDistributiveConditionalType<'b'> | MyDistributiveConditionalType<'d'>
// T0: ('a' extends ValidValue ? 'yes' : 'no') | ('b' extends ValidValue ? 'yes' : 'no') | ('d' extends ValidValue ? 'yes' : 'no')
// T0: 'yes' | 'yes' | 'no'
// T0: 'yes' | 'no'
```

In this example, the type parameter `TypeParameter` of the conditional type `MyDistributiveConditionalType` is used as is, it is **not wrapped** in the conditional part, hence it's used as a naked type.

The conditional type is **distributed** over each element of the union type `'a' | 'b' | 'd'` passed as the type parameter. Therefore, we end up with the union type `MyDistributiveConditionalType<'a'> | MyDistributiveConditionalType<'b'> | MyDistributiveConditionalType<'d'>`, which computes to `'yes' | 'no'`.

##### Example

```ts
type ValidValue = 'a' | 'b' | 'c'

type WrappedValue<Value> = { value: Value }

type MyConditionalType<TypeParameter> =
  WrappedValue<TypeParameter> extends  WrappedValue<ValidValue> ? 'yes' : 'no'

type T1 = MyConditionalType<'a' | 'b' | 'd'>
// T0: WrappedValue<'a' | 'b' | 'd'> extends  WrappedValue<ValidValue> ? 'yes' : 'no'
// T0: { value: 'a' | 'b' | 'd' } extends { value: 'a' | 'b' | 'c' } ? 'yes' : 'no'
// T0: 'no'
```

Here, the type parameter of `MyConditionalType` is **wrapped** inside the `WrappedValue` type in the conditional part (i.e. `WrappedValue<TypeParameter> extends X ? Y : Z`), hence it's used as a non-naked type.

The conditional type is **not distributed** over each element of the union type `'a' | 'b' | 'd'` passed as the type parameter. Thus, we get `WrappedValue<'a' | 'b' | 'd'> extends  WrappedValue<ValidValue> ? 'yes' : 'no'`, which computes to `'no'` because `'a' | 'b' | 'd' extends 'a' | 'b' | 'c'` is false.

Instead of a custom `WrappedValue` type, a 1-element **tuple** is often used in the wild to prevent distributive conditional types:

```ts
type ValidValue = 'a' | 'b' | 'c'

type MyOtherConditionalType<TypeParameter> =
  [TypeParameter] extends  [ValidValue] ? 'yes' : 'no'

type T2 = MyOtherConditionalType<'a' | 'b' | 'd'>
// T0: ['a' | 'b' | 'd'] extends [ValidValue] ? 'yes' : 'no'
// T0: ['a' | 'b' | 'd'] extends ['a' | 'b' | 'c'] ? 'yes' : 'no'
// T0: 'no'
```

### Indirect type narrowing

This is a feature added for TypeScript v4.4 (cf. [this PR](https://github.com/microsoft/TypeScript/pull/44730)). It's the ability for the compiler to [narrow the type](#type-narrowing) of a value by using intermediate `const` values defined with [type guards](#type-guard).

An example is worth a thousand words, so here we go:

##### Example

```ts
function getSize(value: unknown): number {
  const valueIsString = typeof value === 'string'
  return valueIsString ? value.length : 0
}
```

Prior to v4.4, we couldn't use `value.length` there because the compiler thought the type of `value` was still `unknown` (and `length` doesn't exist on the type `unknown`). TypeScript didn't know that we narrowed its type to `string` with the intermediate type guard `valueIsString`. As of v4.4, this will compile without any error.

Furthermore, given the original implementation of this feature (this limitation may change in the future), we can go up to **5 intermediate `consts`** before "losing" the type narrowing:

##### Example

```ts
function getSize(value: unknown): number {
  const valueIsString1 = typeof value === 'string'
  const valueIsString2 = valueIsString1
  const valueIsString3 = valueIsString2
  const valueIsString4 = valueIsString3
  const valueIsString5 = valueIsString4
  const valueIsString6 = valueIsString5
  // type narrowing of `value` lost, TS error: "Object is of type 'unknown'.(2571)"
  return valueIsString6 ? value.length : 0
}
```
([playground](https://www.typescriptlang.org/play?ts=4.4.0-dev.20210628#code/GYVwdgxgLglg9mABAcwKZQMowF6oBQBuAhgDYioBci4A1mHAO5gCUVYIAtgEaoBOiAbwBQiRBAQBnKImJlUASQkYovGGGQBGRAF5EUAJ4AHVHGAzS5Hdt0ByKavU2RYydNnlFyh8gBMO83KeKmqazuJgUgEeSsHqAMz+7gox3j5hrlHJXiEALIkWWbHIcekRbgVB3gCs+YEpuaWRSZUhAGy10dnqVc686CC8SM316u0A-JkAdCSo6lAAFohUAAxCAL5AA))

### IntelliSense

IntelliSense is the ability for the IDE - thanks to the language - to provide valuable hints for the developer depending on the context they're in, such as relevant and accurate autocompletions and code transformations. Usually, IDEs provide this feature via the `ctrl + space` combination of keys.

### Intersection type

Intersection types were introduced in [TypeScript v1.6](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-6.html#intersection-types) as a complement to [union types](#union-type). This allows developers to type a value that is both a `A` and a `B`.

##### Example

```ts
interface WithName {
  name: string
}

interface WithAge {
  age: number
}

type User = WithName & WithAge
```

The `User` type is computed as `{ name: string, age: number }`, which is the intersection of both types `WithName` and `WithAge`.

The intersection of types that have nothing in common results in the `never` type, introduced in [TypeScript v2.0](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#the-never-type).

##### Example

```ts
type A = 'a' & 'b'

type B = { name: string; age: number } & { name: number; adult: boolean }
```

Here, `A` is `never`, because the [string literals](#literal-type) `'a'` and `'b'` have nothing in common.

For `B`, its type results in `{ name: never, age: number, adult: boolean }` because `string` and `number` have nothing in common. As one of its properties type is `never`, there's no way to create a value whose type is `B`, because no value can be assigned to the `never` type.

### Intrinsic type

An intrinsic type is a type whose implementation is provided by the TypeScript compiler. It's a type that cannot be expressed by any feature provided by the type system of the language.

It was first introduced [in this PR](https://github.com/microsoft/TypeScript/pull/40580) by a TypeScript maintainer for the v4.1 release. This PR adds 4 intrinsic string types to TypeScript:

```ts
type Uppercase<S extends string> = intrinsic;
type Lowercase<S extends string> = intrinsic;
type Capitalize<S extends string> = intrinsic;
type Uncapitalize<S extends string> = intrinsic;
```

More intrinsic types could be added in the future.

### Literal type

Literal types were introduced in [TypeScript v1.8](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-8.html#string-literal-types). They are meant to expect only a specific set of strings, numbers or boolean, instead of "_any_ string, number or boolean".

##### Example

```ts
type Scheme = 'http' | 'https'

const prependWithScheme = (scheme: Scheme, domain: string, path: string): string => `${scheme}://${domain}/${path}`

prependWithScheme('http')
prependWithScheme('https')
prependWithScheme('')
```

Here, no TypeScript error is raised for `prependWithScheme('http')` and `prependWithScheme('https')`. Even better, the IDE knows which values are available for autocompletion. However, an error is raised for `prependWithScheme('')` as the empty string `''` is not available in the string literal type `Scheme`.

##### Example

```ts
interface SuccesfulResponse {
  successfulRequest: true
  status: 200 | 201 | 202
  data: unknown
}

interface UnsuccesfulResponse {
  successfulRequest: false
  status: 400 | 500
  errorMessage: string
}

const handleResponse = (response: SuccesfulResponse | UnsuccesfulResponse): void => {
  if (response.successfulRequest) {
    console.log(`Status: ${response.status}, data: ${response.data}`)
  } else {
    console.log(`Status: ${response.status}, error message: ${response.errorMessage}`)
  }
}
```

### Mapped type

Mapped types were introduced in [TypeScript v2.1](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html#mapped-types). They can be used to make _uniform type mapping_, allowing a developer to transform a type into another one of the same "form":

- A mapped type can use a string or subset of a string (i.e. [string literal type](#literal-type)) to build an object thanks to the built-in `Record` mapped type:

  ```ts
  type A = Record<'firstname' | 'lastname' | 'surname', string>
  /**
   * {
   *   firstname: string,
   *   lastname: string,
   *   surname: string
   * }
   */
  ```

- A mapped type can be a subset of the original type, e.g. using `Pick` or `Omit` (cf. example below).
- A mapped type can change the _optionality_ of all the properties of the original type using `Required`, `Nullable` or `Partial`.
- A mapped type can change the _visibility_ of all the properties of the original type using `Readonly`.

##### Example

```ts
interface Config {
  address: string
  port: number
  logsLevel: 'none' | 'error' | 'warning' | 'info' | 'debug'
}

type A = Partial<Config>
type B = Omit<Config, 'address' | 'logsLevel'>
type C = Pick<Config, 'logsLevel'>
type D = Readonly<Config>
```

Types `A`, `B`, `C` and `D` are all uniform mappings of `Config` thanks to the built-in mapped types `Partial`, `Omit`, `Pick` and `Readonly`:

- `A` is computed as:
  ```ts
  {
    address?: string | undefined,
    port?: number | undefined,
    logslevel?: 'none' | 'error' | 'warning' | 'info' | 'debug' | undefined
  }
  ```
- `B` is computed as:

  ```ts
  {
    port: number
  }
  ```

- `C` is computed as:

  ```ts
  {
    logslevel: 'none' | 'error' | 'warning' | 'info' | 'debug'
  }
  ```

- `D` is computed ad:
  ```ts
  {
    readonly address: string,
    readonly port: number,
    readonly logslevel: 'none' | 'error' | 'warning' | 'info' | 'debug'
  }
  ```

There is a [section in the official documentation](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types) about mapped types.

### Type alias

A type alias attaches a name to the definition of a type, no matter its complexity. In addition to naming function and object types such as _interfaces_, type aliases can be used to name primitives, unions, tuples... Any type actually.

##### Example

```ts
type Name = string
type LazyName = () => string

interface User_1 {
  name: string
  age: number
}

type User_2 = {
  name: string
  age: number
}

interface Predicate_1<A> {
  (value: A): boolean
}

type Predicate_2<A> = (value: A) => boolean
```

You can go to the [official documentation](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases) regarding type aliases for more information.

### Type assertion

[Type assertions](https://www.typescriptlang.org/docs/handbook/basic-types.html#type-assertions), frequently mistakenly named _type casting_, let the developer tell the compiler "trust me, I know a type more suitable for this value than the one you inferred". Contrary to casting in other languages, assertions don't change the structure of the value, they don't have any impact at runtime. The language assumes the developer did the runtime checks before using a type assertion, so it's the developer's responsibility to make sure a value has the correct type when using an assertion.

There are 2 ways to use a type assertion, either by using the `as X` keyword, or the `<X>` syntax.

##### Example

```ts
const name: unknown = 'Bob'

const strLength = (name as string).length
// or
const altStrLength = (<string>name).length
```

Bear in mind that, if the type inferred by TypeScript and the type assertion don't overlap, then the compiler will raise an error. To avoid this, either the type assertion must be a subtype of the type inferred by TypeScript, or the other way around.

```ts
const value = 42 // inferred as `number`

const res = (value as string).length
/** TS error 2352 on `value as string`:
 * Conversion of type 'number' to type 'string' may be a mistake because neither type sufficiently overlaps with the other.
 * If this was intentional, convert the expression to 'unknown' first.
 */
```

### Type check

Type checking is one of the features available with the TypeScript language. The type checker verifies the semantics of the types of one's project to make sure the program is "correct" type-wise. Other features included in TypeScript are code parsing, code transformations (TS/JS => JS), language service for tools that allow e.g. [IntelliSense](#intellisense).

If a line of code doesn't type check then a [diagnostic](#diagnostic-message) is generated and displayed as an error/warning to the developer.

##### Example

```ts
const name: string = 12
```

Here, the type checker tells us there is an error for the type of `name`: `Type '12' is not assignable to type 'string'.`.

### Type guard

> "A type guard is some expression that performs a runtime check that guarantees the type in some scope."
>
> - From the [official documentation](https://www.typescriptlang.org/docs/handbook/advanced-types.html#user-defined-type-guards).

We can use the `typeof` operator or a [type predicate](#type-predicate) to enable _type guards_. It also allows for a type to be [narrowed](#type-narrowing) in a conditional branch.

##### Example

```ts
declare const value: string | number

const getNumberOfChars = (s: string): number => s.length

const isGreaterThan = (n: number, bound: number): boolean => n > bound

if (typeof value === 'string') {
  console.log(`Number of chars in ${value}: ${getNumberOfChars(value)}.`)
} else if (typeof value === 'number') {
  console.log(`Is ${value} greater than 20? ${isGreaterThan(value, 20) ? 'Yes' : 'No'}.`)
} else {
  throw 'Impossible, perhaps the archives are incomplete'
}
```

By checking the type of `value` with `typeof`, TypeScript knows that in the `if` branch, `value` must have _that_ type and not the others:

- In the `if (typeof value === 'string') { ... }` branch, `value` type is narrowed from `string | number` to `string`.
- In the `if (typeof value === 'number') { ... }` branch, `value` type is narrowed from `string | number` to `number`.
- In the `else { ... }` branch, since all the possible types have been checked in the previous `if` conditions, `value` type is narrowed from `string | number` to `never` (it "can't" happen since all the possible cases have been checked already).

### Type inference

Type inference is the ability for the TypeScript compiler to appropriately **guess** the type of a value, without manually specifying the type for that value.

##### Example

```ts
const username = 'Bob'
const port = 8080
const names = ['Bob', 'Henri', 'Elizabeth']
const finiteNames = ['Bob', 'Henri', 'Elizabeth'] as const
```

- Type of `username` is the [string literal type](#literal-type) `'Bob'`
- Type of `port` is the [number literal type](#literal-type) `8080`
- Type of `names` is `string[]`
- Type of `finiteNames` is `readonly ['Bob', 'Henri', 'Elizabeth']`, in other words a _readonly_ tuple of 3 elements

### Type narrowing

It's the ability for the TypeScript language to restrict the type of a value to a subset of that type.

##### Example

```ts
type A = { kind: 'a'; arg1: 'hey' } | { kind: 'b'; arg1: 'Hello'; arg2: 'World' } | { kind: 'c' }

declare const value: A

if (value.kind === 'a') {
  console.log(`Value of kind ${value.kind} has argument ${value.arg1}`)
} else if (value.kind === 'b') {
  console.log(`Value of kind ${value.kind} has arguments ${value.arg1} and ${value.arg2}`)
} else {
  console.log(`Value of kind ${value.kind} has no argument`)
}
```

- If `value.kind === 'a'` is true, then `value` type is `{ kind: 'a', arg1: 'hey' }`
- Otherwise, if `value.kind === 'b'` is true, then `value` type is `{ kind: 'b', arg1: 'Hello', arg2: 'World' }`
- Otherwise, since we've already handled the cases `{ kind: 'a', arg1: 'hey' }` and `{ kind: 'b', arg1: 'Hello', arg2: 'World' }`, only the last one of the [discriminated union `A`](#discriminated-union) is left, i.e. `{ kind: 'c' }`

More examples are available in the [type guard](#type-guard) section.

### Type predicate

The developer can define type guards specific to their domain by returning a _type predicate_ to a function. A type predicate uses the `paramName is Type` syntax.

##### Example

```ts
interface User {
  name: string
  age: number
}

// You can ignore this function for the sake of this example
const hasOwnProperty = <A extends {}, B extends PropertyKey>(obj: A, prop: B): obj is A & Record<B, unknown> =>
  obj.hasOwnProperty(prop)

const isUser = (v: unknown): v is User =>
  typeof v === 'object' &&
  v !== null &&
  hasOwnProperty(v, 'name') &&
  typeof v.name === 'string' &&
  hasOwnProperty(v, 'age') &&
  typeof v.age === 'number'

declare const value: unknown

if (isUser(value)) {
  console.log(`User(name = ${value.name}, age = ${value.age})`)
} else {
  throw `Invalid user provided: ${JSON.stringify(value)}`
}
```

We created the `v is User` type predicate as the returned value of the `isUser` function, which tells TypeScript that if `isUser(value)` returns true, then `value` is guaranteed to be a `User` in the `if` branch, otherwise it will keep the initial type (`unknown` here).

Some other examples of type predicates are available in the [TypeScript documentation](https://www.typescriptlang.org/docs/handbook/advanced-types.html#using-type-predicates).

### Union type

Union types were introduced in [TypeScript v1.4](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-4.html#union-types). It's a way of expressing mulitple possible types for a given value.

##### Example

```ts
declare const value: string | number | boolean
```

Here, `value` type can be either `string`, `number` or `boolean`.
