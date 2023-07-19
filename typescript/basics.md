# TypeScript

TypeScript is a static typechecker for JavaScript programs. It's a tool that's run _before_ the code runs (static) and ensures that the types of the program are correct (typechecked).

For a simple example, suppose we have a file `greet.ts`:

```ts
const greet = (person: string, date: Date) => {
  console.log(`Hello ${person}, today is ${date}!`);
};
 
greet('Luke'); // ❌
```

We can install TypeScript and run `tsc`:

```sh
$ npm i -g typescript

added 1 package # ...
```

Then we can run `tsc` on `greet.ts`:

```sh
$ tsc --noEmitOnError greet.ts
greet.ts:5:1 - error TS2554: Expected 2 arguments, but got 1.

5 greet('Luke'); // ❌
  ~~~~~~~~~~~~~

  greet.ts:1:32
    1 const greet = (person: string, date: Date) => {
                                     ~~~~~~~~~~
    An argument for 'date' was not provided.


Found 1 error in greet.ts:5

```

`tsc` will always transpile the source TypeScript to JavaScript _even if_ there are TypeScript errors unless told otherwise using the `--noEmitOnError` flag. We could have omitted that flag and the command would have emitted a transpiled `.js` file despite the error.

In the output, we see that `tsc` found an error with our usage of the `greet` function: we didn't supply the required second `date` argument. If we're using a code editor with intellisense, this output is more or less what we'll see when using intellisense.

Now that we've identified the error in `greet.ts`, let's go back and fix it:

```ts
const greet = (person: string, date: Date) => {
  console.log(`Hello ${person}, today is ${date}!`);
};
 
greet('Luke', new Date());
```

```sh
$ tsc --noEmitOnError greet.ts
# No output
```

Now we see that there weren't errors in the output of using `tsc` on `greet.ts`. Because of this, `tsc` emitted a `greet.js` file:

```js
var greet = function (person, date) {
    console.log("Hello ".concat(person, ", today is ").concat(date, "!"));
};
greet('Luke', new Date());
```

There are few things to note in this `greet.js`.

For one, type annotations are gone. Browsers don't recognize TypeScript and so our `ts` file needs transpiled to a Javascript `js` file. This means that type annotations never change the runtime behavior of our program.

We'll notice that template string syntax converted to string concatenation. This is because `tsc` "downlevels" to a version of [ECMAScript](https://en.wikipedia.org/wiki/ECMAScript) that is ubiquitous to most browsers. Here, our resulting `js` file is ECMAScript version 3 (ES3) which only recognizes string concatenation and not template strings. This is good for us as it means that we're able to write in the latest ES versions for readability knowing that `tsc` will transpile our code to a version of `js` that supports the most browsers.

## Essential Types

### Primitive Types

The basic building block types are what we'd expect; strings, numbers, booleans, null / nil, but there are others that TypeScript recognizes at a primitive level as well:

```ts
// let name: string
let name = 'Luke';

// const age: number
const age: number = 25; // at heart

// const isAwesome: boolean
const isAwesome: boolean = true;

// const secondsUntilDeceased: bigint
const secondsUntilDeceased = BigInt(10e307);

// let access: symbol
let access: symbol;

// const nope: null
const nope: null = null;

// const yomi: undefined
const yomi: undefined = undefined;
```

### Union Types

We're able to build types by intersecting other types. Unions can be made by intersecting with any other types whether they're primitive types or higher level types. Suppose we have a `unionTypes.ts` file:

```ts
type Flag = 'auto' | 'all' | 'rec'; // union of string literals
type Options = { width: number; height: number };

const configure = (configurationArgument: Options | Flag) => {};

configure('auto'); // ✅
configure({ width: 100, height: 100 }); // ✅
configure('automatic'); // ❌
configure({ width: 50 }); // ❌
configure({ breadth: 10, height: 20 }); // ❌
```

Here we declare a type alias `Flag` with a union of string literals which we expect as a possible argument for `configurationArgument`. We declare an `Options` type alias which expects an object shape with only the required `width` and `height` entries. We then declare a function which expects an argument of a union type of either `Options` or `Flag` and so when we call `configure` with either a valid `Options` or `Flag` argument, they're valid uses of `configure` but when we call it with arguments other than is annotated, we'll see TypeScript errors:

