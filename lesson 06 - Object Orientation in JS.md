# Lesson 6 - Objected Orientation / JavaScript in the Browser

## Recap & Intro
- So far we've covered JS fundamentals, this is the last topic in that series
- Today we're going to be talking about Object Oriented programming and Inheritance in JS.


## Object Orientation
Object-oriented programming (OOP) is a programming model organized around objects rather than "actions". That is, data rather than logic. While JS is more suited to functional programming, it does have some OO features that we need to understand.

Some key concepts in OO:

- Class - A template for a new object
- Instance - A single object created from a class
- Inheritance - An object can inherit properties from a parent object
- Encapsulation - Enclosing an object's data and functionality inside of it. (like the slideshow)

JS has some OO features.

### Classes
In JS, you create a class by simply defining a constructor. A constructor is a function used to create a new object instance. Like an object factory.

```javascript
function Teacher(name) {
	this.name = name;
	this.teach = function() {
		console.log(this.name + " says constructors are cool");
	}
}

var teacher1 = new Teacher('Shane');
var teacher2 = new Teacher('Assaf');

teacher1.teach();
teacher2.teach();
```

- The `new` keyword calls the function and returns the resulting object.
- Attributes are assigned to the object using `this` keyword
- Use this pattern when you need to create an object repeatedly.
- Note we're passing arguments into the constructor to customize our object

## Constructor Exercise

Write a constructor that creates a `Particle` object.

- The constructor takes `startX` and `startY` parameters
- Particle has a property `x` that defaults to the value of the argument `startX`
- Particle has a property `y`that defaults to the value of the argument `startY`
- Particle has a property `velocity` that defaults to {x: 0, y: 0}
- Create an array `particles`
- Write loop to create 100 particles
	- x values will be assigned from 0-99
	- y will be random from 0 to 500 (`Math.random()*500`)

## Constructor Exercise Answer
```javascript
function Particle(startX,startY) {
	this.x = startX;
	this.y = startY;
}

var particles = [];
for(var i = 0; i < 10; i++) {
	particles.push(new Particle(i,Math.random()*500));
}

```

## Inheritance (Classical vs Prototypal)

- Java and C use Classical Inheritance
	- Classical Inheritance models the world in terms of classes
	- Think of classes as templates
	- E.g. A `Teacher` class that inherits a `Person` class.

- JavaScript uses Prototypal inheritance
	- JavaScript inheritance is INSTANCE BASED.
	- JavaScript achieves inheritance using a "prototype" object.

### Simple example
In the following example, shane inherits properties from the Teacher prototype object.

```javascript
function Teacher(name) {
	this.name = name;
}

Teacher.prototype = {
	teach: function(){
		console.log('Prototypes are important, says ' + this.name);
	}
}

var shane = new Teacher('shane');
shane.teach();
```

How is this different than just using constructors?
- A prototype object is a live instance
- The prototype object is shared across all instances.

```javascript
function Teacher() {}

Teacher.prototype = {
	disposition: 'evil'
}

var shane = new Teacher();
var assaf = new Teacher();
console.log(shane.disposition, assaf.disposition) //evil, evil

Teacher.prototype.disposition = 'happy';
console.log(shane.disposition, assaf.disposition) //happy, happy
```

### Constructors vs Prototype
These two techniques seem pretty similar. When would you use one over the other?
- **Constructors**
	- Once an instance is created, it no longer has no connection to the class or other instances.
- **Prototype**
 	- Gives you the power to control many instances after creation
	- Is more memory efficient because it removes duplication

### Prototype Chain
What happens if an instance has a property with the same name as a prototype property?
```javascript
function Teacher() {}

Teacher.prototype = {
	name: 'John Doe'
}

var shane = new Teacher();
var assaf = new Teacher();

shane.name = "Shane";

console.log(shane.name, assaf.name) //'Shane', 'John Doe'
```

If `name` is a prototype property, how come changing it didn't change both instances' names?
- This works just like scope chains
- A property name is resolved by looking at the instance first. If the instance doesn't have a property with that name, then JS looks at the instance's prototype.
- Defining a property on an instance just covers over the prototype's property. It doesn't replace it.
- Think of instance properties as being "layered" on top of the prototype

