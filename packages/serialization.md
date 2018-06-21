## Installation

```bash
npm install @brainhubeu/hadron-serialization --save
```

[More info about installation](http://hadron-docs.dev.brainhub.pl/core/#installation)

## Overview

The serializer allows you to quickly and easily shape and parse data way you want it. You just need to create a schema (in a json file, or as a simple object) and you are ready to go.

## Initializing as a Hadron package

Pass the package as an argument for the Hadron bootstrapping function:

```javascript
// ... importing and initializing other components

hadron(expressApp, [require('@brainhubeu/hadron-serialization')], config);
```

That way, you should be able to get it from the DI [container](http://hadron-docs.dev.brainhub.pl/core/#dependency-injection) like so:

```javascript
const serializer = container.take('serializer');
serializer.addSchema({
  name: 'User',
  properties: [ ... ],
});

// ...

const data = { ... };
serializer.serialize(data, 'User');
// or
const data = new User();
serializer.serialize(data);
```

## Initializing without Hadron

Just import the `serializerProvider` function from the package and pass your [schemas](#schema) and parsers there.

```javascript
const serializerProvider = require('@brainhubeu/hadron-serialization');

const serializer = serializerProvider({
  schemas: mySchemas,
  parsers: {
    superParser: (value) => `Super ${value}`,
  },
  // ...
});
```

## Configuration

If you are using Hadron, you just need to add to its config schemas and set parsers:

```javascript
const config = {
  // ...
  serializer: {
    schemas: [ ... ],
    parser: [ ... ],
  }
};
```

If you are using TypeScript, you can just implement the exported interface `ISerializerConfig`:

```typescript
interface ISerializerConfig {
  parsers?: object;
  schemas?: ISerializationSchema[];
}
```

## Usage

The serializer contains three methods.

```javascript
serialize(data, groups, schemaName);
```

* `data` - object we want to serialize
* `groups` - optional array of access [groups](#groups), by default `[]`
* `schemaName` - name of a schema, by default the name of the passed object

```javascript
addSchema(schemaObj);
```

* `schemaObj` - [schema](#schema) object we want to add

```javascript
addParser(parser, name);
```

Adds a parser that can be used in schemas, where:

* `parser` is a method
* `name` is the name under which parser will be available

## Schema

Schema is a basic structure, that allows you to easily define your desired object. You can provide them as `json` files, for instance:

```json
{
  "name": "User",
  "properties": [
    { "name": "name", "type": "string" },
    { "name": "address", "type": "string", "groups": ["admin"] },
    {
      "name": "money",
      "type": "number",
      "parsers": ["currency"],
      "groups": ["admin"]
    },
    {
      "name": "friends",
      "type": "array",
      "properties": [
        { "name": "name", "type": "string" },
        { "name": "profession", "type": "string", "groups": ["admin"] },
        { "name": "salary", "type": "number", "parsers": ["currency"] }
      ]
    }
  ]
}
```

Each schema should contain a `name`, which will be its identifier, and `properties` which should be an array of fields of the defined schema.

All properties that are not defined in the schema, will be excluded from the serialized data result.

If you are using TypeScript, you can just implement the exported interface `ISerializationSchema`:

```typescript
interface ISerializationSchema {
  name: string;
  properties: IProperty[];
}
```

## Properties

Each property contains the following fields:

* `name` - (required) name of the field
* `type` - (required) one of the following types:
  * `string`
  * `number`
  * `bool`
  * `array`
  * `object`
* `groups` - array of strings, that will define access to this field ([link](#groups)). If empty, the field will be considered public and will be always returned.
* `parsers` - array of parsers' names, that should be run on this field before it's returned.
* `properties` - array of properties, that are required in case of the type `object` or `array`
* `serializedName` - name of the field after serialization

If you are using TypeScript, you can just implement the exported interface `IProperty`:

```typescript
interface IProperty {
  name: string;
  type: string;
  serializedName?: string;
  groups?: string[];
  parsers?: string[];
  properties?: IProperty[];
}
```

## Groups

While defining a schema, you can add the `groups` parameter to properties. That way, while serializing data, you can specify the serialization group.

```javascript
const schema = {
  name: 'User',
  properties: [
    // ...
    { name: 'firstname', type: 'string' },
    { name: 'lastname', type: 'string', groups: ['friends'] }
  ],
}
serializer.addSchema(schema);

// ...

const data = {
  firstname: 'John',
  lastname: 'Doe',
  id: 481,
};

console.log(serializer.serialize(data, [], 'User');
// { 'firstname': 'John' }

console.log(serializer.serialize(data, ['friends'], 'User');
// { 'firstname': 'John', 'lastname': 'Doe' }
```
