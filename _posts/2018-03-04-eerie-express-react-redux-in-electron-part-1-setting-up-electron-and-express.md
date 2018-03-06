---
layout: post
title: "!EERiE: Part 1: Setting up Electron and Express"
description: "Part one of a multi-part series on using Electron, Express, React and Redux together."
comments: true
---

**Not EERiE** is a series that demonstrates that *EERiE* (**E**lementary [**E**xpress + **R**eact **i**n **E**lectron]) is not really *eerie* and pretty easy to approach.

> # In the series:
>
> **Part 1**: Setting up Electron and Express **(You are here)**
>
> **Part 2**: [Creating an API with Express, SQLite and Sequelize](/blog/2018/03/eerie-express-react-redux-in-electron-part-2-creating-api-with-express-sqlite-and-sequelize/)

---

There are lots of tutorials about using React, Express, Electron with lots of other *starer-kit* repositories but I'd like to take another shot at it.

In this series, we will do this:

- Create a simple REST API with [Express](https://expressjs.com/)
  - We will use [SQLite](https://www.sqlite.org/) with [Sequelize](http://docs.sequelizejs.com/)
  - Use [Postman](https://www.getpostman.com/) to test the API

- Create a simple frontend that consumes the API with [React](https://reactjs.org/)
  - We will use [Redux](https://redux.js.org/) to manage the state of our app
  - [AntD](https://ant.design/) as our UI library so everything looks neat

- We'll set it all up so that the Electron app on launch will show our React Frontend.

We will create an invoicing app in the end. However if you just want to see how everything is put together, read the first three parts of the series, I'll create a very basic app using everything I mentioned above (except AntD). After that I'll start with the Invoicing App. 

# Prerequisites

- Node.js installed (I have `v8.9.4` installed, that's the LTS at the time of writing this)

- Basics of everything we're going to use.
  - This is not a React/Express/Electron tutorial, I am not going to go in much detail. At the end though we'll have built a fairly complex application.

Let's begin.

We'll first create a project directory and run `npm init`.

```
$ npm init
```

Start by installing Electron. I will put a link to my github repository of this project if you want to check the `package.json`.

```
$ npm i --save electron
```

We'll keep the electron related code in a file called **_app/electron.js_**.

```
$ mkdir app && touch app/electron.js
```

This 'app' directory will have our React, Express and Electron code. It is just so we can keep all the config files separate from our code.

The **_electron.js_** will have the code to start an Electron app and create a window and load `google.com`.

Here's how **_electron.js_** looks like:

```javascript
const {app, BrowserWindow} = require('electron');

let win;

function createWindow () {
  win = new BrowserWindow({width: 800, height: 600});
  win.loadURL('https://google.com/');
  win.on('closed', () => {
    app.quit();
  });
}

app.on('ready', createWindow);
```

What it does is pretty simple, and is very similar to the [Electron Quick Start](https://electronjs.org/docs/tutorial/quick-start) (I highly recommend you check it out, especially [Writing Your First Electron App](https://electronjs.org/docs/tutorial/first-app) which has a very nicely commented example.

On **line 13**, we're listening for an event from the Electron application, 'ready' is emitted when Electron has finished initialization and in that case we're calling a function called `createWindow`.

As the name suggests, **from line 5 to 11**, `createWindow` does exactly that, on **line 6** it creates a new `BrowserWindow` with passing width and height in an `options` object (You can find every option available here: [BrowserWindow Options](https://electronjs.org/docs/api/browser-window#new-browserwindowoptions))

**Line 7** then loads google.com in the newly created `BrowserWindow` and **line 8** listens for 'closed' event and in that case quits the app. 

We're using a single window, if we had multiple windows, we'd keep them in an array and delete the respective elements on each closed event, or if we had multiple variables containing different events, we'd set them to null.

In that case, there's another event that the app emits called 'window-all-closed', you can listen to that and quit your app.

I recommend you check out [Guides](https://electronjs.org/docs/tutorial) on Electron's website, although I haven't even gone through all of it (yet), I suggest you at least read this: [Electron Quick Start](https://electronjs.org/docs/tutorial/quick-start).

Now we have a simple electron app that creates a single `BrowserWindow` and loads google.com. 

One last thing that we need to do is add a start script in our **_package.json_** and change the entry point (main). For 'main', replace whatever the initial value is with path to our **_electron.js_** file. (**_'app/electron.js'_** in this case).

Now in scripts, create another key value pair besides the 'test' script, key will be 'start' and the value will be 'electron .'

Here's how **_package.json_** looks like:

```json
{
  "name": "alchemy",
  "version": "1.0.0",
  "description": "A sample project for a series on using Express, React & Redux in Electron",
  "main": "app/electron.js",
  "scripts": {
    "start": "electron .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "Electron",
    "Express",
    "React",
    "Redux",
    "SQLite",
    "Sequelize"
  ],
  "author": "sacredSatan",
  "license": "MIT",
  "dependencies": {
    "electron": "^1.8.2"
  }
}
```

Start the app with `npm start` to see if everything worked. You should see an Electron window that loads google.com.

That's it for our initial Electron setup. Let's setup Express.

```
$ npm i --save express
```

We'll keep our API related code in a directory called 'api' inside of the 'app' directory. It will mostly have Express + Sequelize related code.

Inside the 'api' directory, a file called **_server.js_** will have the Express app related code.

Here's how **_server.js_** looks like:

```javascript
const express = require('express');

const app = express();
app.get('/', (req, res) => {
    res.send('It is working!');
});

app.listen(5000, () => console.log('App running on port 5000'));
```

It is pretty simple, we're listening to any GET requests made on '/' path (**line 4**) and sending a response that says 'It is working!'. At the end we're listening for any requests made to port 5000.

Here's how the directory structure looks like:

```
$ tree -I node_modules
.
├── app
│   ├── api
│   │   └── app.js
│   └── electron.js
├── package.json
└── package-lock.json

2 directories, 4 files
```

Now let's modify our **_electron.js_** so that our Express server starts with Electron and Electron window loads 'localhost:5000/' instead of google.com.

First we'll import our **_api/server.js_** so our server code gets executed, you can simply do it like this:

```javascript
require('./api/server');
```

And then in function `createWindow`, change the url to 'http://localhost:5000'.

We're done here, if you start the app again (`npm start`), it should load a webpage that says 'It is working!'.

I want to end this post here, but before that, one thing that we can still do is use [Babel](https://babeljs.io/) to transpile our code. If you try to use an `import` statement, it won't be recognized and will throw a `SyntaxError`. The easiest way we can use it is by using [babel-register](https://babeljs.io/docs/usage/babel-register/).

Replace all the `require()` statements with `import` statements in both **_server.js_** and **_electron.js_**.

```javascript
require('./api/server'); 
// will become 
import './api/server';
```

You can try running the app after making those changes, you'll get a `SyntaxError` as expected.

We're going to be using `env` preset with babel-register plugin, if you don't know about Babel presets, a preset is basically the right config put toghther and `env` is one of the official presets. (You can learn more about presets and plugins here: [Plugins · Babel](https://babeljs.io/docs/plugins/))

```
$ npm i --save-dev babel-preset-env babel-register
```

For Babel to work, we'll need to create a config file in our project root called **_.babelrc_**.

Here's how **_.babelrc_** looks like:

```json
{
    "presets": ["env"]
}
```

The last thing we'll do is modify our start script in **_package.json_**

```json
"start": "electron ." 
// becomes
"start": "electron -r babel-register ."
```

That's it, now `npm start` shouldn't be throwing any `SyntaxError` for the `import` statements.

Here's a screenshot:

![Image that shows that It is working!](https://i.imgur.com/dh4u14f.png "It is working!")
{: .center}

---

**What's Next?**

In the next post, we'll create a REST API with Express and use Sequelize, with SQLite as the database.

If you have any questions or suggestions, please comment below or send an email to me. It would be very helpful.

Thank you for reading.

[Git Repository](https://github.com/sacredSatan/alchemy/) (**part-1** branch for all the changes done till this post.)
