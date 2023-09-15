# Functions

## Function Type Expressions

##### types/functionTypes/functionTypeExpressions.ts

```ts
type GreeterFn = (a: string) => void;
const greeter = (fn: GreeterFn) => fn('Hello, World'); // ✅

greeter(console.log);

// Parameter name is required:
type HiFn = (string) => void; // ✅ (but also ❌)
// Parameter has a name but no type.
//   Did you mean 'arg0: string'?
// (parameter) string: any
```

---

# Generic Functions

##### types/functionTypes/genericFunctions.ts

```ts
// const firstElement: (arr: any[]) => any
const firstElement = (arr: any[]) => arr[0]; // ❌

// const firstElementToo: <Type>(arr: Type[]) => Type | undefined
const firstElementToo = <Type>(arr: Type[]): Type | undefined => arr[0]; // ✅

// const s: string | undefined
const s = firstElementToo(['a', 'b', 'c']);
// const u: undefined
const u = firstElementToo([]);

// Inference

const myMap = <In, Out>(arr: In[], func: (arg: In) => Out): Out[] => arr.map(func);

// (parameter) n: string
// const parsed: number[]
const parsed = myMap(['1', '2', '3'], (n) => parseInt(n));
```

<!-- Note that in this example (parsed), TypeScript could infer both the type of the Input type parameter (from the given string array), as well as the Output type parameter based on the return value of the function expression (number). -->

---

# Constraints

##### types/functionTypes/constraints.ts

```ts
const longest = <T extends { length: number }>(a: T, b: T) =>
  (a.length >= b.length) ? a : b;
 
// const longerArray: number[]
const longerArray = longest([1, 2], [1, 2, 3]);
// const longerString: "alice" | "bob"
const longerString = longest('alice', 'bob');

const notOK = longest(10, 100); // ❌
// Argument of type 'number' is not assignable
//   to parameter of type '{ length: number; }'
```

---

# Constraints (cont'd)

<!-- Common problem! -->

##### types/functionTypes/constraintsContd.ts

```ts
type MinLength = <T extends { length: number }>(obj: T, min: number) => T;

const minLength: MinLength = (obj, min) => {
  if (obj.length >= min) {
    return obj;
  } else {
    return { length: min };
// T '{ length: number; }' is not assignable to type 'T'.
//   '{ length: number; }' is assignable to the constraint of type 'T',
//     but 'T' could be instantiated with a different
//     subtype of constraint '{ length: number; }'.
  }
};

// 'arr' gets value { length: 6 }
const arr = minLength([1, 2, 3], 6);
// and crashes here because the returned object doesn't have a 'slice' method!
console.log(arr.slice(0)); // ❌
```

<!-- It might look like this function is OK - Type is constrained to { length: number }, and the function either returns Type or a value matching that constraint. The problem is that the function promises to return the same kind of object as was passed in, not just some object matching the constraint. If this code were legal, you could write code that definitely wouldn’t work: -->

---

# Specifying Type Arguments

##### types/functionTypes/specifyingTypeArguments.ts

```ts
const combine = <T>(a1: T[], a2: T[]): T[] => a1.concat(a2);

const arr = combine([1, 2, 3], ['hello']); // ❌
// Type 'string' is not assignable to type 'number'.

// const shhIsOk: (string | number)[]
const shhIsOk = combine<string | number>([1, 2, 3], ['hello']); // ✅
```

---

# Best Practices

##### types/functionTypes/bestPractices.ts

```ts
// Push type parameters down
const firstElement = <T extends any[]>(a: T)  => a[0]; // ❌
const b = firstElement([1, 2, 3]); // ❌ const b: any
const firstElementToo = <T>(a: T[])  => a[0]; // ✅
const a = firstElementToo([1, 2, 3]); // ✅ const a: number

// Use fewer type Parameters
const filter = <T, F extends (arg: T) => boolean>(a: T[], f: F): T[] => a.filter(f); // ❌
const filterToo = <T>(a: T[], f: (arg: T) => boolean): T[] => a.filter(f); // ✅
```

---

# Optional Parameters

##### types/functionTypes/optionalParameters.ts

```ts
// (parameter) x: number | undefined
const f = (x?: number) => x;
f(); // ✅ -> undefined
f(10); // ✅ -> 10
f(undefined); // ✅ -> undefined
f(null); // ✅ -> null

// (parameter) x: number
const g = (x: number = 5) => x;
g(); // ✅ -> 5
g(20); // ✅ -> 20
g(undefined); // ✅ -> 5
g(null); // ✅ -> null
```

---

# void

<!-- void represents the return value of functions which don’t return a value -->
<!-- It’s the inferred type any time a function doesn’t have any return statements -->
<!-- In JavaScript, a function that doesn’t return any value will implicitly return the value undefined -->
<!-- However, void and undefined are not the same thing in TypeScript -->

