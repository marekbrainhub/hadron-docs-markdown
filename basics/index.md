Hadron is a high-level Node.js framework built on top of Express (with support for other micro frameworks coming in the future).

It abstracts away underlying request and response objects, providing simple data structures as input and output of your routes' handlers, making them simple to test and easy to deal with.

Thanks to using dependency injection containers as a central dependency management solution, it provides a convenient way to access all dependencies in handler functions.

Hadron is modular, in addition to core functionalities mentioned above we provide a complete solution for requests processing via separate packages:

- security management
- input validation
- database integration (through TypeORM)
- data serialization
- logging
- events handling
- CLI tool

Hadron is built with TypeScript, but it's primary target are JavaScript apps - we build our API to embrace current ECMAScript standards, with the cherry of good IDE support via codebase types declarations on top.
