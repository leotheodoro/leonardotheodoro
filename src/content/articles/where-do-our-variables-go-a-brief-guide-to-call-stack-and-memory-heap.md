---
title: "Where do our variables go - A brief guide to Call Stack and Memory Heap"
date: 2025-02-19
---

Have you written countless JavaScript codes and developed various systems, but have you ever stopped to wonder how some things work behind the scenes?

Today, I want to introduce you to two terms that I didn't know before and found interesting to learn about: Call Stack and Memory Heap.

## JavaScript Memory Model
First, we need to understand what happens when we declare a variable and initialize it. In the example below, I declared the variable `x` with the value of 1 and created another variable equivalent to `x`, then changed the value of `x`.

```javascript
let x = 1
let y = x

x = x + 1;

console.log(y) // Result?
```

What do you think will happen when we print `y`?

If you think the value of `y` will be 2, you're wrong—but let me explain why.

1. When `x` was created, JavaScript generated a unique identifier and allocated it to a memory address (e.g., `M001`) while also storing the variable's value at that memory address.
2. When I said `let y = x`, I meant that **`y` is equivalent to the memory address `M001` and not equal to the value of `x`.**
3. When I changed the value of `x`, **JavaScript allocated a new memory address and saved the value 2 at address `M002`.** **This happens because primitive data types (string, number, boolean, undefined, and symbol) are immutable.**

## What is the Role of the Call Stack?
The Call Stack works like a stack (FILO - First In, Last Out). **It organizes function calls, adding and removing them after the code is executed.** Additionally, **it is responsible for storing primitive types.**

It’s also in the Call Stack that a phenomenon you’ve probably heard of occurs: **Stack Overflow.** This happens when functions are stacked indefinitely, and the stack exceeds the maximum memory limit.

Here’s a quick example of how we can cause a stack overflow using recursion:

```javascript
function recursion() {
	recursion();
}

recursion();
```

## So, what does the Memory Heap do?
Unlike the Call Stack, the Memory Heap is where non-primitive data (arrays, objects, and functions) is stored. It allows data to grow dynamically and be organized in a non-sequential manner.

Here’s how the Memory Heap works. Check out this example:

```javascript
const arr = [];
arr.push(1);
```

In this case, when we declare the variable `arr`, the **Call Stack saves a reference value pointing to the Memory Heap, where the actual values are managed.** In this case, the `push` modifies the Heap's content but not the reference address in the Call Stack.

### `const` and Non-Primitive Data
When we declare a variable with `const`, we cannot reassign its value. However, for arrays and objects, it’s possible to modify the items internally. This happens because the identifier in the Call Stack remains the same, while the values are managed in the Heap.

In summary, when we perform a `push` on an array, it doesn’t change the memory address—it changes the value in the Heap. The same applies to objects.

Here’s a reference table to make this clearer:

| Variable | Call Stack             |       | Heap                |       |
| -------- | ---------------------- | ----- | ------------------- | ----- |
|          | Address                | Value | Address             | Value |
| x        | CS001                  | 2     |                     |       |
| y        | CS002                  | 1     |                     |       |
| arr      | CS003 (points to H001) |       | H001 (points to ->) | []    |

### Can Primitive Types Ever "Go" to the Heap?
Yes, there are cases where primitive types can "go" to the Heap, and this happens indirectly when you use a wrapper.

For example, if I declare a variable as `new String("Text")` instead of a plain string, the value is treated as an object and stored in the Heap.

## Garbage Collection and Memory Leaks
JavaScript has automatic memory management, meaning it cleans up unreferenced addresses (i.e., those no longer in use) automatically. This process is called "garbage collection" and uses the [Mark and Sweep](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Na%C3%AFve_mark-and-sweep) algorithm, which identifies and removes inaccessible variables.

However, Memory Leaks (when available memory is exceeded) can still occur. Here are the most common causes of Memory Leaks:

**Global Variables**
Global variables are difficult to collect because they are always accessible:

```javascript
const x = 1
const y = 2;
const z = 3
```

**Event Listeners**
Event listeners that are not removed can remain active, causing leaks:

```javascript
const button = document.getElementById('myButton'); 

// Add an event listener to the button
button.addEventListener('click', () => { console.log('Button clicked!'); }); 

// Problem: the listener will not be removed even if the button is deleted from DOM 
document.body.removeChild(button);
```

**setInterval**
Funções dentro de um `setInterval` nunca são coletadas, a menos que o intervalor seja limpo:

```javascript
setInterval(() => {
	let x = 1;
})
```

## Conclusion
Understanding the JavaScript memory model is essential to writing more efficient code and avoiding issues like memory leaks.

Remember:
1. Primitive types are immutable and stored in the Call Stack as values.
2. Non-primitive types reside in the Memory Heap and are stored as references.
3. Exceeding the stack limit causes a Stack Overflow.
4. Exceeding the available memory causes Memory Leaks.

And if you want to learn more about value and reference variables, read this [article](https://dev.to/thdr/why-cant-we-compare-arrays-and-objects-with--5h0j) I wrote, explaining why we can't compare arrays and objects using `===`.

I found it interesting to learn a bit about this and hope you find it helpful too. See you next time!