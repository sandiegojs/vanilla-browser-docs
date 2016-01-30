# The HTML

We should know what kind of document we are working with before we move forward. Since the boilerplate was setup for us, we'll need to open up the `app/index.html` file and take a look.

You should see that the styles are included in the header from `app/index.css` and the script for our JavaScript is included at the top of the body from `public/index.js`. All of our JavaScript files in `app/` are compiled by Gulp into a single file, `public/index.js`

**ProTip™:** When the browser encounters a `<script>` tag, it is blocking - meaning the browser will immediately download and execute it. To load the `<script>` asynchronously and not block the browser's parsing of the document, set the attribute `async='true'` on the tag.

Within the `app/index.html` you should see a form with labels and inputs. Familiarize yourself with the different fields.

Are you ready to get coding?

# HTML field validation

Have you ever used a form where you didn't realize you missed a required field until after clicking the submit button? Waiting for the page to send data off to the server, waiting for the page to reload, and then finally to get the perplexing red error message at the top. What a pain!

Let's save our users the hassle and let them know right away that they are required to fill in certain fields _before_ sending anything off to the server.

There are a few different ways to do this, and we will elaborate on this later, but the most basic way to get going with form validation is using the built-in HTML field validations.

## Required fields

We really only need the "name" and "email" field to be required. In order to make these inputs required, all we have to do is add the word `required` as an attribute on the `<input>` tag.

For example:

`<input type='text' placeholder='Name' required>`

Go ahead and update the "name" and "email" inputs to include this special attribute.

*After making any changes to this project, be sure to save the file and reload your browser!*

Let's head over to [localhost:3000][localhost] again and try it out.

If you hit the submit button now, you should immediately see a pop-up message near the first missing input telling you the field is required like below.

