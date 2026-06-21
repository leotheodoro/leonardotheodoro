---
title: "Why can’t we compare arrays and objects with ==="
date: 2025-01-29
---

Hi everyone, I decided to write this article to answer a question I’ve always had: why can’t we compare arrays and objects with `===`? So, I did some research, and this is what I found about how JavaScript works under the hood.

JavaScript has five data types that are passed by **value**: `boolean`, `null`, `undefined`, `string`, and `number`. These are called **primitive types**.

JavaScript has three data types that are passed by **reference**: `array`, `function`, and `object`. Technically, all three are objects, so we can refer to them collectively as objects. These are called **non-primitive types**.

## Primitive Types (Value)
When you declare a variable of a primitive type, it stores the value directly.

```javascript
const name = 'John'
const age = 25
```

| Variable | Value  |
| -------- | ------ |
| name     | 'John' |
| age      | 25     |

Simple as that.

## Non-Primitive Types (Reference)
Variables holding non-primitive values are assigned a reference to that value. This reference points to the location of the object in memory. In this case, the variable stores the reference, not the value itself.

For example:

```javascript
const fruits = []
fruits.push('Banana')
```

Here’s a representation of what happens in these lines:

First line:

| Variable | Value | Address | Object |
| -------- | ----- | ------- | ------ |
| fruits   | H001  | H001    | []     |

Second line:

| Variable | Value | Address | Object     |
| -------- | ----- | ------- | ---------- |
| fruits   | H001  | H001    | ['Banana'] |

## What happens when we declare a reference variable?
When a reference variable is copied using `=`, the reference address is copied, not the value. **Objects are copied by reference, not by value.**

```javascript
const fruits = ['Banana']
const yellowFruits = fruits
```

| Variable     | Value | Address | Object     |
| ------------ | ----- | ------- | ---------- |
| fruits       | H001  | H001    | ['Banana'] |
| yellowFruits | H001  |         |            |

If we add a fruit to the `yellowFruits` array, we’re also adding it to the `fruits` array since they share the same reference.

```javascript
yellowFruits.push('Pineapple')
```

| Variable     | Value | Address | Object                  |
| ------------ | ----- | ------- | ----------------------- |
| fruits       | H001  | H001    | ['Banana', 'Pineapple'] |
| yellowFruits | H001  |         |                         |

## Reassigning a reference variable
When we reassign a reference variable, it replaces the reference, not the value.

```javascript
let person = { name: 'John' }
person = { name: 'Mary' }
```

In memory, this happens:

| Variable | Value | Address | Object           |
| -------- | ----- | ------- | ---------------- |
| person   | H002  | H001    | { nome: 'John' } |
|          |       | H002    | { nome: 'Mary' } |
The old reference still exists, but the variable now points to the new reference.

## Why can’t we compare arrays with `===`?
When `===` is used with reference types (non-primitive data), the condition will only return true if both variables have the same reference. This is because **`===` checks references, not values**, for non-primitive types.

```javascript
const arr1 = ['1']
const arr2 = arr1

console.log(arr1 === arr2) // True
```

However, if they are different objects, even with the same value, the references differ, so the comparison returns `false`.

```javascript
const arr1 = ['1']
const arr2 = ['2']

console.log(arr1 === arr2) // False
```

## Function Parameters
When we pass primitive values as parameters, the behavior is simple: the value is copied into the parameter, as if using `=`.

```javascript
const score = 10
const multiplier = 2

function multiply(x, y) {
	return x * y
}

const result = multiply(score, multiplier)
```

Here, `score` is `10`, `multiplier` is `2`, `x` is `10`, and `y` is `2`. This happens because the value is copied into the parameter.

| Variable   | Value | Address | Object        |
| ---------- | ----- | ------- | ------------- |
| score      | 10    |         |               |
| multiplier | 2     |         |               |
| multiply   | H001  | H001    | function(x,y) |
| x          | 10    |         |               |
| y          | 2     |         |               |

### Pure Functions
Pure functions don’t affect anything outside their own scope, and all variables used within them are cleared once the function returns a value.

The function `multiply` above is an example of a pure function.

### Impure Functions
Impure functions can receive objects and modify their state outside their scope. This happens because they receive the reference of the value, and any modification to this reference within the function affects the object outside as well. For example:

```javascript
function changeName(person) {
	person.name = 'Mary'
	return person
}

const john = {
	name: 'John'
}

const mary = changeName(john);

console.log(john) // Mary
console.log(mary) // Mary
```

This happens because within the function, we’re modifying the value at the reference, which is shared between `john` and `maria`.

### Making it a pure function
To make this function pure, we create a copy of the object inside the function, manipulate it, and return the copy. This avoids altering the original reference.

Here’s the refactored version:

```javascript
function changeName(person) {
	const newPerson = JSON.parse(JSON.stringify(person))
	newPerson.name = 'Mary'
	return newPerson
}

const john = {
	name: 'John'
}

const mary = changeName(john);

console.log(john) // John
console.log(mary) // Mary
```

In this function, we convert the object to a string (a primitive type) and back to an object, creating a new reference. The original reference remains untouched.

## Conclusion

1. **Primitive Types**: Store values directly in variables.
2. **Non-Primitive Types**: Store a reference to the value.
3. **Reference Copying**: Objects sharing the same reference reflect changes mutually.
4. **`===` Operator**: Compares references, not values, in non-primitive types.
5. **Pure Functions**: Don’t alter values outside their scope.
6. **Impure Functions**: Can modify the state of shared references.
7. **Making Functions Pure**: Creating copies prevents changes to original references.

If you’d like to learn more about how variables are stored in JavaScript, check out this [article](https://dev.to/thdr/where-do-our-variables-go-a-brief-guide-to-call-stack-and-memory-heap-jge) I wrote about the Call Stack and Memory Heap.

Hope you enjoyed this, see you soon!
