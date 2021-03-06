# Light Efficient Server - for creating servers on Dart. Inspired by ExpressJS and Koa

Created under a MIT-style
[license](https://github.com/YevhenKap/les/blob/master/LICENSE).

## Overview

The server builds by creating instance of `Server` class. It have few methods for adding `Middleware`s and `Route`s to handling incoming requests.

You can create server that is listening for requests over default `http` scheme, over `https` (secure) scheme or
via socket.

```dart
Server() // or Server.secure(...) or Server.socket(...)
    ..use(...) // Add middlewares
    ..add(...) // Add routes
    ..listen(2000); // Start listen to requests
```

### Route

One of the main concept of library is `Route` class that  describes the response for a particular path.
It accepts path, function that describe how to create response for such request, optionally method of the request and middlewares.

```dart
Route(
    '/',
    (Context ctx) => ctx.send('Hello world'),
    method: HttpMethod.get // default. No need to be provided
);
```

If exist many routes that starts with the same path prefix, then you can provide a `Router`. Its just a collection of `Route`s for paths that start with identical prefix. Or you can use `Router` if you need handle requests with the same paths, but different http methods.

```dart
// Define [Route]s that describe answers for concrete requests
final routeOne = Route(
        r'/:a(\d+)', // Will match /identical/5, /83645, but not /identical/word or other symbols
        (ctx) => ctx.send('Hello get')
      );
final routeTwo = Route(
      '/b',
      (ctx) => ctx.send('Hello post'),
      method: HttpMethod.post
      )
final routeThree = Route(
      '', // If you left empty path, then it will be sets to `Router`'s /identical
      (ctx) => ctx.send('Hello put'),
      method: HttpMethod.put
      )
final routes = Router('/identical');
  ..addAll(<Route>[routeOne, routeTwo, routeThree]);
```

All paths that are defined in `Route` or `Router` transforms to `RegExp` and parses by [path_to_regexp](https://pub.dartlang.org/packages/path_to_regexp) package. For more info about constructing and managing paths see its documentation. Thanks to this you can provide parameter variable in path and sets to it some bounds.

### Middleware

`Middleware` is like a `Route` controller, except that it isn't send responses to client, but just process request before routes does. It is useful for processing some work before sending a response. **Middleware must not sends a response to the client!**

```dart
Middleware(
    (Context ctx) => // do come work over ctx object
);
```

Middlewares can be provided before each route in `Server`, before group of routes in `Router` or before specific `Route`.

### Context

The other main concept of this library is `Context` class. It contains the request object and build a response object. Handlers of `Route` and `Middleware` receive this object. It has various convenient methods for sending a responses and properties for reading request details or construct response (headers, ...). For details about methods and properties see `dartdoc`.

### Predefined middlewares

1. **bodyParser**: with library ships `bodyParser` middleware that process context object and parses its body if it exists. The content body is parsed, depending on the `Content-Type` header field. When the full body is read and parsed the body content is made available.

The following content types are recognized:

- text/*
- application/json
- application/x-www-form-urlencoded
- multipart/form-data

For content type `text/*` the body is decoded into a `String`. The 'charset' parameter of the content type specifies the encoding used for decoding. If no 'charset' is present the default encoding of ISO-8859-1 is used.

For content type `application/json` the body is decoded into a string which is then parsed as JSON. The resulting body is a `Map`. The 'charset' parameter of the content type specifies the encoding used for decoding. If no 'charset' is present the default encoding of UTF-8 is used.

For content type `application/x-www-form-urlencoded` the body is a query string which is then split according to the rules for splitting a query string. The resulting body is a `Map<String, String>`. If the same name is present several times in the query string, then the last value seen for this name will be in the resulting map. The encoding US-ASCII is always used for decoding the body.

For content `type multipart/form-data` the body is parsed into it's different fields. The resulting body is a `Map<String, dynamic>`, where the value is a `String` for normal fields and a `HttpBodyFileUpload` instance for file upload fields. If the same name is present several times, then the last value seen for this name will be in the resulting map.

When using content type `multipart/form-data` the encoding of fields with String values is determined by the browser sending the HTTP request with the form data. The encoding is specified either by the attribute `accept-charset` on the HTML form, or by the content type of the web page containing the form. If the HTML form has an `accept-charset` attribute the browser will use the encoding specified there. If the HTML form has no `accept-charset` attribute the browser determines the encoding from the content type of the web page containing the form. Using a content type of `text/html; charset=utf-8` for the page and setting `accept-charset` on the HTML form to utf-8 is recommended as the default for `HttpBodyHandler` is UTF-8. It is important to get these encoding values right, as the actual `multipart/form-data` HTTP request sent by the browser does not contain any information on the encoding.

For all other content types the body will be treated as uninterpreted binary data. The resulting body will be of type `List<int>`.

```dart
Server()
    ..use(bodyParser) // Add bodyParser
    ..use(...) // Add middlewares
    ..add(...) // Add routes
    ..listen(2000); // Start listen to requests
```

2. **staticFilesHandler**: builds static files handler for provided path to `static`(default) folder.
Path starts from the root of project and in that level must be folder that contains all static files. All files will be stored in `Context` object for current session. Note that context doesn't store actual files, instead it will contain
`File` objects with some info about actual file. You must read and convert that files by yourself.
If directory isn't exist context object will have empty `Map<String, File>` object.

```dart
Server()
    ..use(bodyParser) // Add bodyParser
    ..use(buildStaticFilesHandler()) // Add staticFilesHandler
    ..use(...) // Add middlewares
    ..add(...) // Add routes
    ..listen(2000); // Start listen to requests
```

3. **cors**: sets appropriate headers to response and enable CORS.

```dart
Server()
    ..use(bodyParser) // Add bodyParser
    ..use(buildStaticFilesHandler()) // Add staticFilesHandler
    ..use(cors()) // Add cors
    ..use(...) // Add other middlewares
    ..add(...) // Add routes
    ..listen(2000); // Start listen to requests
```

## Usage

See [example.](example/example.dart)

## Features and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: https://github.com/YevhenKap/les/issues