![required error box](https://s3.amazonaws.com/f.cl.ly/items/172n3w0f0h0f2d0y0i01/Screen%20Shot%202016-01-17%20at%204.43.54%20PM.png?v=b83630c4)

Neato! This is all taken care of for you by the browser out-of-the-box!

## Native validations

Some `<input>` types have intrinsic constraints, such as `type='email'`. If you look at the `app/index.html` you will see that our email field is currently setup as a `type='text'`. Go ahead and change it.

Once you have, we can test it out. Head over to the browser, type in a "Name" value (so that we don't get the required error) and type in a phoney string that doesn't look like an email address. Once you hit submit, you should see a nice message come up and tell you that your input doesn't look like an email.

![email error message](https://s3.amazonaws.com/f.cl.ly/items/3y1l2h0R2x1Q0S1r351B/Screen%20Shot%202016-01-17%20at%204.52.45%20PM.png?v=e72c1557)

If you want to learn more about validations that are available for inputs, [MDN has a great article covering the details][mdn-validations].

# The DOM

The Document Object Model (DOM) is how we are able to interact with our page via JavaScript. The DOM is a representation of what is on the page, including the elements and styles.

> The Document Object Model (DOM) is a programming interface for HTML, XML and SVG documents. It provides a structured representation of the document (a tree) and it defines a way that the structure can be accessed from programs so that they can change the document structure, style and content. The DOM provides a representation of the document as a structured group of nodes and objects that have properties and methods. Nodes can also have event handlers attached to them, and once that event is triggered the event handlers get executed. Essentially, it connects web pages to scripts or programming languages. -[MDN][mdn-dom]

One of the trickiest things about the DOM is that it's up to the browser vendor to implement it and therefore every implementation is a little different. This is why we get browser compatibility issues.

But as you will see there is a large API exposed by the DOM which allows us to craft powerful user experiences. An easy way to think of the DOM is a tree of nested nodes and each node is an element from our HTML document, e.g. a `div`, `p`, `button`, etc...

To visualize this all you need to do is open your web inspector and look at the **Elements** tab to see this structure realized. Each one of those nodes (elements) has many properties and functions you can take advantage of.

In Chrome on OSX you can open the web inspector with the keyboard shortcut `Command + Option + j` or `Command + Option + i` and `Control + Shift + j` on Windows.

![element inspector](https://s3.amazonaws.com/f.cl.ly/items/2o2Y3L1w433t2p1c2M38/kfm9fl7Prq.gif?v=8c898ff8)

# DOM selection

In order to interact with the DOM within JavaScript, the first thing you will need to understand is element selection. By selecting an element (DOM node) we get a reference to that element which allows us to take actions on it.

There are several functions we can use to get a reference to a DOM node. Overtime these functions have been introduced to solve different needs. As such, the browser support for them can vary. You should always research browser compatibility to ensure they will work for your user base.

- [querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector)
- [querySelectorAll](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll)
- [getElementById](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById)
- [getElementsByClassName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByClassName)
- [getElementsByName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByName)
- [getElementsByTagName](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagName)

For now, we are going to focus on `querySelector`. This function allows you to specify a [css selector][css-selector] that will be used to find the first matching node. One of the nice feature of this function is that it is available both on the `document` as well as `element`. This means you can search for an element in the entire document or narrow your search to the sub elements of an existing element.

For now, let's just open up the dev console and mess around. In Chrome OSX this can be accomplished with the keyboard shortcut `Command + Option + j` or `Command + Option + i` and `Control + Shift + j` on Windows. We can begin by selecting the input with a name attribute of `name`.

```js
var inputName = document.querySelector('input[name="name"]')
```

This sets the variable inputName to an [`HTMLInputElement`][html-input] which has a `value` property available. Go ahead and set the value property to see an example of how you can change the DOM.

```js
inputName.value = 'My new value!'
```

You should see the value you set reflected in the form's input.

Whenever dealing with a DOM node it's important to understand what type of element you have and what are its parents. That will determine the properties and functions available to you.

# Event handling

When users interact with the web page the DOM publishes these interactions as events, for example `click`, `scroll`, `keypress`, and [more][mdn-events].

After selecting an element from the DOM, we can call its [`addEventListener`][event-listener] method which will execute a callback function we provide any time that event occurs. Lets start by listening for a `click` on the `document` and trigger an `alert` whenever the user clicks anywhere on the page.

We are going to keep working in the dev console for now, and continue to mess around. Once we get the hang of event handlers, we can start building our app. But for now, let's just keep using the dev console.

```js
var handler = function() {
  alert('user clicked on the page')
}

document.addEventListener('click', handler)
```

Now if you click on the page you should receive an alert with your message. Alerting users every time they click the page can get annoying so lets remove that event listener.

```js
document.removeEventListener('click', handler)
```

It's important to note that in order for us to be able to remove an event listener we need to name our functions so we can specify what to remove for that event.

Using this simple API we can trigger the complex logic we will be writing shortly.


## Multiple event listeners

One of the great things about event listeners is that we can attach multiple listeners per event. For example:

```js
var handlerOne = function() {
  // Do some logic
}

var handlerTwo = function() {
  // Do some logic
}

document.addEventListener('click', handlerOne)
document.addEventListener('click', handlerTwo)
```

Now both of these functions will be executed whenever someone clicks the page.

## Event listener parameter

When we attach callback functions to events these functions are passed as an event argument. The type of event given will change depending on the type input device which triggered it. Since we are listening for a `click` event we will receive a [`MouseEvent`][mouse-event].

```js
var logEvent = function(evt) {
  console.log(evt)
}

document.addEventListener('click', logEvent)
```

If you inspect this event in the console you will see there are quite a few properties available to us. The ones most commonly used are `target` and `currentTarget`.

`target` is the element who dispatched the event.

`currentTarget` is the element who handled the event during the event capture (a.k.a bubbling) phase.

Although important, we won't dive into the distinction between these two just yet. For our purposes we are going to use the `currentTarget` event to always get a reference to the element who has the event listener listening for the event.

[css-selector]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_Started/Selectors
[event-listener]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
[html-input]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement
[jquery]: https://jquery.com/
[localhost]: http://localhost:3000
[mdn-dom]: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model
[mdn-events]: https://developer.mozilla.org/en-US/docs/Web/Events
[mdn-validations]: https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5/Constraint_validation
[mouse-event]: https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent
[zepto]: http://zeptojs.com/
