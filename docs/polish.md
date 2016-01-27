## Add JS Validation
Earlier you added form validation using attributes. This is a quick and easy way to perform validation, but it does have limits. For example, you can't add custom error messages or styling. To get more flexibility and control, use JavaScript.

In this section, you will be playing with the HTML5 constraint validation API to check and customize the state of a form element. You have several goals:

- Provide a custom validation message when the Email the user types is invalid
- Hide that custom validation message as soon as the email is valid
- Add custom validation logic to the State field without using an attribute
- Apply custom styling to the validation error messages
- Prevent form submission if there are validation errors.

To achieve these goals you will:

0. Add a keyup event listener to the email field to present a custom message
0. Add `validateForm` function to handle validation logic
0. Modify the form submit event listener to validate the form
0. Loop through the form elements
0. Check the validity of each field
0. Set or clear the validation error messages
0. Add custom validation logic to the state field


### Handle `keyup` event for email field

0. Get a reference to email field. Hint: Use a query selector you learned about earlier.
0. Add a `keyup` event listener attached to the email field.
0. Inside the event listener function, you will check the `email.validity.typeMismatch` property to determine if the email the user has typed is valid.

```js
if (email.validity.typeMismatch) {

} else {

}
```

If there is a problem `typeMatch` will be `true`. This is your opportunity to set a custom error message using the `setCustomValidity` function.

```js
if (email.validity.typeMismatch) {
  email.setCustomValidity('Oops, try a real email address.');
} else {
  email.setCustomValidity('');
}
```

If you pass `''` to `setCustomValidity` the field will be considered valid.

The complete event handler should look like this:

```js
var email = document.querySelector('input[name="email"]');

// Check for valid email while the user types
email.addEventListener('keyup', function(event) {
  // see https://developer.mozilla.org/en-US/docs/Web/API/ValidityState for
  // other validity states
  if (email.validity.typeMismatch) {
    email.setCustomValidity('Oops, try a real email address.');
  } else {
    email.setCustomValidity('');
  }
}, false);
```

Try the new logic. Go to your form, add a name, but leave the email blank. Click Submit. Start typing your name in the Email field. Notice that now you see your custom message. Now type a valid email address. The message goes away as soon as the address is valid.

### Add `validateForm` function to handle validation logic

You can use the technique above to handle each validated field, but you are still stuck with the standard formatting for the messages. To get more control, you need to turn off the automatic validation and handle the submission event yourself. 

Add a line to prevent the automatic validation behavior.

`form.noValidate = true;`

ProTip: You can achieve the same thing by adding a `novalidate` attribute to the form.

With the automatic validation turned off, you have more control over when the validation check is done. 

Add a new function, `validateForm`, to handle the validation check. This function, after you implement it fully, will determine any field errors and allow the form to determine if it valid. 

```js
function validateForm(form){

}
```

### Modify the `submit` event listener

Back in the "Add submit event" section, you added a `submit` event handler. You will modify that to call the validation logic. Inside the `submitHandler` function, you will call `validateForm`.

```js

function submitHandler(evt) {
  evt.preventDefault();
  
  var form = evt.target;
  validateForm(form);
  
  // remaining submit handler logic goes here

}
```
 
Now you need to have the form check if it is valid before submitting data to the server. The `form` element has a convenient `checkValidity` function to do just that. 

```js

function submitHandler(evt) {
  evt.preventDefault();
  
  var form = evt.target;
  validateForm(form);
  
  if (form.checkValidity()) { // if no error, go ahead and submit
    // remaining submit handler logic goes here
  }
}
```

In the next few steps, you will implement the validation logic for the fields.

### Loop through the fields

Every form has an elements array. You can use that to loop through the fields.

In the `validateForm` function, create the loop.

```js
function validateForm(form) {
  var f;
  for (f = 0; f < form.elements.length; f++) {

  }
}
```

Within the loop, get a reference to the current field. `validateForm` should now look like:

```js
function validateForm(form) {
  var f;
  var field;

  for (f = 0; f < form.elements.length; f++) {
    field = form.elements[f];
  }
}
```

The behavior of the form hasn't changed so charge on to the next section.

### Check the field validity

Use a function to encapsulate all of the field validation logic. Start with using the standard validation logic.

Create a new function `isValid` that takes a field as a parameter.

Like the form, fields have a `checkValidity` method. Use this method to determine the return value for the function. Here's how it should look:

```
function isValid(field) {
  return field.checkValidity();
}
```

Call this function from the loop inside the `validateForm` function.

```js
function validateForm(form) {
  var f;
  var field;

  for (f = 0; f < form.elements.length; f++) {
    field = form.elements[f];
    if(isValid(field)) {
      // remove error styles and messages
    } else {
      // style field, show error, etc.
    }
  }
}
```

### Set and clear error messages

