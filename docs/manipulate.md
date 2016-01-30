# Add and remove skills

Now that we have learned about the DOM and used the console inside of our browser to get the hang of event handlers, let's create a new JavaScript file and start building our app.

You might remember that our build tool, Gulp, will compile any `.js` files that we put in the `app` directory into a single `index.js` file and Gulp outputs this file in the `public` directory.  You should leave all the files in `public` alone and let the build tool manage creating these files.

There is also a script tag in the `index.html` file to load all of your scripts, our Gulp script will create a copy of the `index.html` in the public folder, which should also not be worked on directly.  You will will be updating the files in the `app` folder.  Let's begin by creating a new file `app.js` inside of the `app` directory.

With that in place, we can now begin to add more functionality to our form. We have more than just a single skill to boast about, so we want to be able to add multiple skills. We may also change our mind while we're filling out the form so we will have to be able to remove a skill, also.

## Adding a skill

You probably noticed the nice `+` button next to the `skills` input. Right now it doesn't do anything. We need to add a `click` handler in order to bring it to life.

In order to do that, we need to first select the `.add-skill` button which we will attach the handler to and then we need to also have a "blueprint" of the original div containing the skill elements so that we can copy it when we add another one.

```js
var addSkillButton = document.querySelector('.add-skill')
var skillTemplate = document.querySelector('.skill')

function addSkillHandler(evt) {
  alert('adding skill')
}
addSkillButton.addEventListener('click', addSkillHandler)
```

If you give this a try in your browser now, you'll see the alert pop up when you click on the plus.

What does "add skill" really mean? We want to duplicate the skill blueprint we have, `skillTemplate` and create another node just like it. Then, we want to append it at the end of the list of skills.

We can clone any DOM node with the `cloneNode` method. The `cloneNode` method takes an optional boolean argument to determine whether it should be a deep or shallow clone. Since we want the entire `input-group` element and all of it's children nodes, we are going to pass a `true` in.

Unfortunately, there is no `appendAfter` method we can use, but there is an [`insertBefore`][insert-before] method. We can use `insertBefore` to append the new cloned node just before the submit button.

The `insertBefore` method uses a handle on the parent node to attach the new node in the correct spot within the tree. We can use the `parentNode` accesor on any DOM node to get it's parent, or, since we know the parent is the form we can just directly select it again.

```js
var addSkillButton = document.querySelector('.add-skill')
var skillTemplate = document.querySelector('.skill').cloneNode(true)

function addSkillHandler(evt) {
  var submitNode = document.querySelector('.submit')
  var form = submitNode.parentNode
  var newSkill = skillTemplate.cloneNode(true)
  form.insertBefore(newSkill, submitNode)
}
addSkillButton.addEventListener('click', addSkillHandler)
```

We just grew a branch on our DOM tree 🌳! Does it look right, though?

![added skill](https://s3.amazonaws.com/f.cl.ly/items/113x3C3q1H133y3b3C1e/Screen%20Shot%202016-01-24%20at%2010.14.00%20AM.png?v=ad58e105)

After we clone a new node, we need to change the plus sign to a minus sign on the previous one. If you look at the `app/index.html` file you'll see that both the plus and minus icons already exist in the DOM, except that the minus is currently hidden with the class `hidden`.

It would be useful to be able to select the last element in a group. Let's write a helper method that does just that for a given selector.

```js
function last(selector) {
  var all = document.querySelectorAll(selector)
  var length = all.length
  return all[length - 1]
}
```

Now let's update our `addSkillHandler` method to get a handle on the previous skill, update it to show the minus instead of the plus, and _then_ add the new skill to the end of the list.

When we have a handle on the DOM node as we will with the previous skill, we can get a list of the node's classes by using the [`classList` accessor][class-list]. We can then use `.add()` and `.remove()` to add and remove classes from this node.


```js
var addSkillButton = document.querySelector('.add-skill')
var skillTemplate = document.querySelector('.skill').cloneNode(true)

function addSkillHandler(evt) {
  var prevSkill = last('.skill')
  var newSkill = skillTemplate.cloneNode(true)
  var submitNode = document.querySelector('.submit')
  var form = submitNode.parentNode

  prevSkill.querySelector('.add-skill').classList.add('hidden')
  prevSkill.querySelector('.remove-skill').classList.remove('hidden')

  form.insertBefore(newSkill, submitNode)
}
addSkillButton.addEventListener('click', addSkillHandler)
```

If you head over to the browser, it should work! Sweet DOM manipulation.

Did you try to click the second plus button like a smarty pants? Then you realize that it didn't work, right?

Fix it by attaching the handler to the new element as well.


```js
var addSkillButton = document.querySelector('.add-skill')
var skillTemplate = document.querySelector('.skill').cloneNode(true)

function addSkillHandler(evt) {
  var prevSkill = last('.skill')
  var newSkill = skillTemplate.cloneNode(true)
  var submitNode = document.querySelector('.submit')
  var form = submitNode.parentNode

  prevSkill.querySelector('.add-skill').classList.add('hidden')
  prevSkill.querySelector('.remove-skill').classList.remove('hidden')

  newSkill.querySelector('.add-skill').addEventListener('click', addSkillHandler)

  form.insertBefore(newSkill, submitNode)
}
addSkillButton.addEventListener('click', addSkillHandler)
```

Great, now you're ready to remove some skills!

## Removing a skill

Much like we created an event handler for adding a skill, we will now create one to remove a skill.

```js
var removeSkillHandler = function(evt) {}
```

When the minus button is clicked, you'll remember from the event section that the minus button element becomes the `currentTarget` on the event object passed in as the first argument to our handler.

If we have a handle on the minus button, then we can select the `parentNode` to get the group of elements that makes up a single skill. And if we can do that, then we can call [`remove()`][remove-node] do remove the node from the DOM.

```js
var removeSkillButton = document.querySelector('.remove-skill')

function removeSkillHandler(evt) {
  var skill = evt.currentTarget.parentNode
  skill.remove()
}

removeSkillButton.addEventListener('click', removeSkillHandler)
```

We will also need to bind the `removeSkillHandler` to our new `.remove-skill` elements.

```js
var addSkillButton = document.querySelector('.add-skill')
var skillTemplate = document.querySelector('.skill').cloneNode(true)

function addSkillHandler(evt) {
  var prevSkill = last('.skill')
  var newSkill = skillTemplate.cloneNode(true)
  var submitNode = document.querySelector('.submit')
  var form = submitNode.parentNode

  prevSkill.querySelector('.add-skill').classList.add('hidden')
  prevSkill.querySelector('.remove-skill').classList.remove('hidden')

  newSkill.querySelector('.add-skill').addEventListener('click', addSkillHandler)
  newSkill.querySelector('.remove-skill').addEventListener('click', removeSkillHandler)

  form.insertBefore(newSkill, submitNode)
}
addSkillButton.addEventListener('click', addSkillHandler)
```

[class-list]: https://developer.mozilla.org/en-US/docs/Web/API/Element/classList
[insert-before]: https://developer.mozilla.org/en-US/docs/Web/API/Node/insertBefore
[remove-node]: https://developer.mozilla.org/en-US/docs/Web/API/ChildNode/remove
