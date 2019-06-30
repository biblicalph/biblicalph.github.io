---
layout: post
title: "Inheritance in Javascript"
author: "Kwabena Boadu"
categories: journal
tags: [javascript,es6,es2015,inheritance,prototype,protoypal,__proto__,[[Prototype]],class]
---

Everything in Javascript is an object; this includes functions, strings, numbers, booleans, arrays and objects. Read more about objects in Javascript [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects){:target="_blank"}
Every object has a private property (the `[[Prototype]]` property) which holds a link to another object called its **prototype**.
The prototype of an object has a prototype of its own and in this way, objects in Javascript are linked until an object is reached whose prototype is **null**. **null** has no prototype and thus, acts as the final link in the chain. This chain linking objects to one another is called the ***prototype chain*** and is the medium through which Javascript implements inheritance.

### How Prototype Chain Works
Consider the four related people below:
- Joe has a friend called James
- James is friends with Jane
- Jane is also friends with a banker by name Doe

Let's say you approach Joe for a loan of $2k. Unfortunately, Joe doesn't have it but rather than turn you away, Joe asks James for help. Like Joe, James doesn't have it and asks from Jane who further asks from Doe. Doe has the money and sends it to you receive through the reverse relation.

Request: `You` ask `Joe` who asks `James` who asks `Jane` who asks `Doe`

Response: `Doe` gives to `Jane` who gives to `James` who gives to `Joe` who gives to `You`

The illustration above is exactly how the prototype chain works; if you query an object's property, it checks if the property is on the object. If the object has the property, it returns it, else it recursively checks the prototype chain of the object and returns the property if found or undefined otherwise.

### Accessing/Modifying an Object's Prototype
You can access or modify the prototype of an object (the private `[[Prototype]]` property) by calling `Object.getPrototypeOf` or `Object.setPrototypeOf` methods respectively. 
Note: The `__proto__` getter/setter for getting or setting the prototype of an object is deprecated - [mozilla docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto){:target="_blank"}

### Difference Between \__proto__, [[Prototype]] and prototype
`__proto__` is an accessor/mutator (getter/setter) for the private
`[[Prototype]]` property. It is, however, deprecated; use `Object.getPrototypeOf` and `Object.setPrototypeOf` instead.

`[[Prototype]]` is the private property of an object which holds a reference to the object's prototype.

Function objects (objects created using the `function` keyword) have a `prototype` property which is an object that has several properties including the private `[[Prototype]]` property of the function. Other properties, such as `name`, `length` and `constructor` are present on the `prototype` of a function. Read more [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype){:target="_blank"}.
The snippet below shows an example of how to get the `[[Prototype]]` of an object and a function.
```js
const obj = {
  name: 'John'
};
const sayHello = function(message) {
  console.log(`Hello! ${message}`);
}
// Print the `[[Prototype]]` of obj
console.log(Object.getPrototypeOf(obj)); 
// Print the `[[Prototype]]` of the function sayHello 
console.log(Object.getPrototypeOf(sayHello.prototype)); 
```
Note: Non-function objects don't have a `prototype` property.

### Prototypal Inheritance Examples
#### Example 1: (Non-function Objects)
```js
const user = {
  name: 'John',
  yearOfBirth: 1992,
  getName: function() {
    return this.name;
  },
  setName: function(name) {
    this.name = name;
  },
  getAge: function() {
    return new Date().getFullYear() - this.yearOfBirth;
  }
};
// Create employee that extends user
const employee = {
  id: '122223',
  getId: function() {
    return this.id;
  },
  setId: function(id) {
    this.id = id;
  }
};
Object.setPrototypeOf(employee, user);
// Employee's prototype is user
console.log(`employee's prototype: `, Object.getPrototypeOf(employee));
// Prints true
console.log(`user is employee's prototype: `, Object.getPrototypeOf(employee) === user);
// Looks up getAge from user, employee's prototype
console.log('employee age: ', employee.getAge());
```

#### Example 2: (Function Objects) - No ES6 class
```js
const User = function({firstname, lastname, email, password, gender, yearOfBirth}) {
  this.firstname = firstname;
  this.lastname = lastname;
  this.email = email;
  this.password = password;
  this.gender = gender;
  this.yearOfBirth = yearOfBirth;
};
User.prototype.getAge = function() {
  if (this.yearOfBirth) {
    return new Date().getFullYear() - this.yearOfBirth;
  }
  return -1;
};
User.prototype.getFullName = function() {
  return `${this.firstname} ${this.lastname}`;
};

