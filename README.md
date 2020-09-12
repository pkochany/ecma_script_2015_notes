#### Ecma Script 2015

1. Arrow functions:

   

```js
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| properly refers to the Person object
  }, 1000);
}

var p = new Person();
```

```javascript
var elements = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryllium'
];

// This statement returns the array: [8, 6, 7, 9]
elements.map(function(element) {
  return element.length;
});

// The regular function above can be written as the arrow function below
elements.map((element) => {
  return element.length;
}); // [8, 6, 7, 9]

// When there is only one parameter, we can remove the surrounding parentheses
elements.map(element => {
  return element.length;
}); // [8, 6, 7, 9]

// When the only statement in an arrow function is `return`, we can remove `return` and remove
// the surrounding curly brackets
elements.map(element => element.length); // [8, 6, 7, 9]

// In this case, because we only need the length property, we can use destructuring parameter:
// Notice that the `length` corresponds to the property we want to get whereas the
// obviously non-special `lengthFooBArX` is just the name of a variable which can be changed
// to any valid variable name you want
elements.map(({ length: lengthFooBArX }) => lengthFooBArX); // [8, 6, 7, 9]

// This destructuring parameter assignment can also be written as seen below. However, note that in
// this example we are not assigning `length` value to the made up property. Instead, the literal name
// itself of the variable `length` is used as the property we want to retrieve from the object.
elements.map(({ length }) => length); // [8, 6, 7, 9] 
```



#### Promises

Always in one of three states.

1. Pending - default
2. Fulfilled - completed succesfully
3. Rejected - failed

Create using Promise() constructor.

Define what happens when promise is fulfilled or rejected.

Consume the value of a fulfilled promise (then() on Promise object), or provide a rejection reason for a rejected promise (catch() on Promise object).

Using fulfilled promises:

```javascript
const breakfastPromise = new Promise( () => {
    setTimeout(() => {
        resolve('Your order is ready. Come and get it!');
    }, 3000);
});

console.log(breakfastPromise);
// then return fulfilled value val
breakfastPromise.then( val => console.log(val) );
```

Catching rejected promises:

```javascript
const breakfastPromise = new Promise( () => {
    setTimeout(() => {
        reject( Error('Oh no! There was a problem with your order.') );
    }, 3000);
});

console.log(breakfastPromise);
// then return fulfilled value
// catch return rejected value
breakfastPromise
    .then( val => console.log(val)
    .catch( err => console.log(err) ) );
```

It is possible to chain promises and pass the return value of one to input value of another:

```javascript
function addFive(n) {
  return n + 5;
}
function double(n) {
  return n * 2;
}
function finalValue(nextValue) {
  console.log(`The final value is ${nextValue}`);
}

const mathPromise = new Promise( (resolve, reject) => {
  setTimeout( () => {
    // resolve promise if 'value' is a number; otherwise, reject it
    if (typeof value === 'number') {
      resolve(value);
    } else {
      reject('You must specify a number as the value.')
    }
  }, 1000);
});

const value = 5;
mathPromise
  .then(addFive)
  .then(double)
  .then(finalValue)
  .catch( err => console.log(err) )

// The final value is 20
```

### Example 1:

```javascript
const astrosUrl = 'http://api.open-notify.org/astros.json';
const wikiUrl = 'https://en.wikipedia.org/api/rest_v1/page/summary/';
const peopleList = document.getElementById('people');
const btn = document.querySelector('button');

function getProfiles(json) {
    const profiles = json.people.map( person => {
        const craft = person.craft;
        // method fetch() gets data from endpoint in form of json
        // it is possible to access data related to request as well as data itself
        return fetch(wikiUrl + person.name)
        		.then( response => response.json() )
        		.then( profile => {
            		return { ...profile, craft }
        		})
        		.catch( err => console.log('Error fetching Wiki: ', err) )
    }); 
    // this is returning array or promises
	return profiles;
    // this is waiting for all promises to resolve
    // and returns a Promise containing as value an array of returned objects (data)
    // it allways follow the order in which the promises have been passed
    return Promise.all(profiles);
}

function generateHTML(data) {
  data.map( person => {
    const section = document.createElement('section');
    peopleList.appendChild(section);
    section.innerHTML = `
    <img src=${person.thumbnail.source}>
	<span>${person.craft}</span>
    <h2>${person.title}</h2>
    <p>${person.description}</p>
    <p>${person.extract}</p>
    `;
  });
}

btn.addEventListener('click', (event) => {
  event.target.textContent = "Loading...";
  fetch(astrosUrl)
    .then( response => response.json() )
    .then(getProfiles)
    .then(generateHtml)
    .catch( err => {
      peopleList.inngerHTML = '<h3>Something went wrong!</h3>';
      console.log(err)
  } )
    .finally( () => event.target.remove() )
});
```