```sh
$ tsc --noEmit unionTypes.ts
unionTypes.ts:8:11 - error TS2345: Argument of type '"automatic"' is not assignable to parameter of type 'Flag | Options'.

8 configure('automatic'); // ❌
            ~~~~~~~~~~~

unionTypes.ts:9:11 - error TS2345: Argument of type '{ width: number; }' is not assignable to parameter of type 'Flag | Options'.
  Property 'height' is missing in type '{ width: number; }' but required in type 'Options'.

9 configure({ width: 50 }); // ❌
            ~~~~~~~~~~~~~

  unionTypes.ts:2:33
    2 type Options = { width: number; height: number };
                                      ~~~~~~
    'height' is declared here.

unionTypes.ts:10:13 - error TS2345: Argument of type '{ breadth: number; height: number; }' is not assignable to parameter of type 'Flag | Options'.
  Object literal may only specify known properties, and 'breadth' does not exist in type 'Options'.

10 configure({ breadth: 10, height: 20 }); // ❌
               ~~~~~~~


Found 3 errors in the same file, starting at: unionTypes.ts:8
```

### Function Types

Function types are composed of parameter types and return types. Let's look at an example:

```ts
type GreeterFn = (a: string) => void;

// const greeter: (fn: GreeterFn) => void
const greeter = (fn: GreeterFn) => fn('Hello, World'); // ✅

greeter(console.log);
```

Here we declare a type alias `GreeterFn` that expects an argument of type `string` and has a return type of `void`. We then declare a `const` function that expects an argument of type `GreeterFn` and its implementation is such that it calls the function argument with a greeting string. We can then call that function supplying a function argument which in turn can expect a string argument, such as `console.log`. Note that since we didn't specify a return type for `greeter`, TypeScript assumes the return type to be `void`.

When declaring a function type, the parameter name is required when declaring function type aliases. If we forget to add the name, TypeScript will assume the type to be `any` but it's also helpful in that it recognizes the common mistake of omitting the parameter name for the expected type. Suppose we have a `functionTypes.ts` file:

```ts
type SomeFunction = (string) => void; // ✅ (but also ❌)
```

If we run `tsc` with the `strict` flag, we'll see helpful error hints:

```sh
$ tsc --noEmit --strict functionTypes.ts
functionTypes.ts:8:14 - error TS7051: Parameter has a name but no type. Did you mean 'arg0: string'?

8 type SomeFunction = (string) => void; // ✅ (but also ❌)
               ~~~~~~


Found 1 error in functionTypes.ts:8
```

### object

`object` is adjacent to the `any` type in that `object` refers to any value that isn’t a primitive (see above). Suppose we have a `objectTypes.ts` file:

```ts
let myObject: object;

myObject = {}; // ✅
myObject = { hi: 'hey' }; // ✅

myObject = () => {}; // ✅
myObject = []; // ✅
myObject = /foo/; // ✅

myObject = 42; // ❌
```

Here we declare a mutable `myObject` variable of type `object`. We can assign objects like we would expect but we can also assign it _any_ value other than a primitive. So we can assign it a function, an array, etc. but we can't assign it a primitive value:

```sh
$ tsc --noEmit --strict objectTypes.ts
objectTypes.ts:9:1 - error TS2322: Type 'number' is not assignable to type 'object'.

9 myObject = 42; // ❌
  ~~~~~~~~


Found 1 error in objectTypes.ts:9
```

Because any non-primitive value can be assigned to `object`, we should try to avoid using it and instead declare object type aliases or utilize the `Record` type.

### void

`void` represents the return value of functions which don’t return a value. It’s the inferred type any time a function doesn’t have any return statements. In JavaScript, a function that doesn’t return any value will implicitly return the value `undefined`. However, `void` and `undefined` are _not_ the same in TypeScript.

```ts
// const noop: () => void
const noop = () => {};
noop(); // -> undefined

// const noopExplicit: () => void
const noopExplicit = () => { return; };
noopExplicit() // -> undefined
```

There are some caveats to know about `void`. We can describe a function as a return type of `void` but that function can still return a value. If we were to use that return value in any way, though, `tsc` will throw an error. Suppose we have a `void.ts` file:

