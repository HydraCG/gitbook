# Create a movie

## Story

As an API client I want to create a new movie resource

{% hint style='info' %}
Several possibilities exist, to define operations with Hydra. In this example we will describe a
custom type `mov:Movies` in the API documenation and add a supported operation there.
{% endhint %}

## API behaviour

```http
GET https://hydra-movies.herokuapp.com/
```

Response body (excerpt):

```json
{
  "@id": "/",
  "collection": {
    "@id": "movies",
    "@type": [
      "Collection",
      "mov:Movies"
    ]
  }
}
```

Api documentation (excerpt):

```json
{
  "@id": "/doc",
  "supportedClass": [
    {
      "@id": "mov:Movies",
      "@type": "Class",
      "supportedOperation": {
        "@id": "mov:create-movie",
        "@type": [
          "Operation",
          "schema:CreateAction"
        ],
        "title": "Create movie",
        "description": "Creates a new movie",
        "method": "POST",
        "expects": "schema:Movie",
        "returns": "schema:Movie"
      }
    },
    {
      "@id": "schema:Movie",
      "@type": "Class",
      "title": "Movie",
      "description": "A single movie",
      "supportedProperty": [
        {
          "property": "schema:name",
          "title": "name",
          "description": "The movie's name"
        },
        {
          "property": "schema:description",
          "title": "description",
          "description": "Textual description of the movie."
        },
        {
          "property": "schema:duration",
          "title": "duration",
          "description": "The movie's duration in ISO 8601 date format."
        }
      ]
    }
  ]
}
```

```http
POST https://hydra-movies.herokuapp.com/movies
Content-Type: application/ld+json

{
  "@context": "https://schema.org/",
  "@type": "Movie",
  "name": "Pulp Fiction",
  "description": "Pulp Fiction is a 1994 American crime film. It tells several stories of criminal Los Angeles.",
  "duration": "PT2H34M"
}
```

Response (excerpt):

```http
HTTP/1.1 201 Created
Location: /movies/d517cae6-6cdc-11e9-a923-1681be663d3e
Content-Type: application/ld+json

{
  "@context": "https://hydra-movies.herokuapp.com/context.jsonld",
  "@type": "schema:Movie",
  "@id": "/movies/d517cae6-6cdc-11e9-a923-1681be663d3e",
  "schema:name": "Pulp Fiction"
}
```


### Explanation

```
"collection": {
    "@id": "movies",
    "@type": [
      "Collection",
      "mov:Movies"
    ]
  }
```

By loading the API entry point a client came across a resource of type `mov:Movies`. This API includes does not include any inline hypermedia controls within the resource representation, so the client has to consult the API documentation to understand what the resource is capable of.

```
"supportedClass": [
  {
    "@id": "mov:Movies",
    "@type": "Class",
```

The type can be found within `supportedClass`.

```
"supportedOperation": {
  "@id": "mov:create-movie",
  "@type": [
    "Operation",
    "schema:CreateAction"
  ],
  "title": "Create movie",
  "description": "Creates a new movie",
```

The `supportedOperation` of `mov:Movies` describe what can be done on resources of that type. `Operation` is the hydra type for any API operation. By adding `schema:CreateAction` we can be more precise and define it as an operation that creates new resources. `title` and `description` allow a friendly human-readable output in generated developer documentation and generic clients.

```
"method": "POST",
"expects": "schema:Movie",
"returns": "schema:Movie"
```

The `method` is exactly telling the client, that it should use the HTTP POST verb to use this operation. `expects` and `returns` specify which types the operation expects as input and returns in response.

```
{
    "@id": "schema:Movie",
    "@type": "Class",
    "title": "Movie",
    "description": "A single movie",
```

This type is also present in the API documentation as a `supportedClass`. Again `title` and `description` give some info for humans.

```
"supportedProperty": [
  {
    "property": "schema:name",
    "title": "name",
    "description": "The movie's name"
  },
```

To construct an HTTP request the client needs to know which properties are supported by the expected type `schema:Movie`. The API documentation contains this information in `supportedProperty`. The client now is aware that it can send `schema:name`, `schema:description` and `schema:duration`.

{% hint style='info' %}
It is also possible to describe which properties are `required`, `readable` and `writable`. Further a property could define a `range` to clarify data type and cardinality. Those features will be addressed in another recipe.
{% endhint %}

```http
POST https://hydra-movies.herokuapp.com/movies
Content-Type: application/ld+json

{
  "@context": "https://schema.org/",
  "@type": "Movie",
  "name": "Pulp Fiction",
  "description": "Pulp Fiction is a 1994 American crime film. It tells several stories of criminal Los Angeles.",
  "duration": "PT2H34M"
}
```

With the information about the `method` and the `@id` from the `mov:Movies` resource, the client is able to perform the correct request against the API. In the body it sends the expected type `schema:Movie` with the properties defined by `supportedProperties`.

```http
HTTP/1.1 201 Created
Location: /movies/d517cae6-6cdc-11e9-a923-1681be663d3e
Content-Type: application/ld+json

{
  "@context": "https://hydra-movies.herokuapp.com/context.jsonld",
  "@type": "schema:Movie",
  "@id": "/movies/d517cae6-6cdc-11e9-a923-1681be663d3e",
  "schema:name": "Pulp Fiction"
}
```

The server responds with a 201 status code and a `Location` header pointing to the new movie resource. It also contains a representation of the new movie in the response body.

{% hint style='info' %}
As you might have seen, client and server can use different JSON-LD contexts. This is OK, as long as both stick to the semantics described by the API documentation.
{% endhint %}
