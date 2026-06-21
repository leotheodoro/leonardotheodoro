---
title: "Before TDD: Why you need to know what Mocks, Stubs, and Spies are?"
date: 2025-01-09
---

Hello, everyone! Today, I bring a topic that I believe is very interesting. I know there are dozens of articles online discussing TDD, BDD, design patterns for testing, how to write tests, and many other related topics. However, I see very few articles explaining more basic terms within the testing universe—those functions we often use but don’t always fully understand what they mean or how they behave. If you’re just starting to learn about testing and don’t know exactly what the library functions do, this article is for you. Enjoy the read!

# What are mocks?
The first thing you might come across as soon as you start writing tests is mocks. Sometimes you already use them but don’t know exactly what they mean. So, let’s dive in.

Mocks are primarily used in unit testing. They are tools used to simulate content, objects, or responses that usually come from an external dependency, or when you need the content to have specific information.

Imagine you’re testing a movie recommendation system. This system fetches a list of movies from an API and returns it to you. 

The problem is: if every time you run the tests, the real API is called, it could be slow and inconsistent (movies might vary, or the API could be down), making the tests unreliable.

Alright, Leo, I get the problem, **but how does a mock solve this?** Well, it’s simple: instead of calling the API, you use its response as a static list of movies. It’s basically “faking” the API response with that list of movies.

In the movie system example, if you want to test a function called `fetchMoviesFromAPI()` that uses the API to fetch movies, you can create a mock to simulate the API response like this:

```javascript
// This is the mock
const MOVIES_FROM_API = [
	{
		id: 1,
		name: "Interstellar"
	},
	{
		id: 2,
		name: "Nosferatu"
	}
]

// Here, we’re telling fetchMoviesFromAPI to return our mock instead of calling the real API. This is a stub, which you’ll learn about in the next section.
const fetchMoviesFromAPI = jest.fn().mockResolvedValue(MOVIES_FROM_API)

;(async () => {
	{
		const expectedMovies = MOVIES_FROM_API
		const movies = await fetchMoviesFromAPI()

		expect(movies).toEqual(MOVIES_FROM_API)
	}
})()
```

With mocks, your tests become more efficient since they don’t depend on external services. Moreover, they gain reliability because you have total control over the returns, allowing the focus to remain on validating the functionality without worrying about potential API instabilities or downtime.

**Mocks are static objects that simulate responses from calls or other objects needed for testing.**

In the end, it’s like testing a car without using real gasoline. You create a controlled environment to ensure the engine is working before taking it out on the road.

# I get mocks, now what are stubs?
Stubs are also testing tools but serve a slightly different purpose. They replace the behavior of functions with something predetermined, often using mocks to return specific values.

Stubs replace the function’s behavior. For example, when I access that movie API, the function won’t make the real call but will look at our mock (the static list of movies).

They also serve as a reminder that **our tests shouldn’t depend on external services or the internet.**

Let me give you some context: imagine you’re testing an application that calculates the total value of an online purchase. The calculation includes fees fetched from an external service. Every time you run the test, this calculation needs to be done, meaning the external service would need to be called. This could result in a slow, unstable, costly (because the external service might charge per request), and inconsistent test (values could change).

Using a stub, you’ll replace the real service call with a fixed, predefined value (yes, a mock). Instead of calling the fee service, you say: *“Always return the value 10 as the fee.”*

Imagine you want to test the function `calculateTotalPurchase()` that sums up the cart items’ values and adds the shipping fee. Using stubs, you replace the shipping fee service with a value that always returns “10” as the shipping fee. Like this:

```javascript
// Function we want to test
async function calculateTotalPurchase(items, getShippingFee) {
	const shippingFee = await getShippingFee()
	const totalItems = items.reduce((sum, item) => sum + item.price, 0)
	return totalItems + shippingFee
}

// Stub to replace the real shipping service
const getShippingFee = jest.fn().mockResolvedValue(10)

;(async () => {
	// Items in the cart
	const cart = [
		{ id: 1, name: "Item 1", price: 50 },
		{ id: 2, name: "Item 2", price: 30 }
	]

	// Calculating the total with the stub
	const total = await calculateTotalPurchase(cart, getShippingFee)

	// Testing the result
	expect(total).toBe(90) // 50 + 30 + 10
})()
```

This simplifies the test and ensures its reproducibility, meaning it will always work the same way. Moreover, stubs help isolate the test, eliminating the need to worry about the state or availability of the fee API.

In summary, it’s like testing a cake recipe using a measuring cup that always says 200ml of milk instead of measuring the actual amount of milk. This way, you’re only testing if you can mix the ingredients without worrying about whether the milk is being measured correctly.

# Mocks, stubs... and finally, what are spies?
We’ve explored mocks, which simulate objects, and stubs, which mimic function behaviors. Now, let’s talk about spies: what exactly do they do?

Spies monitor functions, recording how many times they were called, what parameters they received, and the results of each execution. They allow you to observe the function’s behavior without altering it, ensuring everything is working as expected.

Imagine you’re testing the notification module of your project. Every time an order is completed, the system should send a message to the customer and log an entry. In this case, you only want to make sure these actions are performed but don’t want to replace any of them. With a spy, you monitor these functions without altering their behavior. This allows you to see:
- If the function was called
- How many times it was called
- What arguments it received

For example, if you want to test the `completeOrder()` function that sends a notification to the customer and logs the entry, with a spy, you can verify:
- If the notification function was called
- If the log function was called
- What arguments these functions received.

```javascript
// Function we want to test
async function completeOrder(customer, sendNotification, logEntry) {
	await sendNotification(customer)
	saveLog(`Order completed for customer: ${customer.name}`)
}

// Mocks for auxiliary functions
const sendNotification = jest.fn()
const logEntry = jest.fn()

;(async () => {
	// Mock customer
	const customer = { id: 1, name: "John Doe" }

	// Calling the function we want to test
	await completeOrder(customer, sendNotification, logEntry)

	// Checking if auxiliary functions were called
	expect(sendNotification).toHaveBeenCalled()
	expect(sendNotification).toHaveBeenCalledWith(customer)

	expect(saveLog).toHaveBeenCalled()
	expect(saveLog).toHaveBeenCalledWith("Order completed for customer: John Doe")
})()
```

To conclude, it’s like placing a camera to observe what a chef does in the kitchen. You don’t interfere with what they’re doing; you just check if they’re following the recipe correctly.

So, that’s it! You’ve learned and understood the terms mocks, stubs, and spies, which are fundamental elements for creating reliable and efficient tests. Now you can continue deepening your studies. See you there, goodbye!

