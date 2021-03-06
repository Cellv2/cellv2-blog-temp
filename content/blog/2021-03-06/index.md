---
title: "TypeScript class, static properties and intellisense"
date: "2021-03-06T14:59:50.745Z"
description: "Using class expressions and not seeing static properties in intellisense?"
---

I've been using TypeScript a little while now, and bumped into an error a while ago which said `Class 'MyClass' incorrectly implements interface 'MyClassConstructor'. Type 'MyClass' provides no match for the signature 'new (): any'.`

I asked Lord Google about it, and I was brought to [this TypeScript docs page](https://www.typescriptlang.org/docs/handbook/interfaces.html#class-types). After skimming through I decided that the easiest way to get around it would just be to use a [class expression](https://www.typescriptlang.org/docs/handbook/interfaces.html#difference-between-the-static-and-instance-sides-of-classes), with the example provided by the TypeScript docs being the below:

```typescript
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}

interface ClockInterface {
    tick(): void;
}

const Clock: ClockConstructor = class Clock implements ClockInterface {
    constructor(h: number, m: number) {}
    tick() {
        console.log("beep beep");
    }
};

let clock = new Clock(12, 17);
clock.tick();
```

I've been using class expressions to implement both a constructor and interface in the same line pretty much ever since.

Jump to just now, I'm writing up a new class and was thinking about whether a static property would be useful, so had something like the below:

```typescript
interface MyClassConstructor {
    new (): MyClassInterface;
}

interface MyClassInterface {
    myFn(): string;
}

export const MyClass: MyClassConstructor = class MyClass
    implements MyClassInterface {
    myFn = () => {
        return "1";
    };

    static staticString = "theClassStaticString";
};
```

When I went to instantiate this, everything worked as expected. However, when I wanted to access the staticString, the TypeScript intellisense pulled up nothing.

```typescript
const instance = new MyClass();
instance.myFn(); // this is fine!
MyClass.staticString; // wasn't there?
```

TypeScript wasn't happy with me adding the `staticString` property to the `MyClassInterface` either, but when I removed the class expression the intellisense picked up the static property without issues at all (note that the class expression and class have the same name, so it would fall back to the actual class declaration).

I thought maybe it was some strangeness around the expression itself, so I split the two out and found something interesting:

```typescript
interface MyClassConstructor {
    new (): MyClassInterface;
    staticString: string;
}

interface MyClassInterface {
    myFn(): string;
}

export class MyClass implements MyClassInterface {
    myFn = () => {
        return "1";
    };

    static staticString = "theStaticString";
}

export const MyClass_Expression: MyClassConstructor = MyClass;

const instance = new MyClass();
instance.myFn(); // this is still fine
MyClass.staticString; // this is now also fine!

const instance_expression = new MyClass_Expression();
instance_expression.myFn(); // this is fine as well!
MyClass_Expression.staticString; // nope, still not here?
```

While I'm sure this would have compiled correctly with a `//@ts-ignore` or `//@ts-expect-error` and given me 'theClassStaticString', I wanted to understand what was going on. So what gives? I (eventually) found the page which I'd read before which said to use class expressions in the first place, and reading it again made me realise what was going on:

_"When working with classes and interfaces, it helps to keep in mind that a class has two types: the type of the static side and the type of the instance side. [...] This is because when a class implements an interface, only the instance side of the class is checked. Since the constructor sits in the static side, it is not included in this check. Instead, you would need to work with the static side of the class directly."_

So, we have a _static_ and an **instance** side of things, with the _static_ side being handled by the _constructor interface_ (assigned to the class expression in my case) and the **instance** side which is handled by the **interface that MyClass implements**. So, all we do is add the 'staticString' property to the constructor interface because that is the _static_ side of the class.

```typescript
interface MyClassConstructor {
    new (): MyClassInterface;
    staticString: string;
}

interface MyClassInterface {
    myFn(): string;
}

export const MyClass: MyClassConstructor = class MyClass
    implements MyClassInterface {
    myFn = () => {
        return "1";
    };

    static staticString = "theStaticString";
};

const instance = new MyClass();
instance.myFn(); // still no problems here
MyClass.staticString; // this is still fine even when we're using a class expression again
```

Honestly, understanding this made a few things click around TypeScript classes, especially when I've been using dependency injection with them. And I get to keep with my habit of using class expressions wherever I can! :smile:
