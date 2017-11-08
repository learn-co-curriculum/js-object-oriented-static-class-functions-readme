# Static Class Functions with Classes

## Objectives

1. The Difference between Instance Functions and Static Functions
2. Defining a Class Function
3. Delcaring a Class Variable
4. Using Class Variables in an Instance Function

### Class Functions

We learned that in the ES2015 Class Syntax we can create Javascript classes that define functions for the instances of those classes.

```js
class User {
  constructor(name){
    this.name = name;
  }

  sayHello() {
    console.log(`Hello, my name is ${this.name}`);
  }
}

var sarah = new User('sarah');
sarah.sayHello(); // Hello, my name is Sarah
```

When desigining classes, we think a lot about responsibility of behavior and data. What if we wanted to be able to return all instances of a `User` that have been created? Before even thinking of how we might implement that, the first question is, what object is responsible for that? 

In the example above, would it be `sarah`'s, an instance of the `User` class, responsibility to return to our system all the other instances of the `User` class? No. `sarah`, an instance, is solely responsible for behavoir related to her. 

It should the `User` classes responsibility. The desired code might look like `User.All()`, a function attached directly to the `User` class the returns all instances of a `User`.

In Object Orientation in Javascript we call functions that operate on the `class` itself `static` functions (or Class Functions). Let's implement a stubbed out version of this `User.All()`.

```js
class User {

  // The static keyword defines the Class Function
  static All() {
    return ["User 1", "User 2"]
  }

}

User.All() //=> ["User 1", "User 2"]
```

By prefacing a function declartion within the class body with the `static` keyword, we are declaring that function to operate on the class level, called directly on the class as a function, `User.All()`.

Once defined, `static` class functions operate the same as any other function, with the important note that `this` within a `static` class function will refer to the class itself, and not an instance.

```js
class User {

  static All() {
    console.log(this)
    return ["User 1", "User 2"]
  }

}
User.All() //=> ["User 1", "User 2"]
// [Function: User]
```

### Delcaring a Class Variable

For the desired functionality of returning all instances of a `User` that have been created through the `User.All()` class function, we need a place to store those instances.

Again, responsibility is everything in our design choice, the `User` class itself should be responsible for storing that data. In the same manner that instances of a class can have properties and data, a class can as well.

Declaring a class variable is syntactically a little different you'd expect and happens outside of the class definition.

```js
class User {
  static All(){
    return this._All;
  }
}

// Attach a property directly to the class
// initializing it to an empty array.
User._All = [];
```

Below the class definition, we can declare a new property on the class itself (much in the same manner that we can add a property to any object in Javascript).

The name of the property declared, `_All`, prefaced with the `_` is simply a convention to accomplish 2 goals:

1. Disambiguating between the property `_All` and the class function `All()`.
2. To convey to other programmers that this property is intended to be private and should not be accessed directly but rather always through the class function `All()`. There are other mechanics to accomplish this desire, but the prefix `_` convention is common.

Finally, within the `static All()` function definition, we can use the scope of `this`, the `User` class itself, to access the class variable `_All` and return it. The `static All()` function becomes a class getter whose only responsibility is to return the class property of `_All`.

### Using Class Variables in an Instance Function

The final step in our desired functionality of returning all instances of a `User` that have ever been created is to understand how to access the class property `_All` within an instance function, specfically, the `constructor`, which is the moment in time when we know an instance has been created and have access to the instance in the form of `this`.

```js
class User {
  constructor(){
    // `this` refers to the currently created instance.

    // push `this` into the array by explictly
    // refrencing the User._All property.
    User._All.push(this)
  }

  static All() {
    return this._All;
  }

}
User._All = []

let sarah = new User()
let bob = new User()

User.All() // returns the 2 instances created above
//=> [ User {}, User {} ]
```

This implementation works. Within the constructor, we `push` the new instance by referencing `this` directly and explicitly into the property of `User._All`. However, we can make it more elegant.

From an instance, we can access the constructor, the class of the instance, via the `constructor` property.

```js
class User {
  constructor(){
    console.log(this.constructor)
  }
}

new User() // [Function: User]
```

This more abstract reference to the class is prefered. In an instance function, to access a class property, `_All`, we could write `this.constructor._All`. The final implementation of the pattern would be:

```js
class User {
  constructor(){
    this.constructor._All.push(this)
  }

  static All() {
    return this._All;
  }

}
User._All = []

let sarah = new User()
let bob = new User()

User.All()
//=> [ User {}, User {} ]
```

This is a tremendously powerful pattern. We could clean it up further by building an explicit instance function to `save` the new instance of `User` and further encapsulate the access to the constructor and the class variable `_All`.

```js
class User {
  constructor(){
    this.save()
  }

  save(){
    this.constructor._All.push(this)
  }

  static All() {
    return this._All;
  }

}
User._All = []

let sarah = new User()
let bob = new User()

User.All()
//=> [ User {}, User {} ]
```

### A Stretch: Building a Finder

Consider the power of the following example that combines everything covered so far to implement being able to find an instance of `User` by their email.

```js
class User {
  constructor(name, email){
    this.name = name
    this.email = email
    this.save()
  }

  save(){
    this.constructor._All.push(this)
  }

  static All() {
    return this._All;
  }

  static FindByEmail(email){
    return this.All().filter(function(user){
      return user.email === email;
    })
  }
}
User._All = []

let sarah = new User("Sarah", "sarah@flatironschool.com")
let bob = new User("Bob", "bob@flatironschool.com")

let matchingUser = User.FindByEmail("sarah@flatironschool.com")
//=> [ User { name: 'Sarah', email: 'sarah@flatironschool.com' } ]
```

## Resources

* [MDN static functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static)
* [Private Data Naming Conventions](http://exploringjs.com/es6/ch_classes.html#sec_private-data-for-classes#_private-data-via-a-naming-convention)
