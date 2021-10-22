## Table of Contents
- [Prototypes](#prototypes)
  - [The `prototype` Property](#the-prototype-property)
  - [The `__proto__` Property](#the-proto-property)
  - [The Internal `[[Prototype]]` Property](#the-internal-prototype-property)
  - [The Prototype Chain](#the-prototype-chain)
- [Constructing Objects](#constructing-objects)
  - [Using `new`](#using-new)
  - [`Object.create()`](#objectcreate)
  - [The `constructor` Property](#the-constructor-property)

<sup><sub>[▲ TOP](#table-of-contents)</sub></sup>

## Prototypes
Unlike more conventional OOP languages, JavaScript uses special **prototype** objects to provide inheritance. An object's prototype acts as a template object that it inherits methods and properties from.

An object's prototype object may itself have a prototype object that it inherits from, and so on. This is often referred to as the **prototype chain**.

A link is made between an object instance and its prototype via the object's `__proto__` property, which is derived from the `prototype` property on the object's constructor.

Note that there is a conceptual distinction between an object's prototype (available via `Object.getPrototypeOf(obj)`, or via the deprecated `__proto__` property) and the `prototype` property on constructor functions. The `prototype` property on a constructor function is the blueprint for instances of this constructor function.

If we were to create an instance of an object with a constructor function &mdash; `let fooInstance = new Foo()` &mdash; then `fooInstance` would take its prototype from its constructor function's `prototype` property. In other words, `Object.getPrototypeOf(fooInstance) === Foo.prototype`.

### The `prototype` Property
The `prototype` property is an object that is basically a bucket for storing properties and methods that we want to be inherited by objects further down the prototype chain. **Only properties and methods defined on the `prototype` property are inherited.**

You can modify a constructor function's `prototype` property to add (or `delete`) any properties or methods you want available to all object instances created from the constructor.

Defining properties on the `prototype` property is pretty rare because they are not very flexible. You can't, for example, reference `this` on such properties since `this` will not be referencing proper scope. So you might define constant properties on the prototype, but it generally works better to define properties inside the constructor.

A fairly common pattern is to define properties inside the constructor, and the methods on the prototype. This makes code easy to read. For example:

```js
// Constructor with property definitions
function Test(a, b, c, d) {
  // property definitions
}

// First method definition
Test.prototype.x = function() { ... };

// Second method definition
Test.prototype.y = function() { ... };
```

### The `__proto__` Property
The `__proto__` property of `Object.prototype` is an accessor property (a getter function and setter function) that exposes the internal `[[Prototype]]` of the object. For objects created with an object literal, this is `Object.prototype`. For objects created using Array literals, this is `Array.prototype`. For functions, this is `Function.prototype`.

The `__proto__` setter allows the `[[Prototype]]` of an object to be mutated. The object must be extensible according to `Object.isExtensible()`.

**Using `__proto__` is deprecated and highly discouraged**, in favor of `Object.getPrototypeOf` / `Object.setPrototypeOf`. Though still, setting the `[[Prototype]]` of an object is a slow operation that should be avoided if performance is a concern.

### The Internal `[[Prototype]]` Property
Before ECMAScript 2015, there wasn't officially a way to access an object's prototype directly &mdash; the "links" between the items in the chain are defined in an internal property, referred to as `[[Prototype]]` in the specification for the JS language.

Most modern browsers, however, do offer the `__proto__` property which contain's the object's constructor's `prototype` object. Since ECMAScript 2015, you can access an object's prototype object indirectly via `Object.getPrototypeOf(obj)`.

### The Prototype Chain
![Prototype Inheritance](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes/mdn-graphics-person-person-object-2.png)

When accessing a property or method on an object, we search for the desired property or method by traversing up the prototype chain, starting with the object itself. For example, what happens if we call a method on `person1`, that is actually defined in `Object.prototype`? For example:

```js
person1.valueOf()
```

`valueOf` returns the value of the object it is called on. What happens in this case is:
1. The browser checks to see if `person1` has a `valueOf()` method available on it, as defined on its constructor `Person()`, and it doesn't.
2. The browser checks to see if `person1`'s prototype object (`Person.prototype`) has a `valueOf()` method available on it and it doesn't.
3. The browser checks to see if `person1`'s prototype object's prototype object (`Object.prototype`) has a `valueOf()` method, and it does. So the method is called!

Note that methods and properties are **not** copied from one object to another in the prototype chain. They are *accessed by walking up the chain*.

Note that the prototype chain is traversed only when *retrieving* properties. If properties are set or `delete`d directly on the object, the prototype chain is not traversed.

<sup><sub>[▲ TOP](#table-of-contents)</sub></sup>

## Constructing Objects
In JavaScript, you can create an instance of a user-defined object type, or of one of the built-in object types that has a constructor function.

```js
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
}

const car1 = new Car('Eagle', 'Talon TSi', 1993);

console.log(car1.make); // "Eagle"
```

### Using `new`
The `new` operator does the following things:
1. Creates a blank, plain JS object
2. Adds a property to the new object (`__proto__`) that links to the constructor function's prototype object (see [protoypes](#prototypes))
3. Binds the newly created object instance as the `this` context (i.e. all references to `this` in the constructor function now refer to the object created in the first step)
4. Returns `this` if the function doesn't return an object. (Most constructors don't return a value, but they can choose to do so if they want to override the normal object creation process.)

Note that while the constructor function can be invoked like any regular function (without the `new` operator), in that case a new Object is not created and the value of `this` will also be different.

Note that not all behavior for a constructor function must be defined within the function itself. You can add a shared property to a previously defined object type by using the constructor function's `prototype` property. This defines a property shared by all objects created with that function.

The following code adds a `color` property with the value `"original color"` to all objects of type `Car`, and then overwrites that value with the string `"black"` only in the instance object `car1`.

```js
function Car() {}
car1 = new Car();
car2 = new Car();

console.log(car1.color);    // undefined

Car.prototype.color = 'original color';
console.log(car1.color);    // 'original color'

car1.color = 'black';
console.log(car1.color);    // 'black'

console.log(Object.getPrototypeOf(car1).color); // 'original color'
console.log(Object.getPrototypeOf(car2).color); // 'original color'
console.log(car1.color);   // 'black'
console.log(car2.color);   // 'original color'
```

See [prototypes](#prototypes) for more information

### `Object.create()`
The `Object.create()` method creates a new object, using an existing object as the prototype of the newly created object.

```js
const person = {
  isHuman: false,
  printIntroduction: function() {
    console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`);
  }
};

const me = Object.create(person);

me.name = 'Matthew'; // "name" is a property set on "me", but not on "person"
me.isHuman = true; // inherited properties can be overwritten

me.printIntroduction();
// expected output: "My name is Matthew. Am I human? true"
```

### The `constructor` Property
Every constructor function's `prototype` property itself contains a `constructor` property. The `constructor` property points to the original constructor function. 

Since it is defined on the constructor function's `prototype` property, the `constructor` property is available to all instance objects created from the constructor function. This means that you can call an object's constructor function from the object instance itself. You won't need this often, but it can be useful when you want to create a new instance and don't have a reference to the original constructor easily available for some reason.

<sup><sub>[▲ TOP](#table-of-contents)</sub></sup>
