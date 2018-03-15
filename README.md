# Sugar2 [![Build Status](https://travis-ci.org/pojozhang/sugar.svg?branch=v2)](https://travis-ci.org/pojozhang/sugar) [![codecov](https://codecov.io/gh/pojozhang/sugar/branch/master/graph/badge.svg)](https://codecov.io/gh/pojozhang/sugar)

### [中文文档](http://www.jianshu.com/p/7ca4fa63460b)

Sugar is a **DECLARATIVE** http client providing elegant APIs for Golang.

Now you can send requests in just one line.


## Set Up
```bash
dep ensure -add github.com/pojozhang/sugar
```

## Usage

### Plain Text
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Content-Type: text/plain
sugar.Post("http://api.example.com/books", "bookA")
```

### Path
```go
// GET /books/123 HTTP/1.1
// Host: api.example.com
sugar.Get("http://api.example.com/books/:id", Path{"id": 123})
sugar.Get("http://api.example.com/books/:id", P{"id": 123})
```

### Query
```go
// GET /books?name=bookA HTTP/1.1
// Host: api.example.com
sugar.Get("http://api.example.com/books", Query{"name": "bookA"})
sugar.Get("http://api.example.com/books", Q{"name": "bookA"})

// list
// GET /books?name=bookA&name=bookB HTTP/1.1
// Host: api.example.com
sugar.Get("http://api.example.com/books", Query{"name": List{"bookA", "bookB"}})
sugar.Get("http://api.example.com/books", Q{"name": L{"bookA", "bookB"}})
```

### Cookie
```go
// GET /books HTTP/1.1
// Host: api.example.com
// Cookie: name=bookA
sugar.Get("http://api.example.com/books", Cookie{"name": "bookA"})
sugar.Get("http://api.example.com/books", C{"name": "bookA"})
```

### Header
```go
// GET /books HTTP/1.1
// Host: api.example.com
// Name: bookA
sugar.Get("http://api.example.com/books", Header{"name": "bookA"})
sugar.Get("http://api.example.com/books", H{"name": "bookA"})
```

### Json
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Content-Type: application/json;charset=UTF-8
// {"name":"bookA"}
sugar.Post("http://api.example.com/books", Json{`{"name":"bookA"}`})
sugar.Post("http://api.example.com/books", J{`{"name":"bookA"}`})

// map
sugar.Post("http://api.example.com/books", Json{Map{"name": "bookA"}})
sugar.Post("http://api.example.com/books", J{M{"name": "bookA"}})

// list
sugar.Post("http://api.example.com/books", Json{List{Map{"name": "bookA"}}})
sugar.Post("http://api.example.com/books", J{L{M{"name": "bookA"}}})
```

### Xml
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Authorization: Basic dXNlcjpwYXNzd29yZA==
// Content-Type: application/xml; charset=UTF-8
// <book name="bookA"></book>
sugar.Post("http://api.example.com/books", Xml{`<book name="bookA"></book>`})
sugar.Post("http://api.example.com/books", X{`<book name="bookA"></book>`})
```

### Form
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Content-Type: application/x-www-form-urlencoded
sugar.Post("http://api.example.com/books", Form{"name": "bookA"})
sugar.Post("http://api.example.com/books", F{"name": "bookA"})

// list
sugar.Post("http://api.example.com/books", Form{"name": List{"bookA", "bookB"}})
sugar.Post("http://api.example.com/books", F{"name": L{"bookA", "bookB"}})
```

### Basic Auth
```go
// DELETE /books HTTP/1.1
// Host: api.example.com
// Authorization: Basic dXNlcjpwYXNzd29yZA==
sugar.Delete("http://api.example.com/books", User{"user", "password"})
sugar.Delete("http://api.example.com/books", U{"user", "password"})
```

### Multipart
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Content-Type: multipart/form-data; boundary=19b8acc2469f1914a24fc6e0152aac72f1f92b6f5104b57477262816ab0f
//
// --19b8acc2469f1914a24fc6e0152aac72f1f92b6f5104b57477262816ab0f
// Content-Disposition: form-data; name="name"
//
// bookA
// --19b8acc2469f1914a24fc6e0152aac72f1f92b6f5104b57477262816ab0f
// Content-Disposition: form-data; name="file"; filename="text"
// Content-Type: application/octet-stream
//
// hello sugar!
// --19b8acc2469f1914a24fc6e0152aac72f1f92b6f5104b57477262816ab0f--
f, _ := os.Open("text")
sugar.Post("http://api.example.com/books", MultiPart{"name": "bookA", "file": f})
```

### Mix
Due to Sugar's flexible design, different types of parameters can be freely combined.

```go
sugar.Patch("http://api.example.com/books/:id", Path{"id": 123}, Json{`{"name":"bookA"}`}, User{"user", "password"})
```

### Apply
You can use Apply() to preset some values which will be attached to every following request. Call Reset() to clean preset values.

```go
sugar.Apply(User{"user", "password"})
sugar.Get("http://api.example.com/books")
sugar.Get("http://api.example.com/books")
```
```go
sugar.Get("http://api.example.com/books", User{"user", "password"})
sugar.Get("http://api.example.com/books", User{"user", "password"})
```
The latter is equal to the former.

### Mapper
This method allows you to change your request directly.
For example, if your project is running as a micro service, you may want to call a remote API via service name, like
```go
sugar.Get("http://book-service/books")
```

The problem is that `book-service` is not the real host and I'm sure you'll get an error.
The following codes show a good solution.
```go
sugar.Apply(Mapper{func(req *http.Request) {
    if req.URL.Host == "book-service" {
        req.URL.Host = "api.example.com"
    }
}})
resp, err := sugar.Get("http://book-service/books")
```

### Extension
You can register your custom resolver which should implement interface `Resolver` and bind it to the target type.  
```go
sugar.Register(Custom{}, &CustomResolver{})
sugar.Get("http://api.example.com/books", Custom{})
```

## License
Apache License, Version 2.0
