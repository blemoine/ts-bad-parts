---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: TypeScript - the bad parts
info: |
  ## Presentation about things to pay special attention to in TypeScript
  ## Because they can lead to confusion or even break the typechecking
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true

canvasWidth: 650
colorSchema: light
---

# TypeScript <br /> the Bad Parts

---
layout: intro
---
# [Benoit Lemoine](https://ca.linkedin.com/in/benoit-lemoine-b63766a0)

<img src="/img/Moloch.jpg" class="w-50 ml-10 border float-right"  />

* Tech lead (manager) at [Busbud](https://busbud.com/)
* Co-organizer of [Montreal TypeScript Meetup](https://www.meetup.com/typescript-montreal/)

---

# The goals of TypeScript


<v-clicks>

- Improve auto-completion
- Reduce number of runtime errors
- Improve self-documentation of the code
 
</v-clicks>

<p class="text-xl" v-click>But it comes at a cost</p>

<v-clicks>

- Verbosity
- New syntax to learn
- increase of build time

</v-clicks>



--- 

# The goal of... this presentation



<v-switch unmount="false">
  <template #0> <span>complain about trade-offs made on purpose</span></template>
  <template #1> <del>complain about trade-offs made on purpose</del></template>
</v-switch>

<del v-click>complain about trade-offs made on purpose</del>

<span v-click>talk about the parts where TS falls short of reaching its goal</span>

---
layout: center
class: text-center
---

# Soundness

---

## TypeScript is unsound...


<v-clicks>

* `sound`: all incorrect programs are rejected
* `unsound`: some incorrect programs compiles without any errors
* TypeScript is unsound _by design_
* Knowing the source of the unsoundness becomes crucial for trusting the language

</v-clicks>

---

## ... but we can make it _sounder_

In your `tsconfig.json` file, ensure that:

<v-clicks>

* `strict: true` is set - by default on new project
* Lots of options to improve type checking: https://www.typescriptlang.org/tsconfig/#Type_Checking_6248
* new options are regularly added

</v-clicks>

---
layout: center
class: text-center
---

## What are the sources of _unsoundness_?


---

## Null Checks

````md magic-move

```typescript

const text: string = null;

text.slice(0, 1);

```

```typescript

const text: string = null;

// Crashes at runtime with 
// Uncaught TypeError: text is null
text.slice(0, 1);

```

```typescript
// With `strictNullChecks: true` in `tsconfig.json`
// Compilation error:
// type 'null' is not assignable to type 'string'.
const text: string = null;

text.slice(0, 1);

```

```typescript
// With `strictNullChecks: true` in `tsconfig.json`
const text: string | null = null;

text?.slice(0, 1);

```

````

<span v-click>`strictNullChecks` is included in `strict`</span>

---
zoom: 0.91
---

## `any`

<v-clicks>


* `any` can be set in anything and anything can be set in `any`
* `any` is a way to say to the compiler _"I know better than you"_
* exists historically to help with JS codebase migration

</v-clicks>

<v-click>

```typescript
const a: string = true as any;
const b: any = 1;

// Runtime error -> getName is not a function
a.getName();
b.getName();

```

</v-click>



<span class="text-2xl" v-click>don't use `any` </span><span class="text-2xl" v-click>, unless you're 100% sure you can't use a proper type</span>


 



---

## What's worst than `any`s?

<v-clicks>

implicit `any`s

</v-clicks>

<v-click>

````md magic-move

```typescript

function getName(dog): string {
  return dog.name;
}

getName(null);

```

```typescript

function getName(dog): string {
  return dog.name;
}

// crash at runtime:
// dog is null 
getName(null);

```

```typescript

// with `noImplicitAny: true` in `tsconfig.json`

// Compilation error:
// Parameter 'dog' implicitly has an 'any' type.
function getName(dog): string {
  return dog.name;
}

getName(null);

```

```typescript
// `noImplicitAny` won't save you from explicit any
function getName(dog: any): string {
  return dog.name;
}

// crash at runtime:
// dog is null 
getName(null);

```


````

</v-click>


--- 

## What's worst than implicit `any`s?

<v-clicks>

Built-in functions returning `any`



````md magic-move

```typescript
type Dog = { name: string };
const a: Dog = JSON.parse("1");

const b: Promise<Dog> = fetch("https://wwww.my.beautiful.api")
                          .then(response => response.json());

```

```typescript

const a: unknown = JSON.parse("1");

const b: Promise<unknown> = fetch("https://wwww.my.beautiful.api")
                              .then(response => response.json());

```

````

[TS-reset](https://github.com/mattpocock/ts-reset) fixes the default behaviour 

But now, what should I do with those `unkwnown`s?


</v-clicks>



---

##  Type Assertions (cast)

````md magic-move

```typescript

type Dog = { name: string };

const foo: unknown = JSON.parse("1");
const bar: Dog = foo as Dog;

// crash at runtime
// bar.name is undefined 
bar.name.toUpperCase()

```

````

<v-clicks>

*  never use `as` casts...
* ... except if you control _at runtime_ that the cast is sound

</v-clicks>



---
zoom: 0.9
---

## Type Guards

A way to do a more type safe cast.


````md magic-move

```typescript

type Dog = { name: string; }

function isDog(x: unknown): x is Dog {
}

```

```typescript

type Dog = { name: string; }

// Type Guards function _are_ unsafe 
// and must be thouroughly tested
function isDog(x: unknown): x is Dog {
    return typeof x === 'object' && !!x && 'name' in x && 
             typeof x.name === 'string'
}


```


```typescript

type Dog = { name: string; }

// Type Guards function _are_ unsafe 
// and must be thouroughly tested
function isDog(x: unknown): x is Dog {
    return typeof x === 'object' && !!x && 'name' in x && 
             typeof x.name === 'string'
}
const foo: unknown = JSON.parse("1");
if (isDog(foo)) {
    console.log(foo.name)
}

```

````

<v-click> 

In the real world, use [zod](https://zod.dev/)

</v-click>


---
zoom: 0.95
---

## Validation with zod



```typescript
import * as z from 'zod';

const Dog = z.object({ name: z.string() })

type Dog = z.infer<typeof Dog>;
//   ^?  { name: string }

const foo: unknown = JSON.parse("1");

const validated_foo = Dog.safe_parse(foo);
if(validated_foo.success) {
    console.log(validated_foo.name)
}

```

---

## Record/Array lookups


````md magic-move

```ts
type Dog = { name: string; }
const dogs: Array<Dog> = []

// Runtime error:
// dogs[0] is undefined 
console.log(dogs[0].name)
```

```ts
// with `noUncheckedIndexedAccess: true` in `tsconfig.json`
type Dog = { name: string; }
const dogs: Array<Dog> = []

// Compilation error:
// Object is possibly 'undefined'.
console.log(dogs[0].name)
```

```ts
// with `noUncheckedIndexedAccess: true` in `tsconfig.json`
type Dog = { name: string; }
const dogs: Array<Dog> = []

if (dogs[0]) {
  console.log(dogs[0].name)
}
// or
console.log(dogs[0]?.name)
```

````

---
zoom: 0.85
---

## What if you _know_ the value at this index exists?

And you don't want to deal with `undefined`s or `if`s

<v-click>

````md magic-move

```ts
type Dog = { name: string; }

function getFirstDogName(dogs: Array<Dog>): string {
    // Object is possibly 'undefined'
    return dogs[0].name
}

getFirstDogName([{name: 'Sif'}]);

```

```ts
type Dog = { name: string; }

type NonEmptyArray<T> = [T, ...Array<T>];

function getFirstDogName(dogs: NonEmptyArray<Dog>): string {
    return dogs[0].name
}

getFirstDogName([{name: 'Sif'}]);

```

```ts
type NonEmptyArray<T> = [T, ...Array<T>];

function isNonEmptyArray<T>(arr: Array<T>): arr is NonEmptyArray<T> {
  return arr.length > 0
}
```



````

</v-click>



---

## Function multivariance

````md magic-move

```ts

function BottomComponent({onclick}: {onclick: () => void}) {
   return <button onClick={onclick}>TEST&lt;/button>
}

```

```ts

function BottomComponent({onclick}: {onclick: () => void}) {
   return <button onClick={onclick}>TEST&lt;/button>
}

const myfn = (now?:Date) => console.log('it is now', now);
function TopComponent() {
  return <BottomComponent onclick={myfn} ></BottomComponent>
}


```

```ts


function BottomComponent({onclick}: {onclick: () => void}) {
   return <button onClick={onclick}>TEST&lt;/button>
}

const myfn = (now?:Date) => console.log('it is now', now);
function TopComponent() {
  return <BottomComponent onclick={myfn} ></BottomComponent>
}

// it is now SyntheticBaseEvent {
//    _reactName: 'onClick', _targetInst: null, type: 'click',
//        nativeEvent: PointerEvent, target: button, …}

```

````


---

## (Co)Variance

````md magic-move

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

// Cat is a subtype of Dog
const test1: Dog = cat;

```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogs: Array<Dog> = [dog];
const cats: Array<Cat> = [cat];

```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogs: Array<Dog> = [dog];
const cats: Array<Cat> = [cat];

const test: Array<Dog> = cats;


```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogs: Array<Dog> = [dog];
const cats: Array<Cat> = [cat];

// Type 'Array<Dog>' is not assignable to type 'Array<Cat>'.
//  Property 'size' is missing in type 'Dog' but required in type 'Cat'.
const test: Array<Cat> = dogs;

```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogs: Array<Dog> = [dog];
const cats: Array<Cat> = [cat];

// Type 'Array<Dog>' is not assignable to type 'Array<Cat>'.
//  Property 'size' is missing in type 'Dog' but required in type 'Cat'.
const test: Array<Cat> = dogs;
// Array<Cat> is subtype of Array<Dog>
// Array is covariant: 
// if X <: Y then Array<X> <: Array<Y>

```


````


---

## (Contra)Variance


````md magic-move

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: () => Dog = () => dog;
const catFn: () => Cat = () => cat;

```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: () => Dog = () => dog;
const catFn: () => Cat = () => cat;

const test: () => Dog = catFn;

```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: () => Dog = () => dog;
const catFn: () => Cat = () => cat;

// Type '() => Dog' is not assignable to type '() => Cat'.
//   Property 'size' is missing in type 'Dog' but required in type 'Cat'.
const test: () => Cat = dogFn;

```

```ts

type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: () => Dog = () => dog;
const catFn: () => Cat = () => cat;

// Type '() => Dog' is not assignable to type '() => Cat'.
//   Property 'size' is missing in type 'Dog' but required in type 'Cat'.
const test: () => Cat = dogFn;
// () => Dog is a subtype of () => Cat
// Return value of functions are contravariant: 
// if X <: Y then () => Y <: () => X

```


````








---

## What about function parameters?

````md magic-move

```ts
type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: (d: Dog) => void  = () => {};
const catFn: (c: Cat) => void = () => {};
```

```ts
type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: (d: Dog) => void  = () => {};
const catFn: (c: Cat) => void = () => {};

const test: (d: Dog) => void = catFn;
const test2: (c: Cat) => void = dogFn;
```

```ts
type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: (d: Dog) => void  = () => {};
const catFn: (c: Cat) => void = () => {};

const test: (d: Dog) => void = catFn;
const test2: (c: Cat) => void = dogFn;

// No errors?
```

```ts
type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const dogFn: (d: Dog) => void  = () => {};
const catFn: (c: Cat) => void = () => {};

const test: (d: Dog) => void = catFn;
const test2: (c: Cat) => void = dogFn;

// No errors -> in TS function parameters are multi-variant
```

```ts
type Dog = { name: string};
type Cat = { name: string; size: number };

const cat: Cat = {name: 'Serosh', size:10};
const dog: Dog = {name: 'Sif'};

const catFn: (c: Cat) => number = (c: Cat) => c.size;

const test: (d: Dog) => number = catFn;

test(dog); // returns undefined

```

````

---
zoom: 0.95
---

## Function multivariance

* Function _should be_ covariant on parameters.

<v-click>


````md magic-move

```ts
// With `strictFunctionTypes: true` in tsconfig.json

type Dog = { name: string};
type Cat = { name: string; size: number };

const catFn: (c: Cat) => number = (c: Cat) => c.size;
// Compilation error:
// Type '(c: Cat) => number' is not assignable to type '(d: Dog) => number'.
//  Types of parameters 'c' and 'd' are incompatible.
//    Property 'size' is missing in type 'Dog' but required in type 'Cat'.
const test: (d: Dog) => number = catFn;



```

```ts
// With `strictFunctionTypes: true` in tsconfig.json

type Dog = { name: string};
type Cat = { name: string; size: number };

type CatFn = { getSize(c: Cat): number }
type DogFn = { getSize(c: Dog): number }


const catFn: CatFn = {getSize: (c: Cat) => c.size};

const test: DogFn = catFn;
```

````

</v-click>


---
layout: center
class: text-center
---

# Syntax

---
zoom: 0.93
---

## TypeScript introduces _lot _ of syntax stuffs

<div class="two-columns grid grid-cols-2 gap-2 my-3">
<div class="col-left">
```typescript
interface Animal {
    name: string
}
abstract class Dog implements Animal {
  public name:string;
  public constructor(name: string) {
    this.name = name.toUpperCase();
  }
}
```
</div>

<div class="col-right" v-click>
```javascript
class Dog {
  constructor(name) {
    this.name = name.toUpperCase();
  }
}
```
</div>

</div>

<p v-click>Goal was to have the new syntax only at <em>type level</em> but...</p>

---

### `enum`s / `namespace`s

````md magic-move

```ts

namespace NS {
   enum Planet { Earth, Mars } 
}

```

```ts
var NS;
(function (NS) {
    let Planet;
    (function (Planet) {
        Planet[Planet["Earth"] = 0] = "Earth";
        Planet[Planet["Mars"] = 1] = "Mars";
    })(Planet || (Planet = {}));
})(NS || (NS = {}));
```

````



<v-clicks>

* `enum` and `namespace` are _runtime syntax_
* Node 22.6 adds `--experimental-strip-types`;
* TS 5.8 adds `--erasableSyntaxOnly`

</v-clicks>

---

#### `enum` alternative - Union Types

````md magic-move


```ts
enum Planet { 
  Earth = "Earth", 
  Mars = "Mars" 
}

const all_planets = Object.values(Planet);
//    ^? Planet[]

function isEarth(planet: Planet): boolean {
    return planet === Planet.Earth;
}
```

```ts
const Planet = {
  Earth: "Earth",
  Mars: "Mars"
} as const

const all_planets = Object.values(Planet);
//    ^? ("Earth" | "Mars")[]

type Planet = typeof all_planets[number];
//   ^? "Earth" | "Mars" 

function isEarth(planet: Planet): boolean {
    return planet === Planet.Earth;
}

```

````

----

#### `namespace` alternative


````md magic-move


```ts

// in ns.ts
export namespace NS {
   export const a_value = "test"
}

// in another file
import { NS } from 'ns';
console.log(NS.a_value);
```

```ts

// in ns.ts
export const NS = {
   a_value: "test"
}

// in another file
import { NS } from 'ns';
console.log(NS.a_value);
```

```ts

// in ns.ts
export const a_value = "test"

// in another file
import * as NS from 'ns';
console.log(NS.a_value);
```

````




---

## TypeScript is Structurally Typed <span v-click="2"> - Except when it's not</span>


````md magic-move

```ts
class Dog {
   constructor(public name:string){}
}

const dog: Dog = { name: 'Sif'};
```

```ts
class Dog {
   constructor(public name:string){}
}

class Cat {
   constructor(public name:string){}
}

const dog: Dog = new Cat('Sif');
```

```ts
class Dog {
   constructor(private name:string){}
}

class Cat {
   constructor(private name:string){}
}

const dog: Dog = new Cat('Sif');
// Type 'Cat' is not assignable to type 'Dog'.
//   Types have separate declarations of a private property 'name'.
```

````

---
zoom: 0.9
---

## Service Pattern


````md magic-move

```ts

class UserService {
    constructor(private repository: UserRepository) {}
    getName(): string { return this.repository.getName(); }
}

class UserController {
    constructor(private service: UserService) {}
    getName(): string { return this.service.getName(); }
}


```

```ts


class UserService {
    constructor(private repository: UserRepository) {}
    getName(): string { return this.repository.getName(); }
}

class UserController {
    constructor(private service: UserService) {}
    getName(): string { return this.service.getName(); }
}

// In tests

const fakeUserService: UserService = {  getName() { return ''; } }
const controller = new UserController(fakeUserService)

```

```ts


class UserService {
    constructor(private repository: UserRepository) {}
    getName(): string { return this.repository.getName(); }
}

class UserController {
    constructor(private service: UserService) {}
    getName(): string { return this.service.getName(); }
}

// In tests
// Property 'repository' is missing in type '{ getName(): string; }' 
//   but required in type 'UserService'.
const fakeUserService: UserService = {  getName() { return ''; } }
const controller = new UserController(fakeUserService)

```

```ts

interface UserService {
  getName(): string
}
class UserServiceImpl implements UserService {
    constructor(private repository: UserRepository) {}
    getName(): string { return this.repository.getName(); }
}

class UserController {
    constructor(private service: UserService) {}
    getName(): string { return this.service.getName(); }
}

// In tests
const fakeUserService: UserService = {  getName() { return ''; } }
const controller = new UserController(fakeUserService)

```



```ts

type PublicInterface<T> = {
  [K in keyof T]: T[K] extends new (...args: unknown[]) => unknown ? never : T[K];
};
class UserServiceImpl {
    constructor(private repository: UserRepository) {}
    getName(): string { return this.repository.getName(); }
}
type UserService = PublicInterface<UserServiceImpl>

class UserController {
    constructor(private service: UserService) {}
    getName(): string { return this.service.getName(); }
}

// In tests
const fakeUserService: UserService = {  getName() { return ''; } }
const controller_under_test = new UserController(fakeUserService)

```

````



--- 

## End Note on Structural Typing

<v-clicks>

* If you want to be _sure_ your users are using your class, declare a private property. Even a fake one.


</v-clicks>



---
zoom: 0.95
---

## Multiple ways to do the same thing - types declaration


```ts

type Dog = {
    name: string;
}
// vs.
interface Dog {
    name: string;
}


```

<v-clicks>

* There is some subtle differences
* Main one is `interface`s are open to modification after declaration  
* [more info on TS doc](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)

</v-clicks>


----

## Multiple ways to do the same thing - arrays



```ts
const arr: number[] = [1,2,3];

// vs.

const arr: Array<number> = [1,2,3];

```

<v-clicks>

* no differences between the 2

</v-clicks>

----

## Multiple ways to do the same thing - casts


```ts
const input = "123";
const num = input as unknown as number;

// vs.

const input = "123";
const num = <number>(<unknown>input);

```

<v-clicks>

* second one is not well supported when in used in TSX (React)

</v-clicks>


---
zoom: 0.87
---

## Multiple ways to do the same thing - types


```ts
function getName(dog: Dog): string {
  return dog.name
}
//vs .
/**
 * @param {Dog} dog
 * @returns {string}
 */ 
function getName(dog) {
  return dog.name
}
```

<v-clicks>

* second one is more complex, but saves the compilation step
* second one forces developer to document their code
* point probably moot with node supporting `--experimental-strip-types`

</v-clicks>




---

# Conclusion


* Activate `strict` options in `tsconfig.json`
* Chose a syntax and stick to it
* Even with (mostly historical) quirks, TypeScript is still quite good


---
layout: center
class: text-center
---

# Questions ?

TODO link to the slides

<PoweredBySlidev mt-10 />
