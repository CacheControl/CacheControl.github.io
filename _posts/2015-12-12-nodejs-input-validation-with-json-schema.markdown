---
layout: post
title: Node.js Input Validation using Json-Schema
date: 2015-12-12
comments: true
external-url:
categories: Node.js
---

Checking the validity of data sent to an API is an important responsibility of any service. Besides being an important security feature, it’s also important for responding intelligently to consumers who provide invalid input. In this post, we’ll explore how to parse, validate, and respond to POST requests using json-schema.

## What not to do

Let’s start by describing the anti-pattern:

```js
// POST /users
{
   "first": "george",
   "last": "washington",
   "username": true,
   "rememberEmail": "gwashington"
}

// RESPONSE:
500 Internal Server error
```

Oops. Looks like our API consumer has a bug in their code. They accidentally switched the values for username and rememberEmail. Unfortunately, the response from the server isn’t very helpful; the API developer didn’t properly validate the input parameters, and has a broken app to boot.

Let’s see if we can do better.

## Creating a Schema

We could certainly write some conditional logic in our application to validate incoming parameters, but that would be re-inventing the wheel. There’s already a full-featured and mature solution for validating json called [json-schema](http://json-schema.org/).

First, let’s create a new-user.json schema file that defines the POST body we expect:

```json
{
  "title": "new user",
  "description": "describes properties required to create a user",
  "type": "object",
  "properties": {
    "first": {
      "type": "string",
      "description": "firstname of the account user"
    },
    "last": {
      "type": "string",
      "description": "lastname of the account user"
    },
    "username": {
      "type": "string",
      "description": "username of account"
    },
    "rememberEmail": {
      "type": "boolean",
      "description": "whether the users email address should be remembered"
    }
  },
  "required": ["first", "last", "username", "rememberEmail"]
}
```

The schema above is easy to read and mostly self-explanatory. It describes what an acceptable request body looks like to our endpoint. If you’d like to read more about json-schema, I recommend [understanding json schema](https://json-schema.org/understanding-json-schema/index.html).

## Honoring the Schema

Now that we have a schema defined, we need to give the application a way to validate incoming requests against it. To accomplish this, let’s install a json-validator package. I highly recommend Evgeny Poberezkin’s most excellent [ajv](https://github.com/epoberezkin/ajv) package (“another json validator”). We’ll install it and write some middleware:

```js
let Ajv = require('ajv')
let ajv = Ajv({ allErrors:true, removeAdditional:'all' })
let userSchema = require('./new-user.schema')
ajv.addSchema(userSchema, 'new-user')

/**
 * Format error responses
 * @param  {Object} schemaErrors - array of json-schema errors, describing each validation failure
 * @return {String} formatted api response
 */
function errorResponse(schemaErrors) {
  let errors = schemaErrors.map((error) => {
    return {
      path: error.dataPath,
      message: error.message
    }
  })
  return {
    status: 'failed',
    errors: errors
  }
}

/**
 * Validates incoming request bodies against the given schema,
 * providing an error response when validation fails
 * @param  {String} schemaName - name of the schema to validate
 * @return {Object} response
 */
let validateSchema = (schemaName) => {
  return (req, res, next) => {
    let valid = ajv.validate(schemaName, req.body)
    if (!valid) {
      return res.send(errorResponse(ajv.errors))
    }
    next()
  }
}
```

…and add the middleware to our route:

```js
app.post('/users', validateSchema('new-user'), (req, res) => {
  // parameters are valid; code can interact with database
  // ! you must still protect from SQL injection !
  res.send(req.body).status(201).end()
})
```

## Trying it out

Now we can try our POST again:

```js
// POST /users
{
   "first": "george",
   "last": "washington",
   "username": true,
   "rememberEmail": "gwashington"
}

// RESPONSE:
{
  "status": "failed",
  "errors": [
    {
      "path": ".username",
      "message": "should be string"
    },
    {
      "path": ".rememberEmail",
      "message": "should be boolean"
    }
  ]
}
```

Huzzah! This response gives our consumer a clear understanding of their mistake. Not bad for 25 lines of code!

## Removing extra parameters

What if the user POSTed additional attributes, which don’t apply to this endpoint? We should still able to process the request, but it’d be nice to strip out the additional properties from the request body, before moving forward with our business logic. This is easily accomplished with the removeAdditional option in ajv:

> var ajv = Ajv({ allErrors:true, removeAdditional:’all’ })

Now, when extra parameters are provided in the POST, req.body will only include the properties mentioned in the schema:

```js
// POST /users
{
   "first": "george",
   "last": "washington",
   "username": "true",
   "EXTRAPARAMETER": "shouldnt be here",
   "rememberEmail": true
}

// req.body:
{
  "first": "george",
  "last": "washington",
  "username": "true",
  "rememberEmail": true
}
```

## Schemas as documentation

In addition to the functional advantages described above, json-schema easily doubles as a source of documentation for your API. Don’t be afraid to provide schemas in your documentation; they describe each contract precisely and in ways consumers can easily understand.

At this point, it should be mentioned that by implementing json-schema this way, we’re about halfway to using [Swagger](https://swagger.io/). Swagger is a specification for describing RESTful API’s, and uses json-schema for describing input parameters and responses. Consider taking the project a step further and defining it in Swagger.

## Summary

With 25 lines of code, we’ve creating a solution for validating input parameters, that easily scales across the entire application. Security has improved, and there is a clear, unambiguous definition for the endpoint’s contract.
