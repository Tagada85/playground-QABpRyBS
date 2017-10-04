## Introduction

Aaaaah, prototypes... How many blog posts did you read where prototypes are listed as a must-know characteristic of the language? How many times senior developers told you about prototypal inheritance? I've spent quite some time avoiding to learn more deeply about this thing. I got tired of procrastinating, so I wrote this thing.

## Simple words please... with examples?

Object in Javascript have an internal property ( in the specification called [[Prototype]] ). This internal property is a reference to another object. Quick example:

```javascript runnable
// A simple object
const myObject = {
  a: 2
}
console.log(myObject.a) // 2

// We link newObject to myObject with Object.create
const newObject = Object.create(myObject)

console.log(newObject) // {}
console.log(newObject.a) // 2  (!!??)
```

*Object.create* creates a new object. It takes another object as an argument. The common way to think about what is happening is ( the *classical* way ) : I made a copy of this object. **You fool!!**

As you can see, *newObject* is empty. We didn't copy, we linked *newObject* to *myObject*. *myObject* becomes a prototype of *newObject*. To know what is inside the prototype of an object, you can use **__proto__**.

```javascript runnable
// A simple object
const myObject = {
  a: 2
}
console.log(myObject.a) // 2

// We link newObject to myObject with Object.create
const newObject = Object.create(myObject)

console.log(newObject.__proto__) // { a: 2 }
console.log(myObject.isPrototypeOf(newObject)) // true
```

Chains have links, [[Prototype]] is a chain. So how does Javascript uses prototypes to retrieve values?

## Up the chain... one link at a time.

```javascript runnable
const original = {
  a: 2
}

const secondComing = Object.create(original)

const thirdLink = Object.create(secondComing)

console.log(thirdLink) // {}
console.log(secondComing) // {}

console.log(secondComing.isPrototypeOf(thirdLink)) // true
console.log(original.isPrototypeOf(thirdLink)) // true
console.log(thirdLink.isPrototypeOf(original)) // false 

console.log(thirdLink.a) // 2

```

Here is how your favourite language works: it tries to get the property *a* in the *thirdLink* object. Can't find it. Does it return undefined or a error? Nope, it looks in the prototype chain for a link. It finds out that *secondComing* is a prototype of *thirdLink*. It looks for *a*, still can't find it. It moves on to another link, called *original*. Finds a = 2 !!

## What if I change something in the bottom of the chain?

- How will it affect the top of the chain? Such a great question.

I decide to change the value *a* in *thirdLink* directly:

```javascript runnable
const original = {
  a: 2
}

const secondComing = Object.create(original)

const thirdLink = Object.create(secondComing)
thirdLink.a = 3

console.log(thirdLink) // { a: 3 }
console.log(thirdLink.a) // 3
console.log(original.a) // 2
```

This is what we call a shadowed property. The new *a* value shadows the other *a* values present in the higher prototypes.

### What if I put some ice on it?

What if the property in the top link can't be overwritten?

```javascript runnable
const original = {
  a: 2
}

const secondComing = Object.create(original)

const thirdLink = Object.create(secondComing)

// Freeze the original, properties can't be changed
Object.freeze(original)
original.a = 3
// a is still equal to 2
console.log(original) // { a: 2 }

// That will NOT change the value, or shadow it.
thirdLink.a = 3
console.log(thirdLink) // {} 
console.log(thirdLink.a) // 2

```

Nothing changed because the prototype's property *a* is read-only.

However, if you need to change the property value anyway when it is read-only. You must use *Object.defineProperty*:

```javascript runnable
const original = {
  a: 2
}

const secondComing = Object.create(original)

const thirdLink = Object.create(secondComing)
// Freeze the original, properties can't be changed
Object.freeze(original)

// Ok, this will work.
Object.defineProperty(thirdLink, 'a', { value: 5 })

console.log(thirdLink.a) // 5
```
So, whenever you think you are changing a value in a object, you must account for the prototypes up the chain. They may have properties with the same name that can't be overwritten in a certain way.

## What does it mean for functions?

In a class oriented language, you can create different instances of a class. You copy the class behavior into an object. And this is done again every time you instantiate a class.

In Javascript, however, there are no classes, just objects. The **class** keyword is only a syntax thing, it doesn't bring anything class-y to the table. Whatever you can do with the **class** keyword in ES6, you could do without a problem in ES5.

By default, every function gets a *prototype* property.

```javascript runnable
function hello(){
  return 'Hello World'
}

function goodBye(){
  return 'Goodbye'
}

console.log(hello.prototype) // hello {}
console.log(goodBye.prototype) // goodBye {}
```