Prototype objects can have their own prototypes, resulting in a prototype chain. To resolve a property, the interpreter will look at the instance, then it's prototype, then its prototype's prototype, and so on.

Just for argument's sake:

```javascript
function Person(){
	this.brain = true;
};
function Student(){
	this.computer = true;
};
function Graduate(){
	this.skillz = 'Mad'
};

var p = new Person();
Student.prototype = p;

var s = new Student();
Graduate.prototype = s;

var g = new Graduate();

console.log(g.skillz, g.computer, g.brain);

```

### hasOwnProperty
Objects have a method called 'hasOwnProperty' that tells you if a method is defined on an instance itself or one of its parent prototypes.

```javascript
//Using example above
console.log(g.hasOwnProperty('skillz'),g.hasOwnProperty('computer')) //true,false

```
* Note - `Object.create()` is an easy way to instantiate an object based on a prototype.

Prototype chains are important to understand, but try not to create complex class hierarchies in your JS code.

## Exercise 2: Prototypes

- Create a `time` variable and set it to 0
- Create a `gravity` variable and set it to 0.5
- Extend the particle class from earlier with a prototype object.
- Create a prototype object that contains:
	- A property `getVelocity()` that returns the value (time * gravity)
	- A property `move()` that adds the value from `getVelocity()` to the `y` position 
		- When `y >= 500` have it `console.log('bottom')`  
- Have all particles inherit the new prototype object.
- Use `setInterval` at 100ms to
	- increment `time` by 1
	- call the `move()` function on every particle with that has a y value < 500

## Exercise 2: Answer
```javascript
var gravity = 0.5;
var time = 0;

function Particle(startX,startY) {
	this.x = startX;
	this.y = startY;
}

Particle.prototype = {
	getVelocity: function(){
		return time * gravity;
	},

	move: function() {

	  this.y += this.getVelocity();
	  if(this.y >= 500)
		  console.log('bottom');
	}
}

var particles = [];
for(var i = 0; i < 10; i++) {
	particles.push(new Particle(i,Math.random()*500));
}

setInterval(function(){
	time++;
	particles.filter(function(p){
		return p.y < 500;
	})
	.forEach(function(p){
		p.move();
	})
},100)

```


## Object Composition
Usually, object composition is a better pattern than inheritance. This is also called the 'mixin pattern'.

Use `Object.assign(target, [...sources])` to copy properties from one or more objects into another without connecting them

### Contrived example

```javascript
var lion = {
	roar: function(){console.log('roar')}
}

var goat = {
	kick: function(){console.log('kick')}
}

var lizard = {
	tail: true
}

var chimera = {}
Object.assign(chimera, lion, goat, lizard);

chimera.roar();
chimera.kick();
chimera.tail;
```

### Real world example
Let's say you're keeping track of multiple configuration settings for an application.

```javascript
var baseConfig = {
	appName: 'Slick',
	apiKey: 'secretPassword',
	apiBaseUrl: 'http://slickapp.co/api/'
}

var localConfig = Object.assign({}, baseConfig, {
	apiKey: 'localPassword',
	apiBaseUrl: 'http://localhost:3000/api'
});

console.log(localConfig.name, localConfig.apiBaseUrl);
```

Let's say you're getting updated user settings from an API, but you don't want to send the whole settings object every time regardless of what's changed.


## Exercise 3: Mixins
- Create a user profile object that contains properties for
	- name
	- address
	- city
	- state
	- zipcode
	- avatar
- Write a function `getProfileUpdate()` that hypothetically asks the user to update some profile properties and returns information about which properties were changed, and to what values.(for now, just hard code the return value)
- Write a function `updateProfile()` that takes a single object of keys:values and overwrites those keys on the profile object
- Get a profile update and update the profile with it.
- log the new profile


## Homework
- Complete Exercise #3 and post it to GitHub
- Complete the [Prototype](https://github.com/sporto/planetproto) Nodeschool Module
- Read Book #3 from You Don't Know JS [this and Object Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20&%20object%20prototypes/README.md#you-dont-know-js-this--object-prototypes)
  - At the end of each chapter, send Shane a 1 sentence summary (in slack)
- [http://javascriptissexy.com/oop-in-javascript-what-you-need-to-know/](http://javascriptissexy.com/oop-in-javascript-what-you-need-to-know/)

