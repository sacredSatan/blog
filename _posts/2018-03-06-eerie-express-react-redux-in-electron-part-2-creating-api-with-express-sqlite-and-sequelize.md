---
layout: post
title: "!EERiE: Part 2: Express, SQLite and Sequelize"
description: "Part two of a multi-part series on using Electron, Express, React and Redux together."
comments: true
---

**Not EERiE** is a series that demonstrates that *EERiE* (**E**lementary [**E**xpress + **R**eact **i**n **E**lectron]) is not really *eerie* and pretty easy to approach.

> # In the series:
>
> **Part 1**: [Setting up Electron and Express](/blog/2018/03/eerie-express-react-redux-in-electron-part-1-setting-up-electron-and-express/)
>
> **Part 2**: Creating an API with Express, SQLite and Sequelize **(You are here)**

---

In the previous post we setup Electron and Express. In this one we're going to create a REST API for managing a list of pirates.

This post can be divided in two main parts:

1. SQLite and Sequelize 

2. Creating and testing the API


## SQLite and Sequelize

Sequelize is an ORM for Node.js with which we will connect to SQLite database, using Sequelize we can connect and talk to our Database using plain old Javascript instead of writing SQL (Sequelize also supports execution of raw SQL queries). You can read more about Sequelize [here](http://docs.sequelizejs.com/).

Let's begin by installing SQLite and Sequelize in our project.

```
$ npm i --save sequelize sqlite3
```

However, you will need to compile sqlite from source for your operating system. You will get some message like: "Please install sqlite3 package manually" if you try to run the app.

One possible solution is to follow this: [Building for node-webkit](https://github.com/mapbox/node-sqlite3#building-for-node-webkit), but we're going to use something else that is simpler and easy to use. We'll use [electron-builder](https://github.com/electron-userland/electron-builder). Another reason of using this besides it being incredibly easy to use is that we'll hopefully use electron-builder at the end of our journey to build for distribution.

Install electron-builder with:

```
$ npm i --save-dev electron-builder
```

We will now have to add a script in our **_package.json_**, call it `postinstall` so when you're installing you don't forget to run the command.

```
"postinstall": "electron-builder install-app-deps"
```

Run the script manually with `npm run postinstall`. It should build the application dependencies for your native platform.

Now sqlite should be properly installed and ready to be used.

First we're going to do something similar to the Example Usage in the [Sequelize Docs](http://docs.sequelizejs.com/) homepage to test if everything is working correctly.

Here's how **_server.js_** looks like now:

```javascript
import express from 'express';
import Sequelize from 'sequelize';

const app = express();

// connecting to sqlite database
const db = new Sequelize('db', null, null, {
  dialect: 'sqlite',
  storage: './db.sqlite'
});

// defining a model called 'pirate'
const Pirate = db.define('pirate', {
  name: Sequelize.STRING
});

// syncing db to the model changes
db.sync()

  // after sync, add a pirate
  .then(() => Pirate.create({
    name: 'Jack Sparrow',
  }));

app.get('/', (req, res) => {

  // querying for all pirates in db for their name
  Pirate.findAll({
    attributes: ['name']
  })

  // send a json response back with the returned data
  .then((pirates) => {
    res.json(pirates);
  });
});

app.listen(5000, () => console.log('App running on port 5000'));
```

The comments should explain everything. On **line 14** the `Sequelize.STRING` is one of many datatypes available in Sequelize, and you can find a list here: [Sequelize DataTypes](http://docs.sequelizejs.com/manual/tutorial/models-definition.html#data-types).

So if everything is fine, this should do the following:

1. Connect to database
2. Sync the database with model changes
 - Then create a new Pirate with name "Jack Sparrow"
3. Start listening for requests on port 5000.

Now our electron window upon creation hits `localhost:5000` with a GET request, which we've handled here on our **_server.js_** on **line 25** where we query 'name' of every pirate in the database, and then send the result as a response. We're only asking for 'name' but Sequelize automatically creates `id`, `updatedAt` and `createdAt` fields.

For more info about [querying in Sequelize](http://docs.sequelizejs.com/manual/tutorial/querying.html).

You can start the app with `npm start` and you should see something like this in the screenshot:

![Test SQLite/Sequelize setup image!](https://i.imgur.com/t5jyH5I.png "SQLite and Sequelize are working as expected.")
{: .center}

We're adding a pirate called "Jack Sparrow" everytime the app is restarted. You can check your console and see the sql commands Sequelize executes for us.

---

### Refactoring

Before we start writing any API related code, let's refactor what we've done till now. Here's my current directory structure right now:

```
$ tree -I node_modules
.
├── app
│   ├── api
│   │   └── server.js
│   └── electron.js
├── db.sqlite
├── package.json
└── package-lock.json

2 directories, 5 files
```

We'll create a few files in different directories:

**_dbHelper.js_**

```javascript
import Sequelize from 'sequelize';

const getDb = () => {
    return new Sequelize('db', null, null, {
      dialect: 'sqlite',
      storage: './db.sqlite'
    });
  };

export {getDb};
```

**_pirateModel.js_**

```javascript
import Sequelize from 'sequelize';

export default (db) => {
  return db.define('pirate', {
    name: Sequelize.STRING
  });
}
```

Here's what the directory structure looks like now:

```
$ tree -I node_modules
.
├── app
│   ├── api
│   │   ├── helpers
│   │   │   └── dbHelper.js
│   │   ├── models
│   │   │   └── pirateModel.js
│   │   └── server.js
│   └── electron.js
├── db.sqlite
├── package.json
└── package-lock.json

4 directories, 7 files
```

We'll make some changes in **_server.js_** by importing stuff from our newly created files and using that instead.

Here's how **_server.js_** looks like now:

```javascript
...
import {getDb} from './helpers/dbHelper';
import PirateModel from './models/pirateModel';

const app = express();

// connecting to sqlite database
const db = getDb();

// defining a model
const Pirate = PirateModel(db);
...
```

You can run the app to test if everything is working fine.

---

## Creating and testing the API

For this, we'll create two routes:

1. /api/pirates/ (List)
2. /api/pirates/:pirateId/ (Detail)

We'll keep our API related routes behind `/api/`. Let's begin with the List route.

### 1. /api/pirates/ [GET, POST]

This route will support 2 methods, `GET` and `POST`. `GET` will return a list of all pirates and `POST` will support adding a new pirate in the database.

We'll start by creating routes using `express.Router()`. Create a directory called 'routes' inside the 'api' directory with **_pirateRoutes.js_** file in it.

Here's how **_pirateRoutes.js_** looks like:

```javascript
import express from 'express';

export default (Pirate) => {
  const pirateRouter = express.Router();

  pirateRouter.route('/')
    .get((req, res) => {
      res.send('GET Pirate')
    })
    .post((req, res) => {
      res.send('POST Pirate')
    });
  return pirateRouter;
}
```

The `Pirate` argument in that function is the 'pirate' model. We'll need it to fetch and update pirates.

Now we need to modify **_server.js_** so it will use the routes.

Here's how **_server.js_** looks like:

```javascript
import express from 'express';
import Sequelize from 'sequelize';
import {getDb} from './helpers/dbHelper';
import PirateModel from './models/pirateModel';
import PirateRouter from './routes/pirateRoutes';

const app = express();

// connecting to sqlite database
const db = getDb();

// defining a model
const Pirate = PirateModel(db);

// syncing db to the model changes
db.sync();

// router
const pirateRouter = PirateRouter(Pirate);

// using the router at /api/pirates
app.use('/api/pirates', pirateRouter);

app.listen(5000, () => console.log('App running on port 5000'));
```

We can check if everything is working correctly by running the app and sending a GET and/or POST request to `/api/pirates`. I recommend using [Postman](https://www.getpostman.com/) for that, it is pretty simple to use.

![Test if router worked as expected!](https://i.imgur.com/omXrwG0.png "Router is working as expected.")
{: .center}

As you can see in the screenshot, it is working as expected. Notice the "Cannot GET /" in our Electron window, because we deleted the code that handled `GET` requests on `/` so express app throws 404 on `/` now.

Now that the router works, we'll create a controller that will handle the requests.

Create a directory called 'controllers' inside the 'api' folder and create a file called **_pirateController.js_** inside that.

Here's my directory structure:

```
.
├── app
│   ├── api
│   │   ├── controllers
│   │   │   └── pirateController.js
│   │   ├── helpers
│   │   │   └── dbHelper.js
│   │   ├── models
│   │   │   └── pirateModel.js
│   │   ├── routes
│   │   │   └── pirateRoutes.js
│   │   └── server.js
│   └── electron.js
├── db.sqlite
├── package.json
└── package-lock.json

6 directories, 9 files
```

Here's how **_pirateController.js_** looks like:

```javascript
const pirateListController = (Pirate) => {
  const post = (req, res) => {
    Pirate.create(req.body) // 'body' property created by bodyParser
    .then((savedPirate) => {
      res.status(201).send(savedPirate);
    })
    .catch((error) => {
      res.status(500).send(error);
    });
  }
  const get = (req, res) => {
    Pirate.findAll()
    .then((pirates) => {
      res.json(pirates);
    })
    .catch((error) => {
      res.status(500).send(error);
    });
  }
  return {
    post,
    get,
  }
};

export {pirateListController};
```

To parse the request body as json, we need use `bodyParser` middleware, it is bundled with Express itself and doesn't need to be separately installed. We just have to add a single line in our **_server.js_**

Here's how **_server.js_** looks like:

```javascript
...
// router
const pirateRouter = PirateRouter(Pirate);

// parse request body as json
app.use(express.json());

// using the router at /api/pirates
app.use('/api/pirates', pirateRouter);
...
```

This will create a `body` property under the `request` object, and we'll have all the data in the request in `json` format.

That's how **line 3** on **_pirateController.js_** works. If we send a request with a single json object in body, with "name" key and some value, it should create a new pirate with that name. We're not validating the request data, ideally we would do that.

One last thing that remains is to edit our **_pirateRoutes.js_** to use the controllers.

Here's how **_pirateRoutes.js_** looks like:
```javascript
import express from 'express';
import {pirateListController} from '../controllers/pirateController';

export default (Pirate) => {
  const pirateRouter = express.Router();
  const listController = pirateListController(Pirate);

  pirateRouter.route('/')
    .get(listController.get)
    .post(listController.post);

  return pirateRouter;
}
```

Start the app and send a `POST` request with `application/json` content type and this data to */api/pirates/*:

```json
{
  "name": "Blackbeard"
}
```

It should return status `201` and the newly created pirate, and a `GET` on the same url should return a list of all pirates. The list route is done.

### 2. /api/pirates/:pirateId [GET, PUT, PATCH, DELETE]

Now let's do the detail route. You can imagine there's one thing that you need to do in handling every method in this, and that is finding the Pirate based on pirateId passed in the url.

We can create a function that does this, since we're using Express, we'll create a middleware. A middleware is nothing but a function that accepts 3 arguments, `request`, `response` and most importantly `next`. When your middleware has finished its work, you should call `next()` so Express execute whatever that is supposed to come next.

You can read more about Express Middlewares [here](https://expressjs.com/en/guide/writing-middleware.html). It is explained clearly with examples.

Create a new file called **_apiMiddlewares.js_** in 'helpers' folder.

Here's how **_apiMiddlewares.js_** looks like:

```javascript
const getPirateById = (Pirate) => {
  return (req, res, next) => {
    Pirate.find({
      where: {
        id: req.params.pirateId,
      }
    })
    .then((pirate) => {
      if(pirate) {
        req.pirate = pirate;
        next();
      } else {
        res.status(404).send('No pirate found!');
      }
    })
    .catch((error) => {
      res.status(500).send(error);
    });
  }
}

export {getPirateById};
```

See the `req.params.pirateId` on **line 5**. What we'll do is in our router we'll define the path as '/:pirateId' (/api/pirates/:pirateId). You can add parameters in your path by adding a colon. Express will take the values and populate an object called `req.params` with the name as specified in the path.

Here's a simple example like the one in the [guide on Routing in Express](https://expressjs.com/en/guide/routing.html):

```
Route path: /api/pirates/:pirateId
Request URL: http://localhost:5000/pirates/13
req.params: { "pirateId": "13" }
```

Let's make the change, edit the **_pirateRoutes.js_** file so that it looks like this:

```javascript
import express from 'express';
import {getPirateById} from '../helpers/apiMiddlewares.js';
import {pirateListController} from '../controllers/pirateController';

export default (Pirate) => {
  const pirateRouter = express.Router();
  const listController = pirateListController(Pirate);
  const getPirateHelper = getPirateById(Pirate);

  pirateRouter.route('/')
    .get(listController.get)
    .post(listController.post);

  pirateRouter.use('/:pirateId', getPirateHelper);

  return pirateRouter;
}
```

We _use_ the middlewares using the `use` method (**line  14**). Now our middleware is in place, that will find a pirate everytime there's a request that matches the path '/api/pirates/:pirateId' and add it to the `req` object.

It is time to create the controller that will handle get, put, patch and delete requests on the pirate added to `req` object by the middleware.

Here's how **_pirateController.js_** looks like:
```javascript
...
// the list controller
...

const pirateDetailController = (Pirate) => {
  const get = (req, res) => {
    res.json(req.pirate);
  }

  const put = (req, res) => {
    if(req.body.name) {
      req.pirate.name = req.body.name;
      req.pirate.save()
        .then((savedPirate) => {
          res.json(savedPirate);
        })
        .catch((error) => {
          res.status(500).send(error);
        });
    } else {
      res.status(500).send('Invalid value for name.');
    }
  };

  const patch = put;

  const del = (req, res) => {
    req.pirate.destroy()
      .then(() => {
        res.status(204).send('Pirate removed!');
      })
      .catch((error) => {
        res.status(500).send(error);
      });
  }

  return {
    get,
    put,
    patch,
    del,
  };
};

export {pirateListController, pirateDetailController};
```

Let's do a quick review of all the handlers:

- `get` simply returns the pirate added to the `req` object by the middleware as a response. 

- `put` first checks to see if there's a `name` sent in the `req.body` (that is also populated by a middleware we added in our **_server.js_** to parse request body as json). If there is a valid value, it assigns the name sent in the request body to the pirate's name (**line 12** in the snippet above) and tries to save it, if successful then returns the updated pirate, otherwise the error.

- `patch` is same as `put` because we only have one field to update in the model.

- `del` is self explanatory.

Edit the **_pirateRoutes.js_** to use the detail controller.

Here's how **_pirateRoutes.js_** looks like:

```javascript
import express from 'express';
import {getPirateById} from '../helpers/apiMiddlewares.js';
import {pirateListController, pirateDetailController} from '../controllers/pirateController';

export default (Pirate) => {
  const pirateRouter = express.Router();
  const listController = pirateListController(Pirate);
  const detailController = pirateDetailController(Pirate);
  const getPirateHelper = getPirateById(Pirate);

  pirateRouter.route('/')
    .get(listController.get)
    .post(listController.post);

  pirateRouter.use('/:pirateId', getPirateHelper);

  pirateRouter.route('/:pirateId')
    .get(detailController.get)
    .put(detailController.put)
    .patch(detailController.patch)
    .delete(detailController.del);

  return pirateRouter;
}
```

and the current directory structure looks like this:

```
.
├── app
│   ├── api
│   │   ├── controllers
│   │   │   └── pirateController.js
│   │   ├── helpers
│   │   │   ├── apiMiddlewares.js
│   │   │   └── dbHelper.js
│   │   ├── models
│   │   │   └── pirateModel.js
│   │   ├── routes
│   │   │   └── pirateRoutes.js
│   │   └── server.js
│   └── electron.js
├── db.sqlite
├── package.json
└── package-lock.json

6 directories, 10 files
```

You can start the app and test the api using Postman or whatever you like. Everything should be working as expected.

A very basic API for managing a list of pirates is ready.

---

**What's Next?**

In the next post, we'll create a frontend for our API with React.

If you have any questions or suggestions, please comment below or send an email to me. It would be very helpful.

Thank you for reading.

[Git Repository](https://github.com/sacredSatan/alchemy/) (**part-2** branch for all the changes done till this post.)
