It
==

README.md / example.js
----------------------

`It` is a library to make it easier to create accessor/iterator functions,
for use with things like `Array.prototype.map` (`_.map`) or `Array.prototype.filter` (`_.filter`).

This library is inspired by the article
[Combinator Recipes for Working With Objects in JavaScript](https://github.com/raganwald/homoiconic/blob/master/2012/12/combinators_1.md)
by Reginald Braithwaite, but I want it to look more object oriented.

```javascript
var _ = require('underscore')
var It = require('./')
```

```javascript
var numbers = [3, 1, 4, 1, 5]
var strings = ['this', 'is', 'a', 'Book']
```

It
--
`It` provides an identity function, just like _.identity, but shorter.

```javascript
// map an array with itself... pretty useless
console.log(_.map(numbers, It))
```


```
[ 3, 1, 4, 1, 5 ]
```

```javascript
// get a sorted copy of an array
console.log(_.sortBy(numbers, It))
console.log(_.sortBy(strings, It))
```


```
[ 1, 1, 3, 4, 5 ]
[ 'Book', 'a', 'is', 'this' ]
```

.get
----
`.get` returns the value of a property. Here's where things get interesting.

```javascript
// equivalent to function(x) { return x.length }
var getLength = It.get('length')
```

```javascript
// sort the strings by their length
console.log(_.sortBy(strings, getLength))  // => ['a', 'is', 'this', 'Book']
```


```
[ 'a', 'is', 'this', 'Book' ]
```

.send
-----
`.send(...)` calls a method on an object

```javascript
// equivalent to function(x) { return x.toUpperCase() }
var toUpperCase = It.send('toUpperCase')
```

```javascript
// map all strings to uppercase
console.log(_.map(strings, toUpperCase)) // => ['THIS', 'IS', 'A', 'BOOK']
```


```
[ 'THIS', 'IS', 'A', 'BOOK' ]
```

```javascript
// therefore, case-insensitive sorting is easy
console.log(_.sortBy(strings, toUpperCase)) // => ['a', 'Book', 'is', 'this']
```


```
[ 'a', 'Book', 'is', 'this' ]
```

Chaining
--------
Of course, all of these are chainable.

```javascript
// equivalent to function(x) { return x.substr(0, 1).toUpperCase() }
var firstCharacterCapitalized = It.send('substr', 0, 1).send('toUpperCase')
console.log(_.map(strings, firstCharacterCapitalized)) // => ['T', 'I', 'A', 'B']
```


```
[ 'T', 'I', 'A', 'B' ]
```

Let's move to something more practical...

```javascript
// Here we have a list of people. (name generation thanks to chance.js)
var addressBook = [
  { first: 'Sifwa', last: 'Duhav', phone: '(416) 984-4454' },
  { first: 'Moc', phone: '(898) 983-5755' },
  { first: 'Diblacbo', last: 'Li', phone: '(258) 838-8314' },
  { first: 'Betu', last: 'Jol', phone: '(219) 234-9591' },
  { first: 'Fuhetu', last: 'Ra', phone: '(631) 437-2332' }
]
console.log(addressBook)
```


```
[ { first: 'Sifwa', last: 'Duhav', phone: '(416) 984-4454' },
  { first: 'Moc', phone: '(898) 983-5755' },
  { first: 'Diblacbo', last: 'Li', phone: '(258) 838-8314' },
  { first: 'Betu', last: 'Jol', phone: '(219) 234-9591' },
  { first: 'Fuhetu', last: 'Ra', phone: '(631) 437-2332' } ]
```

```javascript
// Let's sort them by the length of first name!
var firstNameLength = It.get('first').get('length')
console.log(_.sortBy(addressBook, firstNameLength))
```


```
[ { first: 'Moc', phone: '(898) 983-5755' },
  { first: 'Betu', last: 'Jol', phone: '(219) 234-9591' },
  { first: 'Sifwa', last: 'Duhav', phone: '(416) 984-4454' },
  { first: 'Fuhetu', last: 'Ra', phone: '(631) 437-2332' },
  { first: 'Diblacbo', last: 'Li', phone: '(258) 838-8314' } ]
```

.set
----
`.set(property, value)` sets a property on an object.
The result of this operation will be the invoked object, so you can chain

```javascript
// Let's set everyone's score to zero! Yes, scores in an address book!
_.each(addressBook, It.set('score', 0))
console.log(addressBook)
```


```
[ { first: 'Sifwa',
    last: 'Duhav',
    phone: '(416) 984-4454',
    score: 0 },
  { first: 'Moc', phone: '(898) 983-5755', score: 0 },
  { first: 'Diblacbo',
    last: 'Li',
    phone: '(258) 838-8314',
    score: 0 },
  { first: 'Betu', last: 'Jol', phone: '(219) 234-9591', score: 0 },
  { first: 'Fuhetu', last: 'Ra', phone: '(631) 437-2332', score: 0 } ]
```

.maybe
------
`.maybe(func)` invokes a function only if it's truthy.

```javascript
// One person in an address book doesn't have lastname.
// Without `.maybe()` we might call `.toLowerCase()` on `undefined`.
//
// equivalent to function(x) { return x.last && x.last.toLowerCase() }
var lastNameLowered = It.get('last').maybe(It.send('toLowerCase'))
console.log(_.map(addressBook, lastNameLowered))
```


```
[ 'duhav', undefined, 'li', 'jol', 'ra' ]
```

```javascript
// Then you can filter out falsy value by using `_.filter(..., It)`.
console.log(_.filter(_.map(addressBook, lastNameLowered), It))
```


```
[ 'duhav', 'li', 'jol', 'ra' ]
```

```javascript
// .or
// ---
// Instead of using `.maybe`, we can use `.or` to put a default value.
var lastNameLowered2 = It.get('last').or('None').send('toLowerCase')
console.log(_.map(addressBook, lastNameLowered2))
```


```
[ 'duhav', 'none', 'li', 'jol', 'ra' ]
```

.instantiate
------------
`.instantiate(Constructor)` can be used to quickly map things
into an instance.

```javascript
// Here we have a Person class.
function Person(info) {
  this.info = info
}
Person.prototype.getName = function() {
  return this.info.first + ' ' + this.info.last
}
Person.prototype.greet = function() {
  console.log('Hello! I am "' + this.getName() + '"')
}
```


```
```

```javascript
// We can map everyone in the address book into a new Person instance!
//
// equivalent to function(x) { return new Person(x) }
var people = _.map(addressBook, It.instantiate(Person))
_.each(people, It.send('greet'))
```

Hello! I am "Sifwa Duhav"
Hello! I am "Moc undefined"
Hello! I am "Diblacbo Li"
Hello! I am "Betu Jol"
Hello! I am "Fuhetu Ra"
It.self
-------
You can use `It.self` instead of `It` to use make a function that
uses the value of `this` instead of the value of passed argument.

You can use it to quickly make an accessor function

```javascript
// Return the first name.
//
// equivalent to function() { return this.info.first }
Person.prototype.getFirstName = It.self.get('info').get('first')
```

```javascript
// this function takes a last name, and returns a name suffix.
// no need to check of nulls here, we'll let `.maybe` do it.
function initial(string) {
  return ' ' + string.substr(0, 1) + '.'
}
```

```javascript
// Return the initial of last name.
// If the person does not have last name, then return empty string.
//
// equivalent to function() { return (this.info.last && initial(this.info.last)) || '' }
Person.prototype.getLastInitial = It.self.get('info').get('last').maybe(initial).or('')
```

```javascript
// redefine the greet function to make use of it
Person.prototype.getName = function() {
  return this.getFirstName() + this.getLastInitial()
}
_.each(people, It.send('greet'))
```

Hello! I am "Sifwa D."
Hello! I am "Moc"
Hello! I am "Diblacbo L."
Hello! I am "Betu J."
Hello! I am "Fuhetu R."
.compose
--------
You can use `.derive` to compose your own functionality.

```javascript
// Here we have vectors...
var vectors = [
  { x: 1, y: 5 }, { x: 5, y: 1 }, { x: 2, y: -3 }
]
```

```javascript
// We have a square function...
function square(x) {
  return x * x
}
```

```javascript
// Let's get the square of x and y components of these vectors!
console.log(_.map(vectors, It.get('x').compose(square)))
console.log(_.map(vectors, It.get('y').compose(square)))
```


```
[ 1, 25, 4 ]
[ 25, 1, 9 ]
```

You can also use `.compose` to chain functions together.

```javascript
var test = { a: { b: 1 }, b: { a: 2 } }
var getA = It.get('a')
var getB = It.get('b')
var getAB = getA.compose(getB)
var getBA = getB.compose(getA)
console.log(test)
console.log(getA(test))
console.log(getB(test))
console.log(getAB(test))
console.log(getBA(test))
```


```
{ a: { b: 1 }, b: { a: 2 } }
{ b: 1 }
{ a: 2 }
1
2
```

.tap
----
`.tap` invokes the passed function with the current value,
and returns the current value.

```javascript
// log the numbers and while mapping to get the squares
console.log(numbers)
console.log(_.map(numbers, It.tap(console.log).compose(square)))
```


```
[ 3, 1, 4, 1, 5 ]
3
1
4
1
5
[ 9, 1, 16, 1, 25 ]
```

```javascript
// make everyone greet while mapping to get their first name
console.log(_.map(people, It.tap(It.send('greet')).send('getFirstName')))
```


```
Hello! I am "Sifwa D."
Hello! I am "Moc"
Hello! I am "Diblacbo L."
Hello! I am "Betu J."
Hello! I am "Fuhetu R."
[ 'Sifwa', 'Moc', 'Diblacbo', 'Betu', 'Fuhetu' ]
```