```ts
type VoidFunc = () => void;
const f: VoidFunc = () => 'hi'; // ✅ ... wait, what?
// const result: void
const result = f().length; // ❌ (but also ✅) -> 2
```

```sh
$ tsc --noEmit --strict void.ts
void.ts:4:20 - error TS2339: Property 'length' does not exist on type 'void'.

4 const result = f().length; // ❌ (but also ✅) -> 2
                     ~~~~~~


Found 1 error in void.ts:4
```

But if we were to still emit this as a `js` file, it would be valid JavaScript and `result` would have a value of `2`.

A function whose return type is `void` doesn't offend `tsc` when it actually returns a value because we need to allow those functions to be used as arguments to other functions that don't care about the return type of the function argument. For example:

```ts
// Array<T>.forEach(cb: (v: T, i: T, a: T[]) => void, thisArg?: any ): void
// Array<number>.push(...items: number[]): number
const a = [0];
[1, 2, 3].forEach((el) => a.push(el)); // ✅
```

`Array#push` returns a number (the new length of the array), but it's an acceptable argument for the callback function argument `Array#forEach` expecting a `void` return type because `Array#forEach` does nothing with the return value even if there is one.

In other words, it doesn't matter what the return value is, it could be anything but it's inconsequential and it doesn't, and shouldn't, matter.

### unknown

The `unknown` type represents any value and is similar to the `any` type, but safer because it’s offensive to `tsc` to do anything with an `unknown` value. Suppose we have a `void.ts` file:

```ts
const f = (a: any) => a.b(); // ✅
const g = (a: unknown) => a; // ✅
const h = (a: unknown) => a.b(); // ❌
```

```sh
$ tsc --noEmit --strict unknown.ts
unknown.ts:3:27 - error TS18046: 'a' is of type 'unknown'.

3 const h = (a: unknown) => a.b(); // ❌
                            ~


Found 1 error in unknown.ts:3
```

`unknown` can be useful in that it allows us to describe functions that accept any value without having `any` values in the function body.

### never

`never` in the context of a function's return value declares that the program will never exit from that function, whether it's because it raised an error or will loop forever.

```ts
const fail = (msg: string): never => {
  throw new Error(msg);
};

const groundhogDay = (): never => {
  while (true) {
    console.log('So put your little hand in mine...');
  }
}
```

`never` can also be used when we want exhaustive checks to error when we've missed a case. For example:

```ts
type Rectangle = { kind: 'rectangle'; height: number; width: number };
type Square = { kind: 'square'; length: number };
type Shape = Rectangle | Square;

// const area: (shape: Shape) => number
const area = (shape: Shape) => {
  if (shape.kind === 'rectangle') {
    return shape.height * shape.width;
  }

  if (shape.kind === 'square') {
    return shape.length * shape.length;
  }

  const _unhandled: never = shape; // ✅

  return _unhandled;
}
```

This function should never get to `_unhandled` and so the assignment of the `shape: Shape` value to it is valid because TypeScript infers that we've exhaustively checked all `Shape`s and therefore `shape` at that point would also have a type of `never`. And since `never` is only ever assignable to other `never`s, this is valid.

Now, what if we added a `Shape` but forgot to update our `area` function? TypeScript will let us know that we didn't exhaustively check all `Shape`s in `area` and `_unhandled` as a `never` type could be assigned a type of our new `Shape`. Suppose we have a `never.ts` file:

```ts
type Rectangle = { kind: 'rectangle'; height: number; width: number };
type Square = { kind: 'square'; length: number };
type Triangle = { kind: 'triangle'; base: number, height: number };
type Shape = Rectangle | Square | Triangle;

// const area: (shape: Shape) => number
const area = (shape: Shape) => {
  if (shape.kind === 'rectangle') {
    return shape.height * shape.width;
  }

  if (shape.kind === 'square') {
    return shape.length * shape.length;
  }

  const _unhandled: never = shape; // ❌

  return _unhandled;
}
```

```sh
$ tsc --noEmit --strict never.ts
never.ts:26:9 - error TS2322: Type 'Triangle' is not assignable to type 'never'.

26   const _unhandled: never = shape;
           ~~~~~~~~~~


Found 1 error in never.ts:26

```

TypeScript let us know that we have to add a case for when the shape could be the new `Triangle` shape.