Now that you are looping through each field and checking its validity, you can use the information to format the feedback to the user. In the HTML, you may have noticed that below the fields that have validation, there is a `span` element. You will use this element to display feedback to the user.

Create a `setError` function that takes a field as a parameter.

Inside the function, get a reference to the `span` you will use to show the error. To do that, use the handy `nextElementSibling`.

Using the reference to the element, set its `innerHTML` to the field's `validationMessage`. The `validationMessage` is either the default message from the browser or a custom message you set. The finished function should loop like:

```js
function setError(field) {
  var error = field.nextElementSibling;
  if (error) error.innerHTML = field.validationMessage;
}
```

Set up a similar function to clear the error text. Copy and paste the `setError` function.

Change the function name to `clearError`.

Instead of `field.validationMessage`, set the `innerHTML` to `''`. The complete function is:

```js
function clearError(field) {
  var error = field.nextElementSibling;
  if (error) error.innerHTML = '';
}
```

ProTip: This is just one way of handling error messages. For example, you could use css to show and hide error messages or put all validation errors in a central location on the page. 

Finish the logic by calling these function from `validateForm`. If the field is valid, call `clearError` and if not call `setError`.

```js
function validateForm(form) {
  var f;
  var field;

  for (f = 0; f < form.elements.length; f++) {
    field = form.elements[f];
    if(isValid(field)) {
      // remove error styles and messages
      clearError(field);
    } else {
      // style field, show error, etc.
      setError(field);
    }
  }
}
```

Try it out. Click Submit. Do you see the error messages? Do the error messages disappear when you enter valid data?

As you type in the email field, does the message change? No? What is missing from the `keyup` event listener?

### Add custom validation logic to the state field

Time for the bonus round! Custom validation logic could involve evaluating multiple fields on the form, making a call to the server, or performing some calculation in a worker thread. While those are beyond the time you have for this workshop, this section will walk you through creating a some custom logic. Keep in mind that there are better ways to enforce the logic presented in the here, but hey, use your imagination. ;-)

You will add this logic to the top of the `isValid` function you created earlier.

First, check if the field passed to the function is named `state`.

If so, check if the state is in the list of valid states. Valid states include: CA, TX, NY.

```js
function isValid(field) {
  // custom logic for state
  if (field.name === 'state') {
    var validStates = ['CA', 'TX', 'NY'];
    if (validStates.indexOf(field.value) === -1) {
      // invalid state
    } else {
      // valid state
    }
  }
  return field.checkValidity();
}
```

When the state is invalid, set a custom validity message. When it is not, set an empty string.

```js
    ...
    if (validStates.indexOf(field.value) === -1) {
      // invalid state
      field.setCustomValidity('Use CA, TX, or NY');
    } else {
      // valid state
      field.setCustomValidity('');
    }
    ...
```

Now when you try to submit without a state or an invalid state, you will get an error message just like with name and email.

That's it for validation! You may now add "HTML5 constraint validation API" to your resume.

## Adding classes to style error message or boxes

## Welcome text with cookies


[class-list]: https://developer.mozilla.org/en-US/docs/Web/API/Element/classList
[cookies]: https://devdocs.io/dom/document/cookie
[css-selector]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_Started/Selectors
[devdocs]: http://devdocs.io
[dom]: https://devdocs.io/dom/
[event-listener]: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
[events]: https://devdocs.io/dom_events
[git-scm]: http://git-scm.com/downloads
[gulp]: http://gulpjs.com/
[heroku-node]: https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction
[heroku-toolbelt]: https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up
[html-input]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement
[insert-before]: https://developer.mozilla.org/en-US/docs/Web/API/Node/insertBefore
[jquery]: https://jquery.com/
[localhost]: http://localhost:3000
[mdn]: https://developer.mozilla.org/en-US/
[mdn-appendchild]: https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild
[mdn-createelement]: https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement
[mdn-createtextnode]: https://developer.mozilla.org/en-US/docs/Web/API/Document/createTextNode
[mdn-events]: https://developer.mozilla.org/en-US/docs/Web/Events
[mdn-foreach]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
[mdn-validations]: https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5/Constraint_validation
[mocha]: https://devdocs.io/mocha/
[mouse-event]: https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent
[node-install]: https://nodejs.org/download/
[node-list]: https://developer.mozilla.org/en-US/docs/Web/API/NodeList
[npm-g-without-sudo]: https://github.com/sindresorhus/guides/blob/master/npm-global-without-sudo.md
[onreadystatechange]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/onreadystatechange
[rails-api]: https://github.com/rails-api/rails-api
[remove-node]: https://developer.mozilla.org/en-US/docs/Web/API/ChildNode/remove
[san diego js]: http://sandiegojs.org/
[sdjs-app]: //sandiegojs-vanilla-workshop.herokuapp.com
[xhr]: https://devdocs.io/dom/xmlhttprequest
[xhr-mdn]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest
[zepto]: http://zeptojs.com/
