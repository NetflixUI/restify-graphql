GraphQL Restify Middleware
==========================

[![Build Status](https://travis-ci.org/netflixui/restify-graphql.svg)](https://travis-ci.org/netflixui/restify-graphql)
[![Coverage Status](https://coveralls.io/repos/netflixui/restify-graphql/badge.svg?branch=master&service=github)](https://coveralls.io/github/netflixui/restify-graphql?branch=master)

Create a GraphQL HTTP server with [Restify](http://mcavage.me/node-restify/).

```sh
npm install --save restify-graphql
```

Install restify-graphql as middleware in your restify server:

```js
var graphqlHTTP = require('restify-graphql');

var app = restify.createServer();

['get', 'post'].forEach(function (method) {
	app[method]('/graphql', graphqlHTTP({ schema: MyGraphQLSchema }));
});
```


### Options

The `graphqlHTTP` function accepts the following options:

  * **`schema`**: A `GraphQLSchema` instance from [`graphql-js`][].
    A `schema` *must* be provided.

  * **`rootValue`**: A value to pass as the rootValue to the `graphql()`
    function from [`graphql-js`][].

  * **`pretty`**: If `true`, any JSON response will be pretty-printed.


### HTTP Usage

Once installed at a path, `restify-graphql` will accept requests with
the parameters:

  * **`query`**: A string GraphQL document to be executed.

  * **`variables`**: The runtime values to use for any GraphQL query variables
    as a JSON object.

  * **`operationName`**: If the provided `query` contains multiple named
    operations, this specifies which operation should be executed. If not
    provided, an 400 error will be returned if the `query` contains multiple
    named operations.

GraphQL will first look for each parameter in the URL's query-string:

```
/graphql?query=query+getUser($id:ID){user(id:$id){name}}&variables={"id":"4"}
```

If not found in the query-string, it will look in the POST request body.

If a previous middleware has already parsed the POST body, the `request.body`
value will be used. Use [`multer`][] or a similar middleware to add support
for `multipart/form-data` content, which may be useful for GraphQL mutations
involving uploading files.

If the POST body has not yet been parsed, graphql-restify will interpret it
depending on the provided *Content-Type* header.

  * **`application/json`**: the POST body will be parsed as a JSON
    object of parameters.

  * **`application/x-www-form-urlencoded`**: this POST body will be
    parsed as a url-encoded string of key-value pairs.

  * **`application/graphql`**: The POST body will be parsed as GraphQL
    query string, which provides the `query` parameter.


### Advanced Options

In order to support advanced scenarios such as installing a GraphQL server on a
dynamic endpoint or accessing the current authentication information,
graphql-restify allows options to be provided as a function of each
restify request.

This example uses [`restify-session`][] to run GraphQL on a rootValue based on
the currently logged-in session.

```js
var session = require('restify-session');
var graphqlHTTP = require('restify-graphql');

var app = restify.createServer();

app.use(session({ secret: 'keyboard cat', cookie: { maxAge: 60000 }}));

app.use('/graphql', graphqlHTTP(request => ({
  schema: MySessionAwareGraphQLSchema,
  rootValue: request.session
})));
```

[`graphql-js`]: https://github.com/graphql/graphql-js
[`multer`]: https://github.com/restifyjs/multer
[`restify-session`]: https://github.com/restifyjs/session
