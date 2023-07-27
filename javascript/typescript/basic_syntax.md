---
layout: page
title: "Typescript Basic Syntax"
permalink: /javascript/typescript/basic_syntax
---

## Declare variable and specify the type

 If you just assign a variable for example `let x = 5`, Typescript will infer the type and make it a number.  If you want to specify what type variable should be you use the `type` notation such as in the examples below.

```typescript
 let x: number
 let y: string
 let z: boolean
 let a: Date
 let b: string[]
```

If you actually want to be able to change variable types you can use the `any` type when defining the variable.

`let b: any`

You can also cast to any type.

`b = "Hello" as any`

Using `any` type should be avoided as it goes against the whole point of using Typescript

## Typing Functions

```typescript
function clone(source: Contact): Contact{
    return Object.apply({}, source);
}
```

The argument here is typed to Contact and the return value is as well.  Note that you can leave off the return type and Typescript will figure it out, but you may not get what you expect.  In this example the apply function would return the any type, so if you don't specify the return type it will be any in this case.

You can also have a function passed as a variable as in example below...

```typescript
function clone(source: Contact, func: (source:Contact) => Contact): Contact{
    return Object.apply({}, source);
}
```

If you want to define a method on an interface you would...

```typescript
interface Contact extends Address{
    id: number;
    name: ContactName;
    clone(name: string): Contact
}
```

## Defining a metatype using generics

A generic type is a metatype (a type that represents any other type you may want to substitute in)

For example we will use our clone function again.

```typescript
function clone<T>(source: T): T{
    return Object.appply({}, source);
}
```

T here is just a convention.  You can use any valid type name.  What we are saying here is whatever type gets passed in, is the type that will be returned.  This is what typescript will enforce.

You can use multiple generic type parameters.

```typescript
function clone<T1, T2>(source: T1): T2 {
    return Object.appply({}, source);
}
```

Since in the example above Typescript cannot infer what the types are we need to specify when calling the function as in the example below...

```typescript
const b = clone<Contact, Contact>(a)
```

Generic constraints allow you to put more restrictive rules on generic type parameters.  For example if you wanted to make sure that your return type is same as the input type you would...

```typescript
function clone<T1, T2 extends T1>(source: T1): T2 {
    return Object.apply({}, source);
}
```

In the above example whatever type you return (T2) must have the same fields as the input type (T1).  It does not have to extend the original type just have same properties and add to it whatever other properties it wants.

If you are trying to add a constraint on something that is not defined you can inline the constraint as in ...

```typescript
function getNextId<T extends { id: number}>(items: T[]): number{
    return items.reduce((max, x) => x.id > max ? max : x.id, 0) + 1
}
```

In this example whatever you pass in to the function needs to be an array of object that must have an id attribute.

Generics are not limited to functions they can be used in interfaces and classes as well.

```typescript
interface UserContact<TExternalId>{
    id: number,
    name: string,
    username: string,
    externalId: TExternalId,
    loadExternalId(): Task<TexternalId>
}
```

## Create custom types with interfaces

The syntax for defining an interface is the keyword interface followed by the custom type name and the properties of the type as in the example below...

```typescript
interface Contact{
    id: number;
    name: string;
    birthDate: Date;
}
```

Interface are only use for Typescript type checking and are never included in your runtime code.

To declare a variable as a custom type you would use the following...

`let primaryContact: Contact;`

If a field is optional you can append a question to the fields name as in example below.

```typescript
interface Contact{
    id: number;
    name: string;
    birthDate?: Date;
}
```

This means that you can omit the `birhtDate`` field, but if you include it; it must be a Date type.

You can also you an interface inside an interface using the `extends` keyword as in the example below...

```typescript
interface Address{
    line1: string;
    line2: string;
    province: string;
    region: string;
    postalCode: string;
}

interface Contact extends Address{
    id: number;
    name: string;
    birthDate?: Date;
}

let primaryContact = {
    birthDate: new Date("01-01-1980"),
    id: 12345,
    name: "Jamie Johnson",
    postalCode: "94044"
}
```

## Defining enumerable types

Enums are a type that has a hard coded list of values nad is defined like this...

```typescript
enum ContactStatus{
    Active,
    Inactive,
    New
}

interface Contact{
    id: number;
    name: string;
    birthDate?: Date;
    status: ContactStatus;
}

let primaryContact = {
    birthDate: new Date("01-01-1980"),
    id: 12345,
    name: "Jamie Johnson",
    status: ContactStatus.Active
}
```

Enums do make it into your runtime code.

By default the Enum value will evaluate to a number (based on position in enum) if you want them to evaluate to a string value you would do the following...

```typescript
enum ContactStatus{
    Active = "active",
    Inactive = inactive",
    New = new
}
```

You can use any values you like as long as they are all of the same type.