const Employee = function({firstname, lastname, email, password, gender, yearOfBirth, id}) {
  // Call parent constructor
  User.call(this, {firstname, lastname, email, password, gender, yearOfBirth});
  this.id = id;
};
// Set prototype of Employee to User's prototype (prototype linkage)
// NB: Using Object.create is important here - we get a new object and can override it's constructor property
Employee.prototype = Object.create(User.prototype, {
  constructor: {
    value: Employee.name,
    enumerable: false,    // shouldn't appear in 'for in' loop
    writable: true
  }
});
Employee.prototype.getDisplayInfo = function() {
  return `
    name: ${this.getFullName()}\n
    email: ${this.email}\n
    gender: ${this.gender}\n
    age: ${this.getAge()}\n
    id: ${this.id}
  `;
};

const employeeOne = new Employee({
  firstname: 'John',
  lastname: 'Doe',
  email: 'johndoe@email.com',
  password: 'secret',
  gender: 'Male',
  yearOfBirth: 1992,
  id: 'emp-12345'
});
console.log(Object.getPrototypeOf(employeeOne));
console.log(employeeOne.getDisplayInfo());
```
Note: The above can be written using es6 classes as illustrated below:
```js
class User {
  constructor({ firstname, lastname, email, password, gender, yearOfBirth }) {
    this.firstname = firstname;
    this.lastname = lastname;
    this.email = email;
    this.password = password;
    this.gender = gender;
    this.yearOfBirth = yearOfBirth;
  }

  getAge() {
    if (this.yearOfBirth) {
      return new Date().getFullYear() - this.yearOfBirth;
    }
    return -1;
  }

  getFullName() {
    return `${this.firstname} ${this.lastname}`
  }
}

class Employee extends User {
  constructor({ firstname, lastname, email, password, gender, yearOfBirth, id }) {
    super({ firstname, lastname, email, password, gender, yearOfBirth });
    this.id = id;
  }

  getDisplayInfo() {
    return `
      name: ${this.getFullName()}\n
      email: ${this.email}\n
      gender: ${this.gender}\n
      age: ${this.getAge()}\n
      id: ${this.id}
    `;
  }
}

const employeeOne = new Employee({
  firstname: 'John',
  lastname: 'Doe',
  email: 'johndoe@email.com',
  password: 'secret',
  gender: 'Male',
  yearOfBirth: 1992,
  id: 'emp-12345'
});
console.log(employeeOne.getDisplayInfo());
```

### Prototypal (Prototype Based) Versus Classical (Class Based) Inheritance
- Class based languages implement inheritance by extending classes, the blue print (or template) from which objects are created, while prototype based inheritance is achieved through objects that hold a reference to another object. There is no concept of a `class`. Infact, the `class` keyword, introduced in es2015 (es6) and beyond, is just syntactical sugar for prototype based inheritance. 
- In classical inheritance, the parent of an object cannot be changed at runtime. The prototype of an object can, however, be changed at runtime using the `Object.setPrototypeOf` mutator. This makes prototypal inheritance extremely flexible

### Summary
In this article we discussed:
- what prototypal inheritance is
- the difference between `__proto__`, `[[Prototype]]` and `prototype` (of function objects)
- examples of prototypal inheritance
- how prototypal inheritance differs from classical inheritance.

Until next time, keep rising. Cheers.

### Additional References
- [Master the Javascript Interview](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9){:target="_blank"}, Eric Elliot
- [Javascript Prototype](https://www.javascripttutorial.net/javascript-prototype/){:target="_blank"}
- [Exploring Javascript](https://www.youtube.com/watch?v=aIVKX5SeLoE&t=758s){:target="_blank"}, Venkat Subramaniam
- [Javascript. The Core](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/){:target="_blank"}