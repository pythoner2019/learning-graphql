# Part 7: Deep Dive into GraphQL Type System

This part may feel more like manuals than a tutorial. I wrote it to save myself from jumping between GraphQL RFC Spec and the test suit of reference JavaScript implementation. I'll be glad if it saves some of your time too.

At the heart of any GraphQL implementation is a description of what types of objects it can return, described with a GraphQL type system.

The JavaScript implementation has the following named types implemented in it:

## `GraphQLScalarType`:

Scalar types hold a single value. GraphQL provides a basic set of well‐defined Scalar types.

`GraphQLInt` (example: 2)

`GraphQLFloat` (example: 2.0)

`GraphQLString` (example: "Hello World")

`GraphQLBoolean` (example: true)

`GraphQLID` (integer or string of any format)

Other types:

## `GraphQLObjectType`

GraphQL Objects represent a list of named fields, each of which also yields a value of their own type. Example:

```
var PersonType = new GraphQLObjectType({
  name: 'Person',
  fields: () => ({
    name: {
      type: GraphQLString,
      description: 'The name of the person.',
    },
    age: {
      type: GraphQLInt,
      description: 'The age of the person.',
    }
  })
});

```

## `GraphQLInterfaceType`

GraphQL Interfaces represent a reusable collection of fields and their arguments. GraphQL object can then implement an interface, which guarantees that they will contain the specified fields.

```
class Dog {
  constructor(name, barks) {
    this.name = name;
    this.barks = barks;
  }
}

class Cat {
  constructor(name, meows) {
    this.name = name;
    this.meows = meows;
  }
}

var NamedType = new GraphQLInterfaceType({
  name: 'Named',
  fields: {
    name: { type: GraphQLString }
  }
});

var DogType = new GraphQLObjectType({
  name: 'Dog',
  interfaces: [ NamedType ],
  fields: {
    name: { type: GraphQLString },
    barks: { type: GraphQLBoolean }
  },
  isTypeOf: (value) => value instanceof Dog
});

var CatType = new GraphQLObjectType({
  name: 'Cat',
  interfaces: [ NamedType ],
  fields: {
    name: { type: GraphQLString },
    meows: { type: GraphQLBoolean }
  },
  isTypeOf: (value) => value instanceof Cat
});

```

## `GraphQLList`

A GraphQL list is a collection type which declares the type of each item in the List.

```
var BlogArticle = new GraphQLObjectType({
  name: 'Article',
  fields: {
    id: { type: GraphQLString },
    isPublished: { type: GraphQLBoolean },
    author: { type: GraphQLString },
    title: { type: GraphQLString },
    body: { type: GraphQLString }
  }
});

var BlogQuery = new GraphQLObjectType({
  name: 'Query',
  fields: {
    article: {
      args: { id: { type: GraphQLString } },
      type: BlogArticle
    },
    feed: {
      type: new GraphQLList(BlogArticle)
    }
  }
});

```

## `GraphQLNonNull`

By default, all types in GraphQL are nullable; the null value is a valid response for all of the above types. To declare a type that disallows null, the GraphQL Non‐Null type can be used.

`GraphQLNonNull` acts more like a wrapper on other types. Remember to instantiate it with `new` when disallowing null on some typed field.

```
var BlogArticle = new GraphQLObjectType({
  name: 'Article',
  fields: {
    id: { type: new GraphQLNonNull(GraphQLString) },
    isPublished: { type: new GraphQLNonNull(GraphQLBoolean) },
    author: { type: new GraphQLNonNull(GraphQLString) },
    title: { type: GraphQLString },
    body: { type: GraphQLString }
  }
});
```


## `GraphQLEnumType`

GraphQL Enum represents one the of possible values:

```
var episodeEnum = new GraphQLEnumType({
  name: 'Episode',
  description: 'One of the films in the Star Wars Trilogy',
  values: {
    NEWHOPE: {
      value: 4,
      description: 'Released in 1977.',
    },
    EMPIRE: {
      value: 5,
      description: 'Released in 1980.',
    },
    JEDI: {
      value: 6,
      description: 'Released in 1983.',
    },
  }
});

```


## `GraphQLUnionType`

GraphQL Unions represent an object that could be one of items in a list of GraphQL Object types. They also differ from interfaces in that Object types declare what interfaces they implement, but are not aware of what unions contain them.

```
class Dog {
  constructor(name, barks) {
    this.name = name;
    this.barks = barks;
  }
}

class Cat {
  constructor(name, meows) {
    this.name = name;
    this.meows = meows;
  }
}

var NamedType = new GraphQLInterfaceType({
  name: 'Named',
  fields: {
    name: { type: GraphQLString }
  }
});

var DogType = new GraphQLObjectType({
  name: 'Dog',
  interfaces: [ NamedType ],
  fields: {
    name: { type: GraphQLString },
    barks: { type: GraphQLBoolean }
  },
  isTypeOf: value => value instanceof Dog
});

var CatType = new GraphQLObjectType({
  name: 'Cat',
  interfaces: [ NamedType ],
  fields: {
    name: { type: GraphQLString },
    meows: { type: GraphQLBoolean }
  },
  isTypeOf: value => value instanceof Cat
});

var PetType = new GraphQLUnionType({
  name: 'Pet',
  types: [ DogType, CatType ],
  resolveType(value) {
    if (value instanceof Dog) {
      return DogType;
    }
    if (value instanceof Cat) {
      return CatType;
    }
  }
});

```

## `GraphQLInputObjectType`

Fields can define arguments that the client passes up with the query, to configure their behavior. These inputs can be Strings or Enums, but they sometimes need to be more complex than that.

The Object type has not been used here intentionally, because Objects can contain fields that express circular references or references to interfaces and unions, neither of which is appropriate for use as an input argument. For this reason, input objects have a separate type in the system.

An Input Object defines input fields; the input fields are either scalars, enums, or other input objects. This allows arguments to accept arbitrarily complex structures.

```
var ComplexInput = new GraphQLInputObjectType({
  name: 'ComplexInput',
  fields: {
    requiredField: { type: new GraphQLNonNull(GraphQLBoolean) },
    intField: { type: GraphQLInt },
    stringField: { type: GraphQLString },
    booleanField: { type: GraphQLBoolean },
    stringListField: { type: new GraphQLList(GraphQLString) },
  }
});

var ComplicatedArgs = new GraphQLObjectType({
  name: 'ComplicatedArgs',
  fields: () => ({
    complexArgField: {
      type: GraphQLString,
      args: {
        complexArg: { type: ComplexInput }
      },
    }
  }),
});

```
