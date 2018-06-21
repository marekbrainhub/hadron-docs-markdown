## Installation

```bash
npm install @brainhubeu/hadron-validation --save
```

[More info about installation](/core/#installation)

## Creating schema files

To use the validation layer, first you need to provide some schemas. We use JSON Schema format, for example:

```json
{
  "type": "object",
  "properties": {
    "id": {
      "type": "number"
    },
    "name": {
      "type": "string"
    },
    "age": {
      "type": "number"
    }
  },
  "required": ["name", "age"],
  "additionalProperties": false
}
```

Full documentation about JSON Schema: [Ajv documentation](https://epoberezkin.github.io/ajv/)

## Preparing schema files for usage with Hadron

When your schemas are ready you need to build an object out of them, where **key name** is a name of the schema, for example:

```js
// schemas.js

const insertTeam = require('./team/insertTeam.json');
const updateTeam = require('./team/updateTeam.json');
const insertUser = require('./user/insertUser.json');
const updateUser = require('./user/updateUser.json');

const schemas = {
  insertTeam,
  updateTeam,
  insertUser,
  updateUser,
};

module.exports = schemas;
```

## Validate function

After you create the schemas object you can create a validate function from the **Hadron validator factory**

```js
// validate.js

const validatorFactory = require('@brainhubeu/hadron-validation');
const schemas = require('./schemas');

module.exports = validatorFactory(schemas);
```

## Using a validate function

After you've created a validate function with **hadron validator factory** you can use it to validate an object by the schemas you provide.

```js
const validObject = {
  name: 'Max',
  age: 22,
};

validate('schemaName', objectToValidate)
  .then((validObject) => {
    console.log('I am a valid object', validObject);
  })
  .catch((error) => {
    console.log('Object is invalid', error);
  });
```

The validate function returns a valid object, otherwise it throws an error.
