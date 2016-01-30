# Add submit event

Now that we've covered the basics of getting a DOM element and hooking into its events, let's add a submit event handler to the form. Later, we'll use this handler to kick off a request to the backend.

Our first step in the `app.js` file we created previously is to get a reference to the form element. Create a variable named `form`, and assign the form DOM element on the page to it.

```js
var form = document.querySelector('form')
```

On the next line create a `submitHandler` function. We will need to have the event passed into the handler available to us so add the parameter `evt` to the function declaration.

```js
var submitHandler = function(evt) {}
```

We will be using the `preventDefault` method on the event object. This handy method will stop the form from submitting to the backend so that we can submit this information using AJAX.

Let's also throw in an `alert` so that we can see some feedback from the submit handler. Otherwise, when we click on the submit button nothing will happen which can be confusing. Add the two new commands to the submit handler declaration like below.

```js
var submitHandler = function(evt) {
  evt.preventDefault()
  alert('submit!')
}
```

In order to hook this handler up to our form element, we will need to use the `addEventListener` method that exists on the form node. Pass in `submit` as the event we want to listen for and pass in the custom handler we just wrote.

```js
form.addEventListener('submit', submitHandler)
```

Now if you refresh the page you should be able to submit the form after filling out the two required fields, but instead of the page refreshing and losing everything you've entered the page will show you an alert and you won't lose any of the information you typed.

Our event handler is now working!

# Serialize the form data

Before we can begin to assemble the pieces for talking to our server, we will need to make sure that we can capture the data from our form in a format that our API endpoint can digest. The API for this workshop is built with [`rails-api`][rails-api] which will be expecting a JSON object on the request.

We need to take the form data and convert it to an object that looks like this

```json
{
  "form": {
    "name": "",
    "email": "",
    "city": "",
    "state": "",
    "github": "",
    "twitter": "",
    "bio": "",
    "skills_attributes": []
  }
}
```

Let's write a `serializeArray` method to take our form data and turn it into the serialized JSON data object.

```js
var serializeArray = function(selector) {}
```

We want to be able to pass in our selector for the form to this method and get the JSON object back. First, let's use `document.querySelector` again to grab the form.

```js
var serializeArray = function(selector) {
  var form = document.querySelector(selector)
}
```

**ProTip™:**  In our example, we know we are passing in a form selector but if you wanted to use this method in a different scenario, you might want to check that the element we have selected is actually a form by using `form.tagName` to check the element's tag name.

Now we need to grab all the inputs from inside the form element. Remember the `querySelectorAll` method mentioned above? While `querySelector` returns a single Node in the DOM, `querySelectorAll` will return a [`NodeList`][node-list].

The unusual thing about this is that `NodeList` is not a JavaScript `Array` and so we cannot use methods like `map` or `forEach` to iterate over the items returned. We could do some fancy work and add these methods to the prototype for `NodeList`, but to keep things simple we will just use a basic `for` loop to iterate over them.

```js
var serializeArray = function(selector) {
  var form = document.querySelector(selector)
  var formInputs = form.querySelectorAll('input,textarea')

  for (var i = 0; i < formInputs.length; i++) {
    var item = formInputs[i]
  }
}
```

Now that we can iterate over them we will need to store the `name` and `value` of each item onto a JSON object as the `key`,`value` pair. When we are accessing the `Node` directly, we can easily call `node.name` or `node.value` to access these attributes.

We can't test our method unless we call our `serializeArray` method from somewhere, so let's add the call into our `submitHandler` method.

```js
var submitHandler = function(evt) {
  evt.preventDefault()
  var data = serializeArray('form')
}
```

Ok, now let's build our final `data` JSON object. Let's also log it to the console so that we can inspect it make sure it matches our expected output that we defined above.

```js
var serializeArray = function(selector) {
  var form = document.querySelector(selector)
  var formInputs = form.querySelectorAll('input,textarea')

  // Empty object for us to set key values of inputs
  var data = {}

  for (var i = 0; i < formInputs.length; i++) {
    var item = formInputs[i]
    data[item.name] = item.value
  }

  // Log out our final object so we can inspect it
  console.log(data)
}
```

When you reload the page and submit the form again, you should see something similar to this outputted in the console.

