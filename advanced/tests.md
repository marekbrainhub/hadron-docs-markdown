## Unit testing

You can write unit tests using standard testing frameworks like Mocha + Chai, Jest etc.

There is nothing special about testing application logic so we won't address this topic.

### Testing controllers (route callbacks)

Hadron makes it easy to unit test your controllers - your input request data is just a data structure, as well as the returned request specification (optionally wrapped in a promise). You can also easily stub your dependencies by providing fake dependencies as objects in the second argument of the controller.

_Note: For our examples we will use Mocha + Chai_

---

Let's start with a simple controller with no external dependencies and no request-specific data - for example request for a specific view i.e. `/about`.

Code under test:

```javascript
export const getAboutPage = () => {
  return {
    view: {
      name: "about"
    }
  };
};
```

Test code:

```javascript
import { expect } from "chai";
import { getAboutPage } from "./staticViewHandlers";

describe("getAboutPage request handler", () => {
  it("returns response spec with proper view", () => {
    const expected = {
      view: {
        name: "about"
      }
    };

    const actual = getAboutPage();

    expect(actual).to.deep.equal(expected);
  });
});
```

---

In the next simple example we test a dumb handler that echos request parameters. To test it in isolation all we need to do is to pass a fake request structure and compare the result.

Code under test:

```javascript
export const echo = ({ params, query }) => {
  return {
    body: { params, query }
  };
};
```

Test code:

```javascript
import { expect } from "chai";
import { echo } from "./dumbHandlers";

describe("echo request handler", () => {
  it("returns response spec provided params", () => {
    const requestStub = {
      params: {
        foo: "bar"
      },
      query: {
        baz: "bat"
      }
    };

    const expected = {
      body: {
        params: {
          foo: "bar"
        },
        query: {
          baz: "bat"
        }
      }
    };

    const actual = echo(requestStub);

    expect(actual).to.deep.equal(expected);
  });
});
```

---

In another example we will request a callback that returns a specific entity selected by id provided by user in the request param. We will stub repository and test for the correct response spec. We will also cover the case when there is no entity with such an ID.

Code under test:

```javascript
import { UserNotFoundError } from "./errors";

export const fetchUser = async ({ params }, { userRepository }) => {
  try {
    return {
      body: await userRepository.byId(params.id)
    };
  } catch (error) {
    if (error instanceof UserNotFoundError) {
      return {
        status: 400,
        body: {
          message: `User with id ${params.id} not found`
        }
      };
    }

    throw error;
  }
};
```

Test code:

```javascript
import { expect } from "chai";
import { fetchUser } from "./userHandlers";
import { UserNotFoundError } from "./errors";

describe("fetchUser request handler", () => {
  it("returns response spec with user data", async () => {
    const requestStub = {
      params: {
        id: 1
      }
    };

    const dependenciesStub = {
      userRepository: {
        byId(id) {
          return Promise.resolve({
            id,
            name: "Stranger"
          });
        }
      }
    };

    const expected = {
      body: {
        id: 1,
        name: "Stranger"
      }
    };

    const actual = await fetchUser(requestStub, dependenciesStub);

    return expect(actual).to.deep.equal(expected);
  });

  it("returns response spec with error when there is no user with provided ID", async () => {
    const requestStub = {
      params: {
        id: 1
      }
    };

    const dependenciesStub = {
      userRepository: {
        byId(id) {
          return Promise.reject(new UserNotFoundError());
        }
      }
    };

    const expected = {
      status: 400,
      body: {
        message: "User with id 1 not found"
      }
    };

    const actual = await fetchUser(requestStub, dependenciesStub);

    return expect(actual).to.deep.equal(expected);
  });
});
```

---

In the last example in addition to returning the response spec we will perform a side effect and test that it was called. For that we will use spy. Instead of writing our own spy we will use the one included in Sinon.JS.

Code under test:

```javascript
export const fetchUsers = async (req, { userRepository, logger }) => {
  const users = await userRepository.all();

  logger.info(`Fetched ${users.length} users`);

  return {
    body: users
  };
};
```

Test code:

```javascript
import { expect } from "chai";
import { fetchUsers } from "./userHandlers";
import sinon from "sinon";

describe("fetchUsers request handler", () => {
  const dependenciesStub = {
    userRepository: {
      all() {
        return Promise.resolve([
          { id: 1, name: "Stranger1" },
          { id: 2, name: "Stranger2" },
          { id: 3, name: "Stranger3" }
        ]);
      }
    },
    logger: {
      info: sinon.spy()
    }
  };

  it("returns response spec with users data", async () => {
    const expected = {
      body: [
        { id: 1, name: "Stranger1" },
        { id: 2, name: "Stranger2" },
        { id: 3, name: "Stranger3" }
      ]
    };

    const actual = await fetchUsers({}, dependenciesStub);

    expect(actual).to.deep.equal(expected);
  });

  it("calls logger with result info", async () => {
    await fetchUsers({}, dependenciesStub);

    expect(dependenciesStub.logger.info.calledWith('Fetched 3 users')).to.be.true;
  });
});
```

## Integration tests

## End-to-end tests

To run end to end tests, use this command in terminal:

```sh
npm run test:e2e
```

### Scenarios

* **Given**

Setting header values

```feature
Given I set header "header-name" with value "header-value"

ex.
Given I set header "content-type" with value "application/json"
```

* **When**

Sending request

```feature
When I send a "METHOD" request to "/path"

ex.
When I send a "GET" request to "/"
When I send a "post" request to "/post/"
```

Sending request with body

```feature
When I send a "METHOD" request to "/path" with body:
  """
  {
    "name": "Wonderful coffee",
    "project": {
      "name": "Coffee"
    }
  }
  """

ex.
When i send a "PUT" request to "/users/add"
  """
  {
    "firstname": "John",
    "lastname": "Doe",
    "email": "john.doe@example.com"
  }
  """
```

* **Then**

Checking response code

```feature
Then the response code should be 200
```

Checking response body

```feature
Then the JSON should match pattern
  """
  {
    "name": "Wonderful coffee",
    "project": {
      "name": "Coffee"
    }
  }
  """
```

Example e2e tests:

```feature
Feature: Feature name

  Feature description

  Scenario: Simple GET request
    When I send a "GET" request to "/"
    Then the response code should be 200
```