### Async / await.

Await:

1. Pauses the execution of an async function and waits for the resolution of a promise.
2. Resumes execution and returns the resolved value.
3. Pausing execution is not going to cause blocking behavior.
4. Valid only inside functions marked async.

Async function always returns a promise.

That promise resolves with the value returned by the async function, or rejects with an error thrown from within the function.

```javascript
async function fetchData(url) {
    const response = await fetch(url);
    const data = await response.json();
    
    return data;
}

// to use data (Promise) returned by async funtion it needs to be passed to a then method
fetchData('dog.ceo/dog-api/breeds')
	.then( data => console.log(data) )
```

### Example:

```javascript
async function getPeopleInSpace() {
    const peopleResponse = await fetch(url);
    const peopleJSON = await peopleResponse.json();
    
    const profiles = peopleJSON.people.map( async (person) => {
        const craft = person.craft;
        const profileResponse = await fetch(wikiUrl + person.name);
        const profileJSON = await profileResponse.json();
        
        return { ...profileJSON, craft };
    })
    
    return Promise.all(profiles);
}

btn.addEventListener('click', (event) => {
    event.target.textContent = 'Loading...';
    
    getPeopleInSpace(astrosUrl)
    	.then(generateHTML)
    	.finally( () => event.target.remove() )
})
```

### Exceptions and error handling.

```javascript
// Handle all fetch requests
async function getJSON(url) {
    try {
        const response = await fetch(url);
        return await response.json();
    } catch (error) {
        throw error;
    }
}

async function getPeopleInSpace() {
    const peopleJSON = await getJSON(url);
    
    const profiles = peopleJSON.people.map( async (person) => {
        const craft = person.craft;
        const profileJSON = await getJSON(wikiUrl + person.name);
        
        return { ...profileJSON, craft };
    })
    
    return Promise.all(profiles);
}


btn.addEventListener('click', (event) => {
    event.target.textContent = 'Loading...';
    
    getPeopleInSpace(astrosUrl)
    	.then(generateHTML)
    	.catch( e => {
        	peopleList.innerHTML = '<h3>Something went wrong!</h3>';
        	console.error(e);
    	})
    	.finally( () => event.target.remove() )
})
```

## Web APIs

**The call stack handles asynchronous tasks in a different way than synchronous tasks**. Whenever the JavaScript environment encounters code that needs to run and execute at a later time, like a `setTimeout()` callback or a network request, that code is handed off to a special Web API that processes the async operation. Meanwhile, the call stack moves on to other tasks then revisits the async task when it's ready to be  dealt with.

It's important to note that **the asynchronous behavior of JavaScript does not come from the JavaScript engine itself**. JavaScript engines like Chrome's V8, for instance, do not have  asynchronous capabilities. Asynchronicity actually comes from the  runtime environment. Runtime environments *(like web browsers and Node)* provide APIs that let JavaScript run tasks asynchronously.

For example, async development in the browser (which is the focus of  this course) happens in a number of places using Web APIs like `setTimeout()`, XMLHttpRequest (XHR), Fetch API, and the DOM event API.

Where does the async callback go before being pushed onto the call  stack? It doesn't immediately go back into the call stack. Instead the  callback gets pushed into something called the **callback queue**.

## How the Callback Queue Works

You can think of the callback queue as a holding area for all the  callbacks that are waiting to be put on the call stack and eventually  executed. Only callbacks for (non-blocking) operations that are  scheduled to complete execution at a later time like `setTimeout`, `setInterval` and AJAX requests, for example, are added to the callback queue

The web APIs add each async task to the callback queue where they wait  to be pushed onto the call stack and executed one at a time. 

## The Event Loop

The event loop has one important job: monitor the call stack and  callback queue. Anytime the call stack is empty, the event loop takes  the first task from the callback queue and pushes it onto the call  stack, and runs it.

## Default Parameters

```javascript
function greet(name = 'Piotr', timeOfDay = 'Day') {
    console.log(`Good ${timeOfDay}, ${name}!`);
}

// we can pass undefined to function with defined default parameters
// and the function will use default value for this parameter
greet(undefined, 'Afternoon');
```

## Rest Parameters and Spread Operator

#### Rest Parameter

```javascript
// rest parameter is three dots ... before variable params
// variable name can be any name, it doesn't have to be params
// Rest Operator have to be assigned to the last function argument
function myFunction(name, ...params) {
    console.log(name, params);
}

// when calling function with rest parameter we can call it with more arguments
// in this example I call 2 argument function using 6 arguments
// rest parameter makes argument ...params contain an array of all arguments
// that have been passed after the first one
myFunction('Piotr', 1, 2, 3, 4, 'Dawid')
```

#### Spread Operator

```javascript
'use strict';

// spread operator enables arrays to be nested
// anything added to oldFlavors or newFlavors will be added to inventory as well
const oldFlavors = ['Chocolate', 'Vanilla'];

const newFlavors = ['Strawberry', 'Mint Chocolate Chip'];

const inventory = ['Rocky Road', ...oldFlavors, 'Neapolitan', ...newFlavors];

console.log(inventory);
```

```javascript
'use strict';

function myFunction(name, iceCreamFlavor) {
    console.log(`${name} really likes ${iceCreamFlavor} ice cream.`)
}

let args = ['Piotr', 'Chocolate'];

// it can also be used to get arguments for array and pass it to a function call
myFunction(...args)
```

## De-structuring

```javascript
'use strict';

let toybox = { item1: 'car', item2: 'ball', item3: 'frisbee'};

// de-structuring lets you take pieces of object and place them in a variable
let {item1, item2} = toybox;

console.log(item1, item2);

// it can also create new variables inside parentheses
let {item3: disc} = toybox;

console.log(disc);
```

```javascript
'use strict';

let widgets = ['widget1', 'widget2', 'widget3', 'widget4', 'widget5'];

let [a, b, c, ...d] = widgets;

console.log(d);
```

```javascript
function getData({ url, method = 'post' } = {}, callback) {
    callback(url, method);
}

getData({ url: 'myposturl.com' }, function (url, method) {
    console.log(url, method);
});

getData({ url: 'myputurl.com', method: 'put' }, function (url, method) {
    console.log(url, method);
});
```

```javascript
'use strict';

let parentObject = {
    title: 'Super Important',
    childObject: {
        title: 'Equally Important'
    }
}

let { title, childObject: { title: childTitle } } = parentObject

console.log(childTitle);
```

## Object Property Shorthand

```javascript
'use strict';

function submit(name, comments, rating = 5) {
	let data = { name, comments, rating };
	
	for (let key in data) {
		console.log(key + ':', data[key]);
	}
	// ... do ajax request
}

submit('English', 'Great course!', 9);
```

## For...Of and For...In loop

```javascript
for (let teacher of teachers) {
    console.log(teacher.name);
    if (teacher.name === 'Nick') {
        console.log(teacher.rating);
        break;
    }
}
```

```javascript
const object = { a: 1, b: 2, c: 3 };

for (const property in object) {
  console.log(`${property}: ${object[property]}`);
}
```

## Set

`Set` objects are collections of values. You can iterate through the elements of a set in insertion order. A value in the `Set` **may only occur once**; it is unique in the `Set`'s collection.