![build the form JSON](https://s3.amazonaws.com/f.cl.ly/items/3V3p121o1C093I0B0F1s/6aUSiYmFDT.gif?v=bb86545c)

Well that looks _nearly_ correct. Can you spot the problems?

Firstly, we're capturing the submit button. There are a couple ways to avoid this.

We could change the submit button from an `input` to a `div` and change our handler from the `form` `submit` event to the `div` `onclick` event. If we were to do this, we would lose the native validations, so let's nix this idea.

We could instead just change our `querySelectorAll(...)` selector string to ignore the `type=submit` input. That seems simple enough, so let's add that in.

```js
var serializeArray = function(selector) {
  var form = document.querySelector(selector)
  var formInputs = form.querySelectorAll('input:not([type=submit]),textarea')

  // Empty object for us to set key values of inputs
  var data = {}

  for (var i = 0; i < formInputs.length; i++) {
    var item = formInputs[i]
    data[item.name] = item.value
  }

  // Log out our final object so we can inspect it
  console.log(data)
}
```

Ok, let's try this again!

![build the form JSON part 2](https://s3.amazonaws.com/f.cl.ly/items/2N0C470x010K2k302e3Q/RRBJpcGUMt.gif?v=94639764)

Something is still off. Did you spot it?

We need our `skills_attributes` key to have an array value since we have multiple skills. Right now, only the very last skill is being saved. We want to catch this special case inside of our `for` loop. If we haven't created this new `skills_attributes` array, do so and append the current skill, and if we have then just append the current skill.

```js
var serializeArray = function(selector) {
  var form = document.querySelector(selector)
  var formInputs = form.querySelectorAll('input:not([type=submit]),textarea')

  // Empty object for us to set key values of inputs
  var data = {}

  for (var i = 0; i < formInputs.length; i++) {
    var item = formInputs[i]

    if (item.name === 'skills_attributes') {
      if (!!data[item.name]) {
        data[item.name].push(item.value)
      } else {
        data[item.name] = [item.value]
      }
    } else {
      data[item.name] = item.value
    }
  }

  // Log out our final object so we can inspect it
  console.log(data)
}
```

Head back over to the console, and you'll see our data now looks correct! Our last step is to remove the `console.log` and replace it with a return statement that returns an object with a `form` property set to our data, since this is the structure our API endpoint is expecting.

```js
var serializeArray = function(selector) {
  var form = document.querySelector(selector)
  var formInputs = form.querySelectorAll('input:not([type=submit]),textarea')

  // Empty object for us to set key values of inputs
  var data = {}

  for (var i = 0; i < formInputs.length; i++) {
    var item = formInputs[i]

    if (item.name === 'skills_attributes') {
      if (!!data[item.name]) {
        data[item.name].push(item.value)
      } else {
        data[item.name] = [item.value]
      }
    } else {
      data[item.name] = item.value
    }
  }

  return {
    "form": data
  }
}
```

# Build XHR and submit

Now that we have our data we need to send it to the server using an XHR or [XMLHttpRequest][xhr-mdn]. This allows us to communicate with the server without changing pages, and then do some action based on the data we get back. We are going to create a function that we can put in our event handlers that will do this.

```js
var xhr = function(method, path, data, callback) {}
```

First, we create a new instance of XHR.

```js
var xhr = function(method, path, data, callback) {
  var request = new XMLHttpRequest()
}
```

Next we'll use `open(method, path, async)` to initialize the request.

- **method:** a string of an HTTP method to use, such as 'GET', 'POST', 'PUT', 'DELETE'. This will match the Verb on the API table up top.
- **path:** a string of the full path to send the request to. This will include the API endpoint URL as well as the path.
- **async:** a boolean flag that dictates whether the script should run asynchronously.

**ProTip™:** `async` should always be `true` to prevent blocking. Stopping JavaScript execution especially hurts time sensitive things like rendering or event listening/handling.

```js
var xhr = function(method, path, data, callback) {
  var request = new XMLHttpRequest()
  request.open(method, path, true)
}
```

Because we'll be using a JSON endpoint any requests we make to it should contain a `Content-Type` header.

```js
var xhr = function(method, path, data, callback) {
  var request = new XMLHttpRequest()
  request.open(method, path, true)
  request.setRequestHeader('Content-Type', 'application/json')
}
```

XHR only has one event we care to listen to and that's [onreadystatechange][mdn-onreadystatechange].

The ready state of the XHR will change a few times, but we are looking for the last state of `4` which is triggered when the request operation is complete.

We'll also check to make sure we got a good server response. If it passes those checks, we'll parse the response JSON, then send the object back though a callback.

```js
var xhr = function(method, path, data, callback) {
  var request = new XMLHttpRequest()
  request.open(method, path, true)
  request.setRequestHeader('Content-Type', 'application/json')
  request.onreadystatechange = function() {
    // ignore anything that isn't the last state
    if (request.readyState !== 4) { return }

    // if we didn't get a "good" status such as 200 OK or 201 Created send back an error
    if (request.readyState === 4 && (request.status !== 200 && request.status !== 201)) {
      callback(new Error('XHR Failed: ' + path), null)
    }

    // return our server data
    callback(null, JSON.parse(request.responseText))
  }
}
```

Lastly we will use `JSON.stringify` to convert our object to a JSON string and send the request with our data using the `send` function.

```js
var xhr = function(method, path, data, callback) {
  var request = new XMLHttpRequest()
  request.open(method, path, true)
  request.setRequestHeader('Content-Type', 'application/json')
  request.onreadystatechange = function() {
    // ignore anything that isn't the last state
    if (request.readyState !== 4) { return }

    // if we didn't get a "good" status such as 200 OK or 201 Created send back an error
    if (request.readyState === 4 && (request.status !== 200 && request.status !== 201)) {
      callback(new Error('XHR Failed: ' + path), null)
    }

    // return our server data
    callback(null, JSON.parse(request.responseText))
  }
  request.send(JSON.stringify(data))
}
```

Now we're ready to use it inside of our `submitHandler`. We can call it with the 'POST' method and pass it the form data.

```js
var apiURL = '//sandiegojs-vanilla-workshop.herokuapp.com'

var submitHandler = function(evt) {
  evt.preventDefault()
  var path = apiURL + '/forms'
  xhr('POST', path, serializeArray('form'), function(err, data) {
    if (err) { throw err }
    console.log(data)
  })
}
```

# Handle server response

Congratulations! We can now submit the form to the server and get a response. Now let's do something with the response.

We're going to write any errors from our XHR request to the screen. We're also going to render the contents of the form to the screen if it was successfully submitted.

Some of the methods we'll be using are:

- [Document.createElement][mdn-createelement] this method creates a DOM element of the type specified in the first parameter. For example, we can pass in `div`, `p`, `ul`, etc.
- [Document.createTextNode][mdn-createtextnode] this method creates a text node that can be placed in the document. We'll use this to hold content we author.
- [Node.appendChild][mdn-appendchild] this method can be called on any element created using createElement. As you can guess from the name it adds the node passed in to the end of node you call appendChild on.

Let's create a small helper function. It creates a DOM node and adds some text to it. We're going to be making a lot calls to this.

```js
var createElementWithTextNode = function(tagName, tagContent) {}
```

Right inside of that function declaration we should add a call to `document.createElement`.

```js
var createElementWithTextNode = function(tagName, tagContent) {
  var node = document.createElement(tagName)
}
```

Next up we will create a text node that will hold whatever is inside of the tagContent variable.

```js
var createElementWithTextNode = function(tagName, tagContent) {
  var node = document.createElement(tagName)
  var textNode = document.createTextNode(tagContent)
}
```

Then we combine the two freshly created nodes. You can add other HTML nodes or text nodes to a node even if the HTML node would usually have children, such as `<br>`.

```js
var createElementWithTextNode = function(tagName, tagContent) {
  var node = document.createElement(tagName)
  var textNode = document.createTextNode(tagContent)
  node.appendChild(textNode)
}
```

There is a possibility that when we call this function we won't have any textContent, and calling `document.createTextNode` without textContent will cause an error. To avoid this kind of error wrap the creation and insertion of the textNode in a conditional test for textContent.

```js
var createElementWithTextNode = function(tagName, tagContent) {
  var node = document.createElement(tagName)
  if (tagContent) {
    var textNode = document.createTextNode(tagContent)
    node.appendChild(textNode)
  }
}
```

And lastly we will return the node we created with a child text node.

```js
var createElementWithTextNode = function(tagName, tagContent) {
  var node = document.createElement(tagName)
  if (tagContent) {
    var textNode = document.createTextNode(tagContent)
    node.appendChild(textNode)
  }
  return node
}
```

Now that we have that built, we can start to output some content. First create the renderError function

```js
var renderError = function(error) {}
```

The error render will be very simple. We are going to get a reference to the response container, create a node with our helper function, add a custom class to it, and append it to the document.

```js
var renderError = function(error) {
  var responseNode = document.querySelector('.response-wrapper')
  var errorNode = createElementWithTextNode('div', error.toString())
  errorNode.className = 'error'
  responseNode.appendChild(errorNode)
}
```

Now it's time to create the renderFormData function. This function is going to create many DOM nodes and append them to the response container.

```js
var renderFormData = function(data) {}
```

The first thing we want to do inside of the renderFormData function is get a reference to the response container just like in our `renderError` function.

```js
var renderFormData = function(data) {
  var responseNode = document.querySelector('.response-wrapper')
}
```

Now, let's create a generic success message so we can see on screen when the form was submitted and the backend saved it.

```js
var renderFormData = function(data) {
  var responseNode = document.querySelector('.response-wrapper')

  //generic success message
  var successNode = createElementWithTextNode('div', 'You\'ve add a new card')
  successNode.className = 'success'
  responseNode.appendChild(successNode)
}
```

Great, now we can give some kind of feedback to the user when the form is successfully processed by backend or not. Let's improve upon this, though. Let's show the information we entered when we submitted so we can verify it saved properly.

To do this we'll create a dictionary list, and fill it with all the some of the information that is returned by the request.

Let's use the `document.createElement` method to create a DL node that will hold all those terms and definitions. Append this to the `renderFormData` function.

```js
var dictionaryNode = document.createElement('dl')
```

The next step is to add some terms and definitions. Let's use the `createElementWithTextNode` helper function to streamline the process of printing out the name. Add the next couple of lines after the `dictionaryNode` variable declaration and assignment.

```js
//create a dom node with the name of a value
var termNode = createElementWithTextNode('dt', 'name')
dictionaryNode.appendChild(termNode)

//create another dom node with the value
var definitionNode = createElementWithTextNode('dd', data.name)
dictionaryNode.appendChild(definitionNode)
```

Now, after the success message we're printing out the name we entered on the form. This response would be more useful if we showed more information. It would be good to show the name, email, and some other attributes. The simplest way to do this is to just copy and paste the code we just wrote and swap out name for email, github, etc.

But, there's a better way. If we create a list of keys we'd like to show on screen, we can loop through them with the native [`forEach`][mdn-foreach] method of an array. This method will iterate over an array and call a passed in callback function for each item in the array. The first parameter of the callback will be the current item that forEach is iterating through.

Create an array called keys. Place it just after the insertion of the success message, but keep it before the last set of statements we wrote.

```js
var keys = ['name', 'email', 'github', 'twitter', 'city', 'state', 'bio']
```

Next call the `forEach` method on it and pass in an **anonymous function** with a single parameter of `key`.

```js
keys.forEach(function(key) {})
```

Move the name definition and insertion we wrote earlier into the callback. Change the text passed into `createElementWithTextNode`. For the first call replace the string literal, `'name'` with the variable `key`, and for the second call replace `data.name` with `data[key]`. This will allow each iteration to print the current key, such as name or email, as the dictionary term and the actual value, such as "John Doe", will be the dictionary definition.

```js
keys.forEach(function(key) {
  //create a dom node with the name of a value
  var termNode = createElementWithTextNode('dt', key)
  dictionaryNode.appendChild(termNode)

  //create another dom node with the value
  var definitionNode = createElementWithTextNode('dd', data[key])
  dictionaryNode.appendChild(definitionNode)
})
```

If you review the array of keys we're looping through, you'll notice we didn't include `skills_attributes` like we did when we posted the data. That's because the content of that property is more complex than the simple strings of the other properties. If we want to print them out we'll have to write some custom logic.

Create a new dictionary term node like we do in the `forEach` callback, but set the text content to "skills".

```js
var skillsTermNode = createElementWithTextNode('dt', 'skills')
dictionaryNode.appendChild(skillsTermNode)
```

Let's add the dictionary definition, now. We'll add the actual skills information in the next step. We'll show all the skills as a list, so let's also add a **UL** node as well and store a reference to it in a variable so we can add **LI** elements to it later.

```js
var skillsDefinitionNode = document.createElement('dd')
var skillsList = document.createElement('ul')
skillsDefinitionNode.appendChild(skillsList)
dictionaryNode.appendChild(skillsDefinitionNode)
```

Now that that's out of the way, let's loop through each skill we received, if any, and combine them into one dictionary definition. To accomplish this we'll test to see if there is any value stored in `skills_attributes`. If there is then we can loop through all the skills by using the `forEach` method to loop through each complex element in the array and append a simpler set of information to the dictionary. In our case, we'll append only the description of each skill wrapped in a **LI** tag.

```js
if (data.skills_attributes) {
  data.skills_attributes.forEach(function (skill) {
    var skillNode = createElementWithTextNode('li', skill.description)
    skillsList.appendChild(skillNode)
  })
}
```

The last steps are to add a custom class of `response` and append the dictionaryNode we've been populating to the responseNode from the beginning of the function. Here is the entire `renderFormData` function:

```js
var renderFormData = function(data) {
  var responseNode = document.querySelector('.response-wrapper')

  //generic success message
  var successNode = createElementWithTextNode('div', 'You\'ve add a new card')
  successNode.className = 'success'
  responseNode.appendChild(successNode)

  var dictionaryNode = document.createElement('dl')
  var keys = ['name', 'email', 'github', 'twitter', 'city', 'state', 'bio']
  keys.forEach(function(key) {
    //create a dom node with the name of a value
    var termNode = createElementWithTextNode('dt', key)
    dictionaryNode.appendChild(termNode)

    //create another dom node with the value
    var definitionNode = createElementWithTextNode('dd', data[key])
    dictionaryNode.appendChild(definitionNode)
  })

  var skillsTermNode = createElementWithTextNode('dt', 'skills')
  dictionaryNode.appendChild(skillsTermNode)

  var skillsDefinitionNode = document.createElement('dd')
  var skillsList = document.createElement('ul')
  skillsDefinitionNode.appendChild(skillsList)
  dictionaryNode.appendChild(skillsDefinitionNode)

  if (data.skills_attributes) {
    data.skills_attributes.forEach(function (skill) {
      var skillNode = createElementWithTextNode('li', skill.description)
      skillsList.appendChild(skillNode)
    })
  }

  dictionaryNode.className = 'response'
  responseNode.appendChild(dictionaryNode)
}
```

We've finished creating all the functions we need to process the response from our backend. Let's add renderError & renderFormData in the appropriate places in the original submitHandler. In addition, let's reset the form after we know it worked.

```js
var submitHandler = function(evt) {
  evt.preventDefault()
  var path = apiURL + '/forms'
  xhr('POST', path, serializeArray('form'), function(err, data) {
    if (err) {
      renderError(err)
      throw err
    }
    console.log(data)
    renderFormData(data)
    document.querySelector('form').reset()
  })
}
```

If you fill out the form, you should see a response similar to what was described in the **Serialize the form data** section in your console. For example:

```json
{
  "id": 1,
  "name": "Testy McTesterson",
  "email": "test@sandiegojs.org",
  "city": "San Diego",
  "state": "CA",
  "github": "testdev",
  "twitter": "testtwitter",
  "bio": "Lorem Ipsum...",
  "skills_attributes": [
    {
      "id": 1,
      "description": "cooking",
      "form_id": 1,
      "created_at": "2016-01-21T07:09:01.640Z",
      "updated_at": "2016-01-21T07:09:01.640Z"
    },
    {
      "id": 2,
      "description": "cleaning",
      "form_id": 1,
      "created_at": "2016-01-21T07:09:01.640Z",
      "updated_at": "2016-01-21T07:09:01.640Z"
    }
  ],
  "created_at": "2016-01-21T07:09:01.640Z",
  "updated_at": "2016-01-21T07:09:01.640Z"
}
```

On screen you should see the this:

![Success screenshot](https://cloud.githubusercontent.com/assets/208819/12669678/48152504-c617-11e5-9f0c-48a768596811.png)

Congratulations! Now we have a fully submitting form that we can use to save people's information.

[mdn-appendchild]: https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild
[mdn-createelement]: https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement
[mdn-createtextnode]: https://developer.mozilla.org/en-US/docs/Web/API/Document/createTextNode
[mdn-foreach]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
[mdn-onreadystatechange]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/onreadystatechange
[node-list]: https://developer.mozilla.org/en-US/docs/Web/API/NodeList
[rails-api]: https://github.com/rails-api/rails-api
[xhr-mdn]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest
