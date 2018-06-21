## Installation

```bash
npm install @brainhubeu/hadron-typeorm --save
```

[More info about installation](/core/#installation)

## Initializing

Pass the package as an argument for the Hadron bootstrapping function:

```javascript
const hadronTypeOrm = require('@brainhubeu/hadron-typeorm');
// ... importing and initializing other components

hadron(expressApp, [hadronTypeOrm], config).then(() => {
  console.log('Hadron with typeORM initialized');
});
```

## Connecting to a database

You can set up a new connection using [connection object](http://typeorm.io/#/connection).

```javascript
{
  connectionName: string,
  type: string,
  host: string,
  port: number,
  username: string,
  password: string,
  database: string
  entitySchemas: entities,
  synchronize: bool,
}
```

* `connectionName` - string that identifies this connection
* `type` - string that defines the type of database, e.g., mysql, mariadb, postgres, sqlite, mongodb
* `host` - url to the database
* `port` - port of the database
* `username` - username of the account in the databse
* `password` - password to the database,
* `database` - name of the database
* `entities` - array of classes that defines models
* `entitySchemas` - if you are describing models with schemas, put those in this parameter
* `synchronize` - parameter that defines if the database should be automatically synchronized with models

Also all other parameters available in TypeORM are available. Please take a look at the [TypeORM documentation](https://github.com/typeorm/typeorm#creating-a-connection-to-the-database)

## Including the database connection in Hadron

_NOTE: Events aren't included in this section so logging into the console is done using setTimeout._

Since we have our connection, we need to include it inside our config object.

```javascript
const hadronTypeOrm = require('@brainhubeu/hadron-typeorm');

const config = {
  connection: {
    /* connection object */
  },
};

hadron(expressApp, [hadronTypeOrm], config).then((container) => {
  console.log('Initialized connection:', container.take('connection'));
});
```

## Entities

Let's assume we want to have a simple table **user**.

| Field     | Type    |
| --------- | ------- |
| 🔑 id     | int     |
| firstName | varchar |
| lastName  | varchar |

We have two options while defining our `entities`.

### Class + Decorators

```typescript
import { Entity, Column, PrimaryColumn } from 'typeorm';

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  @Column()
  id: number;

  @Column() firstName: string;

  @Column() lastName: string;
}
```

When using this method, while creating the connection to the database, those classes should be in the `entities` parameter.

### Schema Way

```javascript
// entity/User.js

module.exports = {
  name: 'User',
  columns: {
    id: {
      primary: true,
      type: 'int',
      generated: true,
    },
    firstName: {
      type: 'varchar',
    },
    lastName: {
      type: 'varchar',
    },
  },
};
```

When using this method, while creating the connection to the database, those schemas should be in the `entitySchemas` parameter.

For more details about defining models, please take a look at the [TypeORM documentation](http://typeorm.io/#/entities). Especially the section about [available types](http://typeorm.io/#/entities/column-types) for each database distribution.

## Injecting entities into Hadron

To include our entities in Hadron, we simply need to include them in our config object.
Let's modify the code that we were using to initialize Hadron:

```javascript
const hadronTypeOrm = require('@brainhubeu/hadron-typeorm');
const user = require('./entity/User');

const config = {
  connection,
  entities: [user],
};

hadron(expressApp, [hadronTypeOrm], config).then((container) => {
  console.log(
    'userRepository available:',
    container.take('userRepository') !== null,
  );
  // User entity should be declared under userRepository key and
  // will be available as a typeORM repository
});
```

Repository keys in the container derive from names of schemas/classes and are built this way:
`<schema/class name in lower case>Repository`

Examples:

```
User = userRepository
SuperUser = superuserRepository
loremIpsumDolor = loremipsumdolorRepository
```

## Repositories

Generated repositories contain the same methods as the ones from TypeORM. You can check them out here:

[http://typeorm.io/#/working-with-entity-manager](http://typeorm.io/#/working-with-entity-manager)

## Troubleshooting

### I can't connect to database:

* Make sure that the connection config contains valid data and there is an existing database with the specified name.

### There are no tables in my database

* There are a few possible reasons for that. First, check if the parameter `synchronize` in configuration is set to true.

* Then make sure that the connection configuration contains an `entities`/`entitySchemas` field.

* Remember, if you are using the class definition of models, you need to put them in the `entities` parameter, otherwise (schema method) in `entitySchemas`.

### There is information that I am missing a driver

* If you decided which database you want to use, you need to add a proper driver to your dependencies. For more details check TypeORM [README](https://github.com/typeorm/typeorm#installation) file.