Because each value in the `Set` has to be unique, the value equality will be checked. In an earlier version of ECMAScript specification, this was not based on the same algorithm as the one used in the `===` operator. Specifically, for `Set`s, `+0` (which is strictly equal to `-0`) and `-0` were different values. However, this was changed in the ECMAScript 2015 specification. See *"Key equality for -0 and 0"* in the [browser compatibility](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set#Browser_compatibility) table for details.

[`NaN`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NaN) and [`undefined`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined) can also be stored in a Set. All `NaN` values are equated (i.e. `NaN` is considered the same as `NaN`, even though `NaN !== NaN`).

```javascript
let mySet = new Set()

mySet.add(1)           // Set [ 1 ]
mySet.add(5)           // Set [ 1, 5 ]
mySet.add(5)           // Set [ 1, 5 ]
mySet.add('some text') // Set [ 1, 5, 'some text' ]
let o = {a: 1, b: 2}
mySet.add(o)

mySet.add({a: 1, b: 2})   // o is referencing a different object, so this is okay

mySet.has(1)              // true
mySet.has(3)              // false, since 3 has not been added to the set
mySet.has(5)              // true
mySet.has(Math.sqrt(25))  // true
mySet.has('Some Text'.toLowerCase()) // true
mySet.has(o)       // true

mySet.size         // 5

mySet.delete(5)    // removes 5 from the set
mySet.has(5)       // false, 5 has been removed

mySet.size         // 4, since we just removed one value

console.log(mySet)
// logs Set(4) [ 1, "some text", {…}, {…} ] in Firefox
// logs Set(4) { 1, "some text", {…}, {…} } in Chrome
```

## Map

A `Map` object iterates its elements in insertion order — a [`for...of`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) loop returns an array of `[key, value]` for each iteration.

- Key equality is based on the [`sameValueZero`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness#Same-value-zero_equality) algorithm.
- [`NaN`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NaN) is considered the same as `NaN` (even though `NaN !== NaN`) and all other values are considered equal according to the semantics of the `===` operator.
- In the current ECMAScript specification, `-0` and `+0` are considered equal.

```javascript
let myMap = new Map()

let keyString = 'a string'
let keyObj    = {}
let keyFunc   = function() {}

// setting the values
myMap.set(keyString, "value associated with 'a string'")
myMap.set(keyObj, 'value associated with keyObj')
myMap.set(keyFunc, 'value associated with keyFunc')

myMap.size              // 3

// getting the values
myMap.get(keyString)    // "value associated with 'a string'"
myMap.get(keyObj)       // "value associated with keyObj"
myMap.get(keyFunc)      // "value associated with keyFunc"

myMap.get('a string')    // "value associated with 'a string'"
                         // because keyString === 'a string'
myMap.get({})            // undefined, because keyObj !== {}
myMap.get(function() {}) // undefined, because keyFunc !== function () {}
```

```javascript
let myMap = new Map()
myMap.set(0, 'zero')
myMap.set(1, 'one')

for (let [key, value] of myMap) {
  console.log(key + ' = ' + value)
}
// 0 = zero
// 1 = one

for (let key of myMap.keys()) {
  console.log(key)
}
// 0
// 1

for (let value of myMap.values()) {
  console.log(value)
}
// zero
// one

for (let [key, value] of myMap.entries()) {
  console.log(key + ' = ' + value)
}
// 0 = zero
// 1 = one
```

## Classes

```javascript
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}
```

An important difference between **function declarations** and **class declarations** is that function declarations are [hoisted](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting) and class declarations are not. You first need to declare your class and then access it, otherwise code like the following will throw a [`ReferenceError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError):

```javascript
const p = new Rectangle(); // ReferenceError

class Rectangle {}
```

A **class expression** is another way to define a class. Class expressions can be named or unnamed. The name given to a named class expression is local to the class's body. (it can be retrieved through the class's (not an instance's) [`name`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name) property, though).

```javascript
// unnamed
let Rectangle = class {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
};
console.log(Rectangle.name);
// output: "Rectangle"

// named
let Rectangle = class Rectangle2 {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
};
console.log(Rectangle.name);
// output: "Rectangle2"
```

## Extending Classes

```javascript
class DateFormatter extends Date {

  getFormattedDate() {
    const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
      'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
    return `${this.getDate()}-${months[this.getMonth()]}-${this.getFullYear()}`;
  }

}

console.log(new DateFormatter('August 19, 1975 23:15:30').getFormattedDate());
// expected output: "19-Aug-1975"
```

```javascript
class Animal { 
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a noise.`);
  }
}

class Dog extends Animal {
  constructor(name) {
    super(name); // call the super class constructor and pass in the name parameter
  }

  speak() {
    console.log(`${this.name} barks.`);
  }
}

let d = new Dog('Mitzie');
d.speak(); // Mitzie barks.
```

## Static Method

```javascript
class ClassWithStaticMethod {
  static staticMethod() {
    return 'static method has been called.';
  }
}

console.log(ClassWithStaticMethod.staticMethod());
// expected output: "static method has been called."
```

## Getters and Setters

```javascript
var o = {
  a: 7,
  get b() { 
    return this.a + 1;
  },
  set c(x) {
    this.a = x / 2;
  }
};

console.log(o.a); // 7
console.log(o.b); // 8 <-- At this point the get b() method is initiated.
o.c = 50;         //   <-- At this point the set c(x) method is initiated
console.log(o.a); // 25
```

