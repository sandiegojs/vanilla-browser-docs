# Add JS Validation

Earlier you added form validation using attributes. This is a quick and easy way to perform validation, but it does have limits. For example, you can't add custom error messages or styling. To get more flexibility and control, use JavaScript.

In this section, you will be playing with the HTML5 constraint validation API to check and customize the state of a form element. You have several goals:

- Provide a custom validation message when the email the user types is invalid
- Hide that custom validation message as soon as the email is valid
- Add custom validation logic to the State field without using an attribute
- Apply custom styling to the validation error messages
- Prevent form submission if there are validation errors.

To achieve these goals you will:

0. Add a keyup event listener to the email field to present a custom message
0. Add `validateForm` function to handle validation logic
0. Modify the form submit event listener to validate the form
0. Loop through the form elements and check their validity
0. Set or clear the validation error messages
0. Add custom validation logic to the state field


## Validate email field

First let's add a `keyup` listener to check the validity of the email field as the user types. In order to do this we will need a reference to the email field.

```js
var email = document.querySelector('[name="email"]')
```

Inside of our `keyup` listener method, we will want to check the validity of the email. We can do this by getting a reference to the `currentTarget` on the passed in `event` object. This is going to be our `email` input.

```js
var email = document.querySelector('[name="email"]')

var emailListener = function(evt) {
  var input = evt.currentTarget
}
email.addEventListener('keyup', emailListener)
```

We can then check the input's `validity.typeMismatch` property in order to determine if the email the user has typed is valid or not.

If there is a problem with the email validity, `typeMatch` will be `true`. This is our opportunity to set a custom error message using the `setCustomValidity` function.

If we pass `''` to `setCustomValidity` the field will be considered valid. There are different types of [validityStates][validity-state].

```js
var email = document.querySelector('[name="email"]')

var emailListener = function(evt) {
  var input = evt.currentTarget
  if (input.validity.typeMismatch) {
    input.setCustomValidity('Oops, try a real email address.')
  } else {
    input.setCustomValidity('')
  }
}
email.addEventListener('keyup', emailListener)
```

Let's try out the new logic. When we visit our form, add our name, type in an invalid email address, and hit submit, we now are able to see our custom error message.


## Validate form

We can use the technique above to handle each validated field, but we are still stuck with the standard formatting for the messages. To get more control, we will need to turn off the automatic validation and handle the submission event ourselves.

In order to do this, we need to disable the automatica validation behavior on our form.

```js
form.noValidate = true
```

ProTip™: The same thing can be achieved byadding a `novalidate` attribute to the form element.


We can confirm that validation is off now by entering garbage data in the form and submitting.

With the automatic validation turned off, we now have more control over when the validation check is done.

Let's create a new function, `validateForm`, to handle the validation check. This function, after we implement it fully, will determine any field errors and allow the form to determine if it valid.


```js
var validateForm = function(form) { }
```

### Update submit handler

Back in the "Add submit event" section we added a `submit` event handler. We need to modify that to call the `validateForm` method.

```js
var submitHandler = function(evt) {
  evt.preventDefault()

  var form = evt.target
  validateForm(form)

  // ...
}
```

Now we need to have the form check if it is valid before submitting data to the server. The `form` element has a convenient `checkValidity` function to do just that.

```js

var submitHandler = function(evt) {
  evt.preventDefault()

  var form = evt.target
  validateForm(form)

  // if no error, go ahead and submit
  if (form.checkValidity()) {
    // ...
  }
}
```

### Loop through the fields

Every form has an elements array and we can use this to loop through the fields.

```js
var validateForm = function(form) {
  for (var f = 0; f < form.elements.length; f++) {

  }
}
```

Within the loop, get a reference to the current field.

```js
var validateForm = function(form) {
  for (var f = 0; f < form.elements.length; f++) {
    var field = form.elements[f]
  }
}
```

The behavior of the form hasn't changed so charge on to the next section.

## Check the field validity

We are going to start off using the native validation check for a field for our validations, first.

Let's create a new function called `isValid` that recieves a field as a parameter.

Like the form, fields have a `checkValidity` method. Use this method to determine the return value for the function.

```js
var isValid = function(field) {
  return field.checkValidity()
}
```

Now we can call this method for each of the fields we pass over in our loop.

```js
var validateForm = function(form) {
  for (var f = 0; f < form.elements.length; f++) {
    var field = form.elements[f]
    if(isValid(field)) {
      // remove error styles and messages
    } else {
      // style field, show error, etc.
    }
  }
}
```

### Set and clear error messages

Under each of the the input's in the `app/index.html` we are going to want to add a `<span></span>` element so that we have somewhere to output an error message. Let's give it a custom class of `error-message` so that we cans style it later.

```html
<span class='error-message'></span>
```

Let's make a function that sets the error message for a given field.

```js
var setError = function(field) { }
```

In order to get a reference to the `span` immediately after the field we want to display the error for, we will use the `nextElementSibling` method.

Once we have our `span` reference, all we need to do is set the `innerHTML` to be the field's `validationMessage`. The `validationMessage` is either the default message from the browser or a custom message that we set.

```js
var setError = function(field) {
  var error = field.nextElementSibling
  if (error) {
    error.innerHTML = field.validationMessage
  }
}
```

Let's also setup a similar function to clear our error message when the field becomes valid again.

