# A POWERFUL INTERFACE TO YOUR API

Swagger is a simple yet powerful representation of your RESTful API. With the largest ecosystem of API tooling on the planet, thousands of developers are supporting Swagger in almost every modern programming language and deployment environment. With a Swagger-enabled API, you get interactive documentation, client SDK generation and discoverability.

We created Swagger to help fulfill the promise of APIs. Swagger helps companies like Apigee, Getty Images, Intuit, LivingSocial, McKesson, Microsoft, Morningstar, and PayPal build the best possible services with RESTful APIs.

Now in version 2.0, Swagger is more enabling than ever. And it's 100% open source software.

## Get Stated

If you're an API provider and want to use Swagger to describe your APIs - there are several approaches available:

A top-down approach where you would use the Swagger Editor to create your Swagger definition and then use the integrated Swagger Codegen tools to generate server implementation.
A bottom-up approach where you have an existing REST API for which you want to create a Swagger definition. Either you create the definition manually (using the same Swagger Editor mentioned above), or if you are using one of the supported frameworks (JAX-RS, node.js, etc), you can get the Swagger definition generated automatically for you. If you're doing JAX-RS have a look at the example at https://github.com/swagger-api/swagger-core/wiki/Swagger-Core-JAX-RS-Project-Setup-1.5.X.
If on the other hand you're an API Consumer who wants to integrate with an API that has a Swagger definition you can use the online version of the Swagger UI to explore the API (given that you have a URL to the APIs Swagger definition) - and then use Swagger Codegen to generate the client library of your choice.

In either case - be sure to check out the long list of both open source projects and commercial vendors that are available for Swagger - perhaps there is something there targeting your specific needs.


## Swagger Tools

### The major core tools are:

- Swagger Core	Java-related libraries for generating and reading Swagger definitions
- Swagger Codegen	Command-line tool for generating both client and server side code from a Swagger definition
- Swagger UI	Browser based UI for exploring a Swagger defined API
- Swagger Editor	Browser based editor for authoring Swagger definitions in YAML or JSON format

### Adjacent tools also provided by the Swagger project are

- Swagger JS	Javascript library for consuming Swagger-defined APIs from both client and node.js applications
- Swagger Node	Swagger module for node.js
- Swagger-Socket	expose/invoke Swagger defined APIs over WebSockets
- Swagger Parser	Standalone library for parsing Swagger definitions from Java