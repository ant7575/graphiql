GraphiQL
========

*/ˈɡrafək(ə)l/* An graphical interactive in-browser GraphQL IDE. [Try the live demo](http://graphql-swapi.parseapp.com/).

[![Build Status](https://travis-ci.org/graphql/graphiql.svg?branch=master)](https://travis-ci.org/graphql/graphiql)

[![](resources/graphiql.png)](http://graphql-swapi.parseapp.com/)

### Getting started

Using a node.js server? Just use [`express-graphql`](https://github.com/graphql/express-graphql)!
It can automatically present GraphiQL. Using another GraphQL service? GraphiQL is pretty easy to set up.

```
npm install --save graphiql
```

GraphiQL provides a React component responsible for rendering the UI, which
should be provided with a function for fetching from GraphQL, we recommend using
the [fetch](https://fetch.spec.whatwg.org/) standard API.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import GraphiQL from 'graphiql';
import fetch from 'isomorphic-fetch';

function graphQLFetcher(graphQLParams) {
  return fetch(window.location.origin + '/graphql', {
    method: 'post',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(graphQLParams),
  }).then(response => response.json());
}

ReactDOM.render(<GraphiQL fetcher={graphQLFetcher} />, document.body);
```

Build for the web with [webpack](http://webpack.github.io/) or
[browserify](http://browserify.org/), or use the pre-bundled graphiql.js file.
See the example in the git repository to see how to use the pre-bundled file.

Don't forget to include the CSS file on the page! If you're using npm, you can
find it in `node_modules/graphiql/graphiql.css`, or you can download it from the
[releases page](/graphql/graphiql/releases).

For an example of setting up a GraphiQL, check out the [example](./example) in
this repository which also includes a few useful features highlighting
GraphiQL's API.


### Features

* Syntax highlighting
* Intelligent type ahead of fields, arguments, types, and more.
* Real-time error highlighting and reporting.
* Automatic query completion.
* Run and inspect query results.


### Usage

GraphiQL exports a single React component which is intended to encompass the
entire browser viewport. This React component renders the GraphiQL editor.

```js
import GraphiQL from 'graphiql';

<GraphiQL />
```

GraphiQL supports customization in UI and behavior by accepting React props
and children.

**Props:**

- `fetcher`: a function which accepts GraphQL-HTTP parameters and returns
  a Promise or Observable which resolves to the GraphQL parsed JSON response.

- `schema`: a GraphQLSchema instance or `null` if one is not to be used.
  If `undefined` is provided, GraphiQL will send an introspection query
  using the fetcher to produce a schema.

- `query`: an optional GraphQL string to use as the initial displayed query,
  if `undefined` is provided, the stored query or defaultQuery will
  be used.

- `variables`: an optional GraphQL string to use as the initial displayed
  query variables, if `undefined` is provided, the stored variables will
  be used.

- `operationName`: an optional name of which GraphQL operation should be
  executed.

- `response`: an optional JSON string to use as the initial displayed
  response. If not provided, no response will be initialy shown. You might
  provide this if illustrating the result of the initial query.

- `storage`: an instance of [Storage][] GraphiQL will use to persist state.
  Only `getItem` and `setItem` are called. Default: window.localStorage

- `defaultQuery`: an optional GraphQL string to use when no query is provided
  and no stored query exists from a previous session. If `undefined` is
  provided, GraphiQL will use its own default query.

- `onEditQuery`: an optional function which will be called when the Query
  editor changes. The argument to the function will be the query string.

- `onEditVariables`: an optional function which will be called when the Query
  varible editor changes. The argument to the function will be the
  variables string.

- `onEditOperationName`: an optional function which will be called when the
  operation name to be executed changes.

- `getDefaultFieldNames`: an optional function used to provide default fields
  to non-leaf fields which invalidly lack a selection set.
  Accepts a GraphQLType instance and returns an array of field names.
  If not provided, a default behavior will be used.

**Children:**

* `<GraphiQL.Logo>`: Replace the GraphiQL logo with your own.

* `<GraphiQL.Toolbar>`: Add a custom toolbar above GraphiQL.

* `<GraphiQL.ToolbarButton>`: Add a button to the toolbar above GraphiQL.

* `<GraphiQL.Footer>`: Add a custom footer below GraphiQL Results.

### Usage Examples

```js
class CustomGraphiQL extends React.Component {
  constructor(props) {
    this.state = {
      // REQUIRED:
      // `fetcher` must be provided in order for GraphiQL to operate
      fetcher: this.props.fetcher,
      
      // OPTIONAL PARAMETERS
      // GraphQL artifacts
      query: '',
      variables: '',
      response: '',
      
      // GraphQL Schema
      // If `undefined` is provided, an introspection query is executed
      // using the fetcher.
      schema: undefined,
      
      
      // Useful to determine which operation to run
      // when there are multiple of them.
      operationName: null,
      storage: null,
      defaultQuery: null,
      
      // Custom Event Handlers
      onEditQuery: null,
      onEditVariables: null,
      onEditOperationName: null,
      
      // GraphiQL automatically fills in leaf nodes when the query
      // does not provide them. Change this if your GraphQL Definitions
      // should behave differently than what's defined here:
      // (https://github.com/graphql/graphiql/blob/master/src/utility/fillLeafs.js#L75)
      getDefaultFieldNames: null
    };
  }
  
  _onClickToolbarButton(event) {
    alert('Clicked toolbar button!');
  }
  
  render() {
    return (
      <GraphiQL ...this.state>
        <GraphiQL.Logo>
          Custom Logo
        </GraphiQL.Logo>
        <GraphiQL.Toolbar>
          // GraphiQL.ToolbarButton usage
          <GraphiQL.ToolbarButton
            onClick={this._onClickToolbarButton}
            title="ToolbarButton"
            label="Click Me as well!"
          />
          // Some other possible toolbar items
          <button name="GraphiQLButton">Click Me</button>
          <OtherReactComponent someProps="true" />
        </GraphiQL.Toolbar>
        <GraphiQL.Footer>
          // Footer works the same as Toolbar
          // add items by appending child components
        </GraphiQL.Footer>
      </GraphiQL>
    );
  }
}
```


### Query Samples

**Query**

GraphQL queries declaratively describe what data the issuer wishes to fetch from whoever is fulfilling the GraphQL query.
```
query FetchSomeIDQuery($someId: String!) {
  human(id: $someId) {
    name
  }
}
```
More examples available from: [GraphQL Queries](http://graphql.org/docs/queries/).

**Mutation**

Given this schema,
```
const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    fields: {
      numberHolder: { type: numberHolderType },
    },
    name: 'Query',
  }),
  mutation: new GraphQLObjectType({
    fields: {
      immediatelyChangeTheNumber: {
        type: numberHolderType,
        args: { newNumber: { type: GraphQLInt } },
        resolve: (function (obj, { newNumber }) {
          return obj.immediatelyChangeTheNumber(newNumber);
        }:any)
      }
    },
    name: 'Mutation',
  })
});
```
then the following mutation queries are possible:
```
mutation TestMutation {
  first: immediatelyChangeTheNumber(newNumber: 1) {
    theNumber
  }
}
```
Read more in [this mutation test in `graphql-js`](https://github.com/graphql/graphql-js/blob/master/src/execution/__tests__/mutations-test.js).

[Relay](https://facebook.github.io/relay) has another good example using a common pattern for composing mutations:
Given the following GraphQL Type Definitions,
```
input IntroduceShipInput {
  factionId: ID!
  shipName: String!
  clientMutationId: String!
}

type IntroduceShipPayload {
  faction: Faction
  ship: Ship
  clientMutationId: String!
}
```
mutation calls are composed as such:
```
mutation AddBWingQuery($input: IntroduceShipInput!) {
  introduceShip(input: $input) {
    ship {
      id
      name
    }
    faction {
      name
    }
    clientMutationId
  }
}
{
  "input": {
    "shipName": "B-Wing",
    "factionId": "1",
    "clientMutationId": "abcde"
  }
}
```
Read more from [Relay Mutation Documentation](https://facebook.github.io/relay/docs/graphql-mutations.html).

**Fragment**

Fragments allow for the reuse of common repeated selections of fields, reducing duplicated text in the document. Inline Fragments can be used directly within a selection to condition upon a type condition when querying against an interface or union.
Therefore, instead of the following query:
```
{
  luke: human(id: "1000") {
    name
    homePlanet
  }
  leia: human(id: "1003") {
    name
    homePlanet
  }
}
```

using fragments, the following query is possible.
```
{
  luke: human(id: "1000") {
    ...HumanFragment
  }
  leia: human(id: "1003") {
    ...HumanFragment
  }
}

fragment HumanFragment on Human {
  name
  homePlanet
}
```

Read more from [GraphQL Fragment Specification](http://facebook.github.io/graphql/#sec-Language.Fragments).