Ok, so what happens if you don't copy like class-oriented languages? You create multiple objects with a [[Prototype]] link. Like so:

```javascript runnable
function hello(){
  return 'Hello World'
}

function goodBye(){
  return 'Goodbye'
}

const a = new hello()
const b = new hello()
const c = new goodBye()
const d = new goodBye()

console.log(Object.getPrototypeOf(a) === hello.prototype) // true
console.log(Object.getPrototypeOf(b) === hello.prototype) // true
console.log(Object.getPrototypeOf(c) === goodBye.prototype) // true
console.log(Object.getPrototypeOf(d) === goodBye.prototype) // true
```

All our objects link to the same *hello.prototype* or *goodBye.prototype* origin. So, our objects ( a, b, c and d ) are not completely separated from one another, but linked to the same origin. So, if I add a method in *hello.prototype*, *a* and *b* will have access to it, because Javascript will go up the chain to find it. But, I didn't change anything about *a* and *b*:

```javascript runnable
function hello(){
  return 'Hello World'
}

function goodBye(){
  return 'Goodbye'
}

// I'm not touching a or b
hello.prototype.sayHello = () => {
  console.log('I say hello to you!')
}

const a = new hello()
const b = new hello()
const c = new goodBye()
const d = new goodBye()

a.sayHello() // I say hello to you!
b.sayHello() // I say hello to you!
```

By **NOT** copying objects but linking them, Javascript doesn't need to have the entire object environment carried in each object. It just goes up the chain.

```javascript runnable
function hello(){
  return 'Hello World'
}

function goodBye(){
  return 'Goodbye'
}

hello.prototype.sayHello = () => {
  console.log('I say hello to you!')
}

const a = new hello()
const b = new hello()
const c = new goodBye()
const d = new goodBye()

// Objects not linked yet => Errors
c.sayHello() // Error: not a function
d.sayHello() // Error: not a function
```

Now, let's now make the *goodBye.prototype* a prototype of *hello.prototype*:

```javascript runnable
function hello(){
  return 'Hello World'
}

function goodBye(){
  return 'Goodbye'
}

hello.prototype.sayHello = () => {
  console.log('I say hello to you!')
}

const a = new hello()
const b = new hello()
const c = new goodBye()
const d = new goodBye()

// This is a ES6 method. First argument will be the link at the bottom of the prototype chain, the second is the top link.
Object.setPrototypeOf(goodBye.prototype, hello.prototype)


// Now, c and d will look up the chain!
c.sayHello() // I say hello to you!
d.sayHello() // I say hello to you!
```

Let me show you a disgusting thing I made, maybe it will be clearer:

<img src="https://www.damiencosset.com/wp-content/uploads/2017/10/prototypesSchema-e1506980959792-300x207.jpg" alt="Prototypes schema" width="400" height="300" />

Beautiful... Notice how the arrows go from the bottom to the top!

## Prototypal inheritance

And that, my dear friends, is the concept of prototypal inheritance. Now, I am not a huge fan of the word *inheritance* here. It would imply some sort of copying, or parent-child relationship, and Javascript doesn't do that. I have seen the word *delegation* to describe this, I like it better. Again, Javascript doesn't natively copy objects, it links them to one another.

I see you waiting for some examples:

```javascript runnable
function Mammal(type){
  this.type = type
  this.talk = () => {
    console.log('Hello friend')
  }
}

Mammal.prototype.myType = function(){
  return this.type
}

function Dog(name, type){
  Mammal.call(this, type)
  this.name = name
  this.woof = () => {
    console.log('Woof!')
  }
}

// Link the Dog prototype to the Mammal prototype
Object.setPrototypeOf(Dog.prototype, Mammal.prototype)
//OR
// Dog.prototype = Object.create(Mammal.prototype)


Dog.prototype.myName = function(){
  return this.name
}

const Joe = new Dog('Joe', 'Labrador')

Joe.woof() // Woof!
console.log(Joe.myName()) //Joe
console.log(Joe.myType()) // Labrador
Joe.talk() // Hello friend

```

```javascript runnable

const SuperHero = {
  statement: function(){
    return 'I am an anonymous superhero'
  }
}

const Batman = Object.create(SuperHero)

console.log( Batman.statement() ) // 'I am an anonymous superhero'

```

## Conclusion

Classical inheritance is a parent-child relationship. It goes from top to bottom. Javascript has *prototypal delegation*. Although it *resembles* the classical inheritance, it's quite different. Objects are linked together, not copied. The references are more from bottom to top.

I hope I have been clear about the way prototypes and prototypal delegation work in Javascript.

Oh, and don't bother giving me feedback about the schema, I already know it's gorgeous.