```js
var clearError = function(field) {
  var error = field.nextElementSibling
  if (error) {
    error.innerHTML = ''
  }
}
```

ProTip™: This is just one way of handling error messages. For example, you could use CSS to show and hide error messages or put all validation errors in a central location on the page.

Now that we have created our `setError` and `clearError` methods, we can call them from our `validateForm` method.

```js
var validateForm = function(form) {
  for (var f = 0; f < form.elements.length; f++) {
    var field = form.elements[f]
    if(isValid(field)) {
      clearError(field)
    } else {
      setError(field)
    }
  }
}
```

When we go to the browser and try it out, we will now see our custom error message from the email input. But as we type in the email field and put in a valid email, the message does not disappear.

Let's add the `clearError` to our email keyup handler.

```js
var email = document.querySelector('[name="email"]')

var emailListener = function(evt) {
  var input = evt.currentTarget
  if (input.validity.typeMismatch) {
    input.setCustomValidity('Oops, try a real email address.')
  } else {
    input.setCustomValidity('')
    clearError(input)
  }
}
email.addEventListener('keyup', emailListener)
```

### Custom validation logic

Time for the bonus round! Custom validation logic could involve evaluating multiple fields on the form, making a call to the server, or performing some calculation in a worker thread. While those are beyond the time you have for this workshop, this section will walk you through creating some custom logic.

As an example, let's add a custom 'State' validation check. We will add this logic to the top of the `isValid` function from earlier.

First, we wnat to check if the field tha is passed in is in fact the `state` field, so verify the `name` matches `state`.

For this example, let's say that only "CA", "TX", and "NY" are valid states.

```js
 var sValid = function(field) {
  if (field.name === 'state') {
    var validStates = ['CA', 'TX', 'NY']
    if (validStates.indexOf(field.value) === -1) {
      // invalid state
    } else {
      // valid state
    }
  }
  return field.checkValidity()
}
```

When the state is invalid we want to set a custom validity message. When it is valid, we can set an empty string like before.

```js
var sValid = function(field) {
  if (validStates.indexOf(field.value) === -1) {
    field.setCustomValidity('Please provide a valid state (CA, TX, or NY)')
  } else {
    field.setCustomValidity('')
  }
  return field.checkValidity()
}
```

Now when we try to submit without a state or with an invalid state we should see our custom error message like before.

# Adding classes to style error message or boxes

# Welcome text with cookies

Cookies are a classic way to provide a unique visitor experience based on previous visits.  In our case we would like to show a generic welcome message the first time a user visits our site, but show an alternative, and more personalized message when a user returns for any additional visits from the same device.  Note that cookies are stored on the user’s device, so if they return from a new device or clear their cookies, we won’t have a cookie and will not know they are a returning visitor.  Cookies are often associated with tracking of user visits, be sure to read up on that before implementing on your own.

Here are the basics.   Cookies are stored as key value pairs as a strings.  The DOM API allows you to set cookies by assigning a value to document.cookie.  It must have a name, should have a value, and may have an expiration date for example:
```js
document.cookie = 'returning=true; expires=Mon, 1 Feb 2016 12:00:UTC;'
```
This would set a cookie with the key = 'returning' and value = 'true', as well as an expiration date.  This is when the cookie will go bye bye.

Getting a cookie involves more work because you have to parse the value of document.cookie (which remember is just a plain text string with ; separators).  It could include quite a few key value pairs, so most users will roll their own GET and SET methods to deal with parsing cookies.

Let’s add cookie set and get methods to help us handle our visitor’s session.
```js
function setCookie(cookieName, cookieValue, cookieDays) {
  var expireTime = new Date()
  expireTime.setTime(expireTime.getTime() + cookieDays * 24 * 60 * 60)
  var expires = "expires=" + expireTime.toUTCString()
  document.cookie = cookieName + "=" + cookieValue + "; " + expires
}

function getCookie(cookieName) {
  var name = cookieName + "="
  var cookieArray = document.cookie.split(';')
  for(var i=0; i < cookieArray.length; i++) {
    var cookieStr = cookieArray[i]
    while (cookieStr.charAt(0)==' ') {
      cookieStr = cookieStr.substring(1)
    }
    if (cookieStr.indexOf(name) == 0) {
      return cookieStr.substring(name.length,cookieStr.length)
    }
  }
  return "";
}
```

Then we will add a conditional block of code that will update the welcome message when a user returns, or set a cookie if its the first visit (or if the cookies have been cleared or expired).

```js
if(getCookie('returning') === 'yes') {
  document.querySelector('.greeting').innerHTML = "Welcome back to the program."
} else {
  setCookie('returning', 'yes', 2)
}
```

Then try refreshing the page and watch it add a custom greeting.  You can clear out your cookies using the Chrome Dev tools > Resources > Cookies

As a bonus try refactoring this code to cookie and display the users name in the greeting message when they return.

A couple of other fun facts about cookies: You can store objects in cookies, but you will need to be sure to serialize them and they must be strings when assigned to document.cookie.  Cookie security is handled for you:  Cookies are stored with a security feature tied to the domain that was used to store the cookie.  That means that your code running in the browser can only read the cookies that were set on the same domain.  Also note, cookies are stored as strings in plain text, so any savvy user can look at the cookies on their machine (delete, or modify them) as they see fit.

[validity-state]: https://developer.mozilla.org/en-US/docs/Web/API/ValidityState
