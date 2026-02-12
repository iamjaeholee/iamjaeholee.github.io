---
layout: post
title: "JavaScript Event Bubbling: Why Parents React to a Child's Click"
date: 2026-02-12 14:00:00 +0900
categories: [JavaScript, Web]
tags: [javascript, frontend, webdev, event-bubbling]
author: Lee Jaeho
description: A comprehensive guide to understanding event bubbling and delegation in JavaScript.
---

# JavaScript Event Bubbling: Why Parents React to a Child's Click

In JavaScript, when you click a button inside a `div`, the click event doesn't just stay on the button. It travels up to its parents, then to the document, and finally to the window. This phenomenon is called **Event Bubbling**.

Understanding this concept is crucial for managing complex UI interactions and optimizing performance through event delegation.

---

## What is Event Bubbling?



**Event Bubbling** describes the process where an event starts from the deepest target element and moves up through its ancestors in the DOM tree. Like bubbles rising in a glass of soda, the event propagates upward.

1. You trigger an event on the **target element** (e.g., clicking a button).
2. The event handler for the target element runs.
3. The event then moves to the **parent element** and runs its handler.
4. This continues all the way up to the root (`<html>`, `document`, `window`).

---

## Code Example

Consider the following HTML structure:

```html
<div onclick="alert('Parent Div')">
  <button onclick="alert('Button')">Click Me</button>
</div>

```

If you click the button, you will see two alerts in this order:

1. `'Button'`
2. `'Parent Div'`

Even though you only clicked the button, the event **bubbled up** to the `div`, triggering its click listener as well.

---

## How to Stop It: `event.stopPropagation()`

Sometimes, you want to prevent the parent from reacting to an event triggered by a child. In such cases, you can use the `stopPropagation` method.

```javascript
button.addEventListener('click', (event) => {
  event.stopPropagation();
  console.log('Only the button reacts.');
});

```

By calling this method, the event "stops in its tracks" and does not move further up the DOM tree.

---

## The Better Way: Event Delegation

Is bubbling always a problem? Not at all! We can use it to our advantage through **Event Delegation**. Instead of attaching a listener to every single child element, we attach **one listener to a common parent**.

### Benefits of Event Delegation:

* **Memory Efficiency:** Fewer event listeners are created.
* **Dynamic Elements:** It works automatically for elements added to the DOM later.
* **Clean Code:** Reduces boilerplate and makes the codebase more maintainable.

---

## Conclusion

Event Bubbling is a core mechanism of the browser's event system. By mastering bubbling and delegation, you can handle user interactions more efficiently and write much cleaner, more performant JavaScript code.