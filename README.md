# Safe Event Listener
_a memory-leak-resistant way to manage event listeners in JavaScript_
## Overview
The *Safe Event Listener Pattern* ensures that event listeners:
* don't leak memory when elements are dynamically removed
* aren't duplicated on the same element
* are cleaned up automatically without manual `removeEventListener` calls
This is accomplished using *WeakSet* to track elements and a *MutationObserver* to detect when an element is removed from the DOM.
Add the following as a utility function in your project:
```
const eventRegistry = new WeakSet();

export function attachSafeEventListener(element, event, handler) {
	if (!(element instanceof Element)) {
		throw new Error("attachSafeEventListener expects a valid DOM element.");
	}
	
	if (eventRegistry.has(element)) return; // Avoid duplicate event listeners.

	element.addEventListener(event, handler);
	eventRegistry.add(element);

	// Observe DOM mutations to detect when the element is removed
	const observer = new MutationObserver(() => {
		if (!document.body.contains(element)) {
			element.removeEventListener(event, handler);
				observer.disconnect(); // Cleanup observer
		}
	});

	observer.observe(document.body, { childList: true, subtree: true });
}
```
Then, use it just like `addEventListener`
```
const button = document.createElement("button");
button.textContent = "Click me!";
document.body.appendChild(button);

attachSafeEventListener(button, "click", () => alert("Button clicked!"));

// Simulate element removal after 5 seconds
setTimeout(() => {
	document.body.removeChild(button);
	console.log("Button removed, event listener automatically cleaned up!");
}, 5000);
