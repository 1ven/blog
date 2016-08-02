---
layout: post
title: "Building fake RESTful API for your single-page applications"
date: 2016-07-31 07:26:00 +0300
categories: REST
comments: true
---
Sometimes, when you start building single-page applications, server is not implemented yet, but you need to work. In this situations you need to have a fake server, which will be handling your api requests.
<!--more-->

Good solution for this purpose will be using `json-server` npm package. It can be a temporary replacement of real RESTful API untill the real one is not implemented.

---

First, you need to install it:

```
npm install json-server --save-dev
```

Official docs says, that you should install it globally, but I don't suggest you doing this. Installing package as `dev` dependency makes bootstrapping of your application more easier.

When new developer is joining your team, everything he needs(in most cases), is to run `npm install` command and he can work. Plus we are avoiding problems assigned with version incompatibility.

To make able to run local packages as global, you should add them to npm tasks. Append this line to `scripts` section of `package.json` file:

```
"fake-server": "json-server --watch db.json -p 1337"
```

This command will be used to run fake server. It port should differ from your client bundler.

---

To start server, simply run `npm run fake-server` command.

All our data will be stored in `json` files, instead of database. `db.json` file will be created, when you will run `json-server` first time. If you need to have some dumb data initially, you can add it to this file by hand, it would not be overwritten by server:

```json
{
  "todos": [
    {
      "id": 1,
      "title": "Buy milk",
      "completed": true
    },
    {
      "id": 2,
      "title": "Feed cat",
      "completed": false
    }
  ]
}
```

---

After server was runned, it can handle CRUD requests. This means you can `create`, `read`, `update` and `delete` entries from our fake database.

Below, there are basic examples of CRUD requests:

`GET` request. Returns all entries of requested type:

```javascript
xhr({
  method: 'GET',
  uri: 'http://localhost:1337/todos',
  headers: {
    "Content-Type": "application/json"
  }
}, (err, response) => {
  // response is `[{ id: 1, title: 'Buy milk'} ... ]`
});
```

`POST` request. Creates entry of particular type.

```javascript
xhr({
  method: 'POST',
  uri: 'http://localhost:1337/todos',
  headers: {
    "Content-Type": "application/json"
  },
  body: {
    title: 'Meet John',
    completed: false
  }
});
```

`PATCH` request. Updates particular entry fields. To update whole entry use `PUT` instead.

```javascript
xhr({
  method: 'PATCH',
  uri: 'http://localhost:1337/todos/2',
  headers: {
    "Content-Type": "application/json"
  },
  body: {
    completed: true
  }
});
```

`DELETE` request. Removes entry.

```javascript
xhr({
  method: 'DELETE',
  uri: 'http://localhost:1337/todos/1',
  headers: {
    "Content-Type": "application/json"
  }
});
```

This is important to use uppercase notation in naming request types(`POST`, `GET` etc.). The fake server can not handle requests with lowercase request types in some ways.

---

If you need more additional information, you can get it from [official](//github.com/typicode/json-server) docs.