##### types/functionTypes/void.ts

```ts
// const noop: () => void
const noop = () => {};
noop(); // -> undefined

// const noopExplicit: () => void
const noopExplicit = () => { return; };
noopExplicit() // -> undefined

type VoidFunc = () => void;
const f: VoidFunc = () => 'hi'; // ✅ ... wait, what?
const result = f().length; // ❌ -> 2
// Property 'length' does not exist on type 'void'

// Array<T>.forEach(cb: (v: T, i: T, a: T[]) => void, thisArg?: any ): void
// Array<number>.push(...items: number[]): number
const a = [0];
[1, 2, 3].forEach((el) => a.push(el)); // ✅
```

<!-- Array#push returns a number (the new length of the array), but it's a safe substitute to use for a void-returning function. -->

<!-- Another way to think of this is that a void-returning callback type says "I'm not going to look at your return value, even if one exists". -->
<!-- In other words, it doesn't matter what the return value is, it could be anything but it's inconsequential -->

---

# object

<!-- The special type object refers to any value that isn’t a primitive (string, number, bigint, boolean, symbol, null, or undefined) -->

##### types/functionTypes/object.ts

```ts
type MyObject = object; // `any` adjacent

let myObject: MyObject;

myObject = {}; // ✅
myObject = { hi: 'hey' }; // ✅

// Because functions;
//  - have Object.prototype in their prototype chain
//  - are instanceof Object
//  - you can call Object.keys on them
//  - etc.
myObject = () => {}; // ✅

myObject = 42; // ❌
// Type 'number' is not assignable to type 'object'
```

---

# unknown

<!-- The unknown type represents any value -->
<!-- Similar to the any type, but safer because it’s not legal to do anything with an unknown value -->

##### types/functionTypes/unknown.ts

```ts
const f = (a: any) => a.b(); // ✅ (but also ❌)

const g = (a: unknown) => a.b(); // ❌
// 'a' is of type 'unknown'

const h = (a: unknown) => a; // ✅
```

 <!-- Allows you to describe functions that accept any value without having any values in your function body. -->

---

# never

<!-- Means that the function throws an exception or terminates execution of the program. -->
<!-- Has other confusing meanings -->

##### types/functionTypes/never.ts

```ts
const fail = (msg: string): never => {
  throw new Error(msg);
};
```

---

# Function

<!-- Has the special property that values of type Function can always be called but these calls return any -->

##### types/functionTypes/function.ts

```ts
// const doSomething: (f: Function) => any
const doSomething = (f: Function) => f(); // ❌ Unsafe

// const doSomethingToo: (f: () => void) => void
const doSomethingToo = (f: () => void) => f(); // ✅ Safer
```

---

# Rest Parameters And Arguments

<!-- implicitly any[] instead of any -->
<!-- type annotations must be given in the form Array<T> or T[], or a tuple type (later) -->
<!-- Arrays are mutable so it's best to freeze them -->

<!-- TODO explain why the failure -->

##### types/functionTypes/restParametersAndArguments.ts

```ts
// const myMax: (...nums: number[]) => number
const myMax = (...nums: number[]) => Math.max(...nums);

const theseNumbers = [1, 2];
const thisMax = myMax(...theseNumbers); // ✅
const thatMax = myMax(...[3, 4, 5]); // ✅
const theOtherMax = myMax(6, 7, 8, 9); // ✅

// const fullName: (fname: string, lname: string) => string
const fullName = (fname: string, lname: string) => `${fname} ${lname}`;

const myNames = ['Luke', 'Winningham'];
const myFullName = fullName(...myNames); // ❌
// A spread argument must either have a tuple type or be passed to a rest parameter.

const myNamesToo = ['Luke', 'Winningham'] as const; // `as const`
const myFullNameToo = fullName(...myNamesToo); // ✅

const myNamesAgain = ['Luke', 'Winningham'] as [string, string];
const myFullNameAgain = fullName(...myNamesAgain); // ✅
```

<!-- The failure is because TypeScript doesn't know that the array length matches the call signature requirements because the array could mutate at runtime. We can 'freeze' it with `as const` or we can type the array as Tuple type with `as [string, string]`, more on Tuple type later -->

---

# Parameter Destructuring

##### types/functionTypes/parameterDestructuring.ts

```ts
type User = { id: string, name: string; age: number; iq: number };

// Require `id`, all other properties optional
type GetUserParams = Partial<User> & Pick<User, 'id'>;

const getUser = ({ id, name = '', age, iq: userIQ }: GetUserParams): User => {
  const userAge = age === undefined ? 39 : age; // ❌ boo hiss

  const averageIQ = getAverageIQ();
  const iq = userIQ ?? averageIQ; // ✅

  return { id, name, age: userAge, iq };
};

const user = getUser({ id: 'a1b2c3d4' }); // ✅
```
