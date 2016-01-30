# Vanilla JavaScript in the Browser Workshop

## Overview

Welcome to Vanilla JavaScript in the Browser workshop hosted by [San Diego JS][san diego js].

The goal of this workshop is to learn how to effectively use vanilla JavaScript in the browser.

Modern web development workflows often rely on libraries like jQuery. But using libraries can add a lot of unneeded bloat to your project. This workshop will help you dig a little deeper into JavaScript and how to use the available browser APIs without using the $.

This workshop will feature:

* DOM API
* Browser Event API
* Using Cookies
* XHR Requests
* Dynamic HTML
* Testing with Mocha

### The example

For this workshop, we are going to build out a business card creator. There is a simple html form inside of `app/index.html` that we will build upon and add life to. We'll collect common profile information that one would normally share professionally, like contact info and skills, and then display it.

### Keep your resources handy

Workshops are fun and can be a fast way to learn while building something potentially reusable. There will come a time, however, when you'll want to reference the API and see if there are other methods or functionality that you didn't know existed.

A really good open-source resource that also provides great offline functionality for you future digital nomads is [DevDocs.io][devdocs]. You can select the different APIs you want to be immediately searchable and they cover everything from the browser, to node, to rails, to elixir, and even more you never even knew about!

For this workshop, it may be useful to reference the sections that will align with this workshop:

* [DOM API][dom]
* [Browser Event API][events]
* [Using Cookies][cookies]
* [XHR Requests][xhr]
* [Testing with Mocha][mocha]

Another good resource for learning about the available APIs and even the internals of how a browser parses, executes, and draws a page, is [Mozilla Developer Network][mdn].

**Have any other great resources you've used? We would love to hear about them and share them with the other attendees!**

## Pre-event Setup Instructions

Prior to your arrival the following should be installed on your system:

0. [Install Git][git-scm]
0. [Install Node.js 4.2 LTS][node-install]
0. Setup NPM for non-sudo installation
    0. NPM is the node package manager.  It will automatically be installed when you install node.
    0. NPM installs packages *locally* (within the directory it is invoked in) for per-project modules, or *globally* for packages you want accessible everywhere.
    0. However, by default NPM installs global packages in a root-restricted location, requiring SUDO to install.  This creates a **huge** headache.  As an alternative, _before_ you install any packages, follow [this guide][npm-g-without-sudo] to configure your NPM to install in your home directory without requiring sudo.
0. Clone the workshop repository:

        git clone git@github.com:sandiegojs/vanilla-browser-workshop.git

0. Change directories into the workshop folder and install your local dependencies with:

        cd vanilla-browser-workshop
        npm install

0. Install these global dependencies using the `-g` flag (ex `npm install <package> -g`)
    * gulp
    * mocha
0. If you plan to deploy your app onto Heroku, setup [Heroku Toolbelt][heroku-toolbelt]

## API service

The app we will be building is a client-side application, meaning it will be running in a user's browser. We will need to persist the data that we are
creating to a backend service and database so that we can recall it later.

We could use fixture data or a mock service for this, but some friendly backend
developers have already made a working API for us, so let's use that.

Our API is setup at [//sandiegojs-vanilla-workshop.herokuapp.com][sdjs-app] and supports the following endpoints


<table class="table table-bordered table-striped">
  <colgroup>
    <col class="col-xs-1">
    <col class="col-xs-3">
    <col class="col-xs-5">
  </colgroup>
  <thead>
    <tr>
      <th>Verb</th><th>path</th><th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GET</td><td>/forms</td><td>List of forms</td>
    </tr>
    <tr>
      <td>POST</td><td>/forms</td><td>Create a new form</td>
    </tr>
    <tr>
      <td>GET</td><td>/forms/:id</td><td>Retrieve a form</td>
    </tr>
    <tr>
      <td>PUT</td><td>/forms/:id</td><td>Update a form</td>
    </tr>
    <tr>
      <td>DELETE</td><td>/forms/:id</td><td>Delete a form</td>
    </tr>
  </tbody>
</table>


## Getting started

This project uses the build tool [Gulp][gulp] to automate a bunch of stuff for you. The gulp tasks are all defined in the `tasks` directory if you're the curious type. They are used to compile the sources and styles found within the `app` directory and put the final compiled source and styles into the `public` directory. The `public/index.html` file will include the final styles and scripts inside of it.

Everything in the `public/` directory of this application is created by Gulp. Think of this as our staging area. We should never be messing with the files in this directory directly. Instead, we should be editing and changing the origin files found in the `app/` directory and running Gulp in the terminal to bundle our project.

In order to get started, this boilerplate has been setup to show us a simple form right from the start. So let's dive in and give the gulp step a try!

Type the following from the root of the project:

```bash
$ gulp
```

You should see some output similar to the following:

```bash
$ gulp
[16:02:13] Requiring external module babel-core/register
[16:02:14] Using gulpfile ~/Code/vanilla-browser-workshop/gulpfile.babel.js
[16:02:14] Starting 'scripts'...
[16:02:14] Starting 'styles'...
[16:02:14] Starting 'serve'...
[16:02:14] Finished 'serve' after 46 ms
livereload[tiny-lr] listening on 35729 ...
[16:02:15] Finished 'styles' after 175 ms
[16:02:15] Finished 'scripts' after 355 ms
[16:02:15] Starting 'default'...
[16:02:15] Finished 'default' after 114 μs
folder "public" serving at http://localhost:3000
```

As you can see, the scripts and styles are compiled, and then the server gets started. Now that we have that running, let's jump over to the browser and visit [localhost:3000][localhost].

![image of boilerplate running](https://s3.amazonaws.com/f.cl.ly/items/1R3O1Q39443m1B2d3k23/Screen%20Shot%202016-01-17%20at%204.03.41%20PM.png?v=bd9646af)

Cool! We have a very simple form with some basic styling already good to go for us.

**If you're having issues bug one of the instructors! We're here to help you out.**

[cookies]: https://devdocs.io/dom/document/cookie
[devdocs]: http://devdocs.io
[dom]: https://devdocs.io/dom/
[events]: https://devdocs.io/dom_events
[git-scm]: http://git-scm.com/downloads
[heroku-toolbelt]: https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up
[localhost]: http://localhost:3000
[mdn]: https://developer.mozilla.org/en-US/
[mocha]: https://devdocs.io/mocha/
[node-install]: https://nodejs.org/download/
[npm-g-without-sudo]: https://github.com/sindresorhus/guides/blob/master/npm-global-without-sudo.md
[san diego js]: http://sandiegojs.org/
[sdjs-app]: //sandiegojs-vanilla-workshop.herokuapp.com
[xhr]: https://devdocs.io/dom/xmlhttprequest
[gulp]: http://gulpjs.com/
