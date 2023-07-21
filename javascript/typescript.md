---
layout: page
title: "Typescript Reference"
permalink: /javascript/typescript
---

[comment]: <> (TODO: Need to break this down into easier to GROC sections)

# Intro

* Typescript is a compoiled language that compiles down to Javascript
* Typescript adds strong static typing on top of Javascript so that its able to provide compile time code checking.
* Typescript is a superset of Javascript
* For more infro and references got to typesriptlang.org
* [GitHub repo for ourse](https://github.com/LinkedInLearning/typescript-EssT-2428199)

## Installing Typescrit and adding it to your project

* In the course we are using node and then installing typescript you can install it locally or globally.  Node is just the package manager.

[NPM Download](https://nodejs.org/en)

[Typescript Download](https://www.typescriptlang.org/download)

Commands I used to get node installed

`curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - &&\
sudo apt-get install -y nodejs`

Command I used to install TypeScript into my project directory

`npm install typescript --save-dev`

**NOTE:** *You will need to mess with your PATH variable to get the tsc executable in the path.  The tsc executalbe is in the node modules directory*

You configure how typescript will behave with the `tsconfig.json` file in the root of your project directory.  There is an example in the sample project from the class.  

include is where you set the paths to serach that typescript should be transpiling

compilerOptions directive is for the compile options

outDir compiler option lets you specify where your transpiled output goes.

target is the target language version you want to transpile to.

    esnext is a target option that outputs to latest version of Javascript (goot to use if you will also use bable).  ES6 is another option as is ES3.

If you want to just use Typescript for type checking an noting else you can set noEmit to true in compile options.  This will cause it to not generate any output files.

## Adding type checking to Javascript Files  

In your tsconfig.js file set paramers (inside of compile options) `allowJs` and `checkJs` to true.

Then you can run tsc and on the .js files that are picked up type errors will be called out.  It will show where youa re trying to assing a type that is different than a previously assigned type.

You can add Type to Javascript (.js) files by using JS docs syntax.  You would add a comment block similar to the one below to tell what types certain variables are.  Its a docstring for the function being defined.  

```
/**
*
* @param {number} contactId
* @returns
*/
async function getContact(contactId){
    ...
}
```

## Importing third-party types

 If typescript does not have build in support for some library you are using you need to add the type defintion.  You can write it yourself or if its an open source library add the one they provide. 

 The type definitions live in the [DefinitelyTyped GitHub repo](https://github.com/definitelytypes/definitelytypes).  This can hard to navigate though so you want to go to [NPM repository](https://www.npmjs.com) and search for `@types package_name`` for example `@types jquery`

 Copy the install command you get from NPM repository and this will add the types definition to your node modules directory.

 Once you add it your editor (VS code) will pick it up and give you documentation and auto completion. 

 For your own libraries you would need to create your own .d.ts files to define the types.  That is a seperete course in end of itself.  You can copy and modify the public ones as a starting point.

 # Basic Typescript Usage

 ## Primatives and built-in types

 If you just assign a variable for exmaple `let x = 5`, Typescirpt will infer the type and make it a number.  If ou want to specify what type variable should be you use the :type notation such as in the examples below.

 `let x: number`
 `let y: string`
 `let z: boolean`
 `let a: Date`
 `let b: string[]` 

If you actually want to be able to change variable types you can use the `any` type when defining the variable.

`let b: any`

You can also cast to any type.

`b = "Hello" as any`

Using any type kind of goes against using Typescript in the first place.

## Creating custom types with interfaces

The syntax for defining an interface is the keyword interface followed by the custom type name and the properties of the type as in the examle below...

```
interface Contact{
    id: number;
    name: string;
    birthDate: Date;
}
```

Interface are only use for Typescript type checking and are never included in your runtime code.

To declare a variable as a custom type you would use the following...

`let primaryContact: Contact;`

If a field is optional you can appen a question to the fields name as in example below.

```
interface Contact{
    id: number;
    name: string;
    birthDate?: Date;
}
```

This means that you can omit the birhtDate field, but if you include it; it must be a Date type.

You can also you an interface inside an interface using the `extends` keyword as in the example below...

```
interface Contact extends Address{
    id: number;
    name: string;
    birthDate?: Date;
}

interface Address{
    line1: string;
    line2: string;
    province: string;
    region: string;
    postalCode: string;
}

let primaryContact = {
    birthDate: new Date("01-01-1980"),
    id: 12345,
    name: "Jamie Johnson",
    postalCode: "94044"
}
```

## Defining types using type aliases

Type alias just provides an alternate name to an elready existing type.  You declare the alias with the `type` keyword as in the example below..

```
type ContactName = string;

interface Contact extends Address{
    id: number;
    name: ContactName;
    birthDate?: Date;
}
```

Type aliases can be used interchangebly with the original type.

You can use them to give a little more meaning to the field you are assigning.  For example "its not just a string value, its a contact name value".

It also allows you to change all the types in a single location rather then multipl locations in yoru code.

## Defining enumerable types

Enums are a type that has a hard coded list of values and is defined like this...

```
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

enums do make it into your runtime code.  

By defulat the Enum value will evaluate to a number (based on position in enum) if you want them to evaluate to a string value you would do the following...

```
enum ContactStatus{
    Active = "active",
    Inactive = inactive",
    New = new
}
```

You can use any values you like as long as they are all of the same type.

## Typing Functions

```
function clone(source: Contact): Contact{
    return Object.apply({}, source);
}
```

The argument here is type to Contact and the return value is as well.  Note that you can leave off the return type and Typescript will figure it out, but you may not get what you expect.  In this example the apply function woudl rerurn the any type, so if you don't specify the return type it will be any in this case.

You can also have a function passed as a variable as in example below...

```
function clone(source: Contact, func: (source:Contact) => Contact): Contact{
    return Object.apply({}, source);
}
```

If you want to define a method on an interface you would...

```
interface Contact extends Address{
    id: number;
    name: ContactName;
    clone(name: string): Contact
}
```

## Defining a metatype using generics

A generic type is a metatatype (a type that represents any other type you may want to substitue in)

For example we will use our clone function again.

```
function clone<T>(source: T): T{
    return Object.appply({}, source);
}
```

T here is just a convention.  You can use any valid type name.  What we are saying here is whatever type gets passed in, is the type that will be returned.  This is what typescript will enforce.

You can use multipel generic type parameters.

```
function clone<T1, T2>(source: T1): T2 {
    return Object.appply({}, source);
}
```

Since in the example above Typescript cannot infer what the types are we need to specify when calling the function as in the example below...

```
const b = clone<Contact, Contact>(a)
```

Generic contraints allow you to put more restictive rules on genric type parameters.  For example if you wanted to make sure that your return type is same as the input type you would...

```
function clone<T1, T2 extends T1>(source: T1): T2{
    return Object.apply({}, source);
}
```

If you are trying ot add a constraint on something that is not defined you can inline the constaid as in ...

```
function getNextId<T extends { id: number}>(items: T[]): number{
    return items.reduce((max, x) => x.id > max ? max : x.id, 0) + 1
}
```

In this example whatever you apss in to the function needs to be an array of object that must have an id attribute.

In the above example whatever type you return (T2) must have the same fields as the input type (T1).  It does not have to extne the origin type just have same properties and add it whatever other properties it wants.

Generics are not limited to functions they can be used in interfaces and classes as well.

```
interface UserContact<TExternalId>{
    id: number,
    name: string,
    username: string,
    externalId: TExternalId,
    loadExternalId(): Task<TexternalId>
}
```

# Defining More Complex Types

## Combining multiple types with union types

If you want a field to support multiple types you would do something like the below.  Where the birthDate field can be a Date a number or a string.  You can list as many types as you wish.

```
interface Contact {
    id: number;
    name: ContactName;
    birthDate?: Date | number | string;
    status?: ContactStatus
}
```

You can also make this code a bit cleaner using type aliases as in the example below...

```
type ContactBirthDate = Date | number | string

interface Contact {
    id: number;
    name: ContactName;
    birthDate?: ContactBirthDate;
    status?: ContactStatus
}
```

You can also use union types instead of enums for example instead of the below code...

```
enum ContactStatus{
    Active = "active",
    Inactive = inactive",
    New = new
}
```

You can use a uniion type as in...

```
type ContactStatus = "active" | "inactive" | "new"
```

Using this approach you no longer have to use the . notations you still write the sting but Typescript will enforce that the value is one of the allowed values.

You can also combine types.  In thte above example we use `|` sysmbol to pick one of the types.  You can use the `&` symbol to combine types.

```
type AddressableContact = Contact & Address
```

This will create a tpe that has the fields of Contact and Address combined.

## Keyof operator

```
type ContactFields = keyof Contact

const field: ContactFields = "birthDate"
```

The above line means that the values of a variable of type ContactField can only contains values that are keys of the Contact type.  If you add a field to Contact the allowed values for ContactFields will be updated.  This is not compiled into runtime.

A real world use of this is for example writing a function that takes an opbect an an attribute of that object as in the example below.

```
function getValue<T>(source: T, propertyName: keyof T){
    return source[propertyName]
}
```

You can pass in any object and Typescript will know to limit you to only keys of that object for the propertyName argument.

A cleaner version of this function is below.  Here you are interoducing a second generic type that can be used elsewhere in the function.

```
function getValue<T, U extends keyof T>(source: T, propertyName U){
    return source[propertyName]
}
```

## Typeof operator

Typeof operator exists in Javascript, but you can make it work with Tpescript.  

```
function toContact(nameOrContact: string | Contact): Contact {
    if (typeof nameOrContact === "object){
        return {
            id: nameOrContact.id,
            name: nameOrContact.name,
            status: nameOrContact.status
        }
    }
    else {
        return {
            id: 0,
            name: nameOrContact,
            status: "active"
        }
    }
}
```

In the code above we are using union operator to limit input to a string or Contact type.

You can also use the typeof operator to avoid declaring a type explicitly.  For exmaple...

```
const myVar = { min: 1, max: 200 }

function save(source: typeof myVar) {...}
```

With the above code Typescript during compilation will enforc that the type you pass in to function matches the type of myVar.

## Indexes access types

You can use an index into an object to define a type.  For example...

```
type Awesome = Contact["id]

interface ContactEvent {
    contactId: Contact["id]
}
```

fist line in exmample above is just showing how that is used on its own.  The second part of an example show how you can use this in an custom type (defined with interface keyword) to show the intention of what the contactId type should be.

Below is a more advance example...

```
function handleEvent<T extends keyof ContactEvents>(
    eventName: T, 
    handler: (evt: ContactEvents[T]) => void
    ){
        if (evnetName === "statusChanged"){
            handler({ contactId: 1, oldStatus: "active", newStatus: "inactive"})
        }
}
```

In the example above we have the type T and keys of that type being used as arguments to the function.  Just a neat way of using this syntax.

## Defining Dynamic but limited types with records

Record type lets yo udefine some structure and typing without having to detail every possible property.  If you find yourself wanting to use any type you should probably look into Record as a work aournd.

```
let x: Record<string, string> = { name: "Bruce Wayne" }
```

The above line syas that you can have an object whose names and propery are strings.

If you need to be more flexible you can have the names be strings, but the values be vaiours types as in the exampe below...

```
let x: Record<string, string | number | boolean | Function> = { name: "Bruce Wayne"}
x.number = 1234
x.active = true
x.log = () => console.log("awesome")
x = 1234 # this will be a violation.
```

As mentioned above you can also do this with the attribut names as in the example below...

function searchContacts(contacts: Contact[], query: Record<keyof Contact, Query>){
    ...
}

Here we are saing that the query objects can only have parameters which are keys of Contact and Typescript will check this for us.

# Extending and Extracing Metadata from Existing Types

## Extending and modifying existing types

You may have a scenario where you are trying to use a Type but not specify all of the attributes of the type as in the example below.

```
type ContactQuery = Record<keyof Contact, Query>

function searchContacts(contacts: Contact[], query: ContactQuery){
    return contacts.filer(contact => {
        for (const property of Object.keys(contact) as (keyof Contact)[]){
            // get the query object for this property
            const propertyQuery = query[property];
            // check to see if it matches
            if (propertyQuery && propertyQuery.matches(contact[property])) {
                return true
            }
        }

        return false;
    })
}

const filteredContacts = searchContacts(
    [/* contacts */],
    {
        id: { matches: (id) => id === 123},
        name: { matches: (name) => name === "Carol Weaver" },  ### Not valid Typescript unless you do the Partial type as in example below...
    }
);
```

In above the object being passed to searchContacts will throw Typescipt error asking for all the fields.  To only use partial field use the `Partial` utility type.

```
type ContactQuery = Partial<Record<keyof Contact, Query>>
```

Partial looks exactly like the type it wraps but all attributes are optional.

If you want the opposite of partial and want to restirct usage of certain properties (for example if you don't want to allow queries on the address properties of Contact class) use the `Omit` utility class.

```
type ContactQuery = Omit<
    Partial<
        Record<keyof Contact, Query>
    >, "address" | "status"
>
```

If you want to not omit but limit to only a list of properties you would use the `Pick` utility class.

```
type ContactQuery = 
    Partial<
        Pick<
            Record<keyof Contact, Query>,
            "id" | "name"
        >
    >
```

With this you will only be able to access "id" and "name" properties.

The last utiltiy type is the `Required` which turns all properties to required instead of optional.

```
type RequiredContactQuery = Required<ContactQuery>
```

## Extracting metadata from existing types

