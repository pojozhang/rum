<img align="middle" height="200px" src="logo.png">

![GitHub (pre-)release](https://img.shields.io/github/release/pojozhang/sugar/all.svg)
[![Build Status](https://travis-ci.org/pojozhang/sugar.svg?branch=master)](https://travis-ci.org/pojozhang/sugar) [![codecov](https://codecov.io/gh/pojozhang/sugar/branch/master/graph/badge.svg)](https://codecov.io/gh/pojozhang/sugar) [![Go Report Card](https://goreportcard.com/badge/github.com/pojozhang/sugar)](https://goreportcard.com/report/github.com/pojozhang/sugar) [![GoDoc](https://godoc.org/github.com/pojozhang/sugar?status.svg)](https://godoc.org/github.com/pojozhang/sugar) 
![license](https://img.shields.io/github/license/pojozhang/sugar.svg)


Sugar is a **DECLARATIVE** http client providing elegant APIs for Golang.

### [中文文档](README.zh-cn.md)

## Features
- Elegant APIs
- New plugin API
- Chained invocations
- Highly extensible

## Download
```bash
dep ensure -add github.com/pojozhang/sugar
```

## Usage
Firstly you need to import the package.
```go
import . "github.com/pojozhang/sugar"
```
And now you are ready to easily send any request to any corner on this blue planet.

### Request
#### Plain Text
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Content-Type: text/plain
Post("http://api.example.com/books", "bookA")
```

#### Path
```go
// GET /books/123 HTTP/1.1
// Host: api.example.com
Get("http://api.example.com/books/:id", Path{"id": 123})
Get("http://api.example.com/books/:id", P{"id": 123})
```

#### Query
```go
// GET /books?name=bookA HTTP/1.1
// Host: api.example.com
Get("http://api.example.com/books", Query{"name": "bookA"})
Get("http://api.example.com/books", Q{"name": "bookA"})

// list
// GET /books?name=bookA&name=bookB HTTP/1.1
// Host: api.example.com
Get("http://api.example.com/books", Query{"name": List{"bookA", "bookB"}})
Get("http://api.example.com/books", Q{"name": L{"bookA", "bookB"}})
```

#### Cookie
```go
// GET /books HTTP/1.1
// Host: api.example.com
// Cookie: name=bookA
Get("http://api.example.com/books", Cookie{"name": "bookA"})
Get("http://api.example.com/books", C{"name": "bookA"})
```

#### Header
```go
// GET /books HTTP/1.1
// Host: api.example.com
// Name: bookA
Get("http://api.example.com/books", Header{"name": "bookA"})
Get("http://api.example.com/books", H{"name": "bookA"})
```

#### Json
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Content-Type: application/json;charset=UTF-8
// {"name":"bookA"}
Post("http://api.example.com/books", Json{`{"name":"bookA"}`})
Post("http://api.example.com/books", J{`{"name":"bookA"}`})

// map
Post("http://api.example.com/books", Json{Map{"name": "bookA"}})
Post("http://api.example.com/books", J{M{"name": "bookA"}})

// list
Post("http://api.example.com/books", Json{List{Map{"name": "bookA"}}})
Post("http://api.example.com/books", J{L{M{"name": "bookA"}}})
```

#### Xml
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Authorization: Basic dXNlcjpwYXNzd29yZA==
// Content-Type: application/xml; charset=UTF-8
// <book name="bookA"></book>
Post("http://api.example.com/books", Xml{`<book name="bookA"></book>`})
Post("http://api.example.com/books", X{`<book name="bookA"></book>`})
```

#### Form
```go
// POST /books HTTP/1.1
// Host: api.example.com
// Content-Type: application/x-www-form-urlencoded
Post("http://api.example.com/books", Form{"name": "bookA"})
Post("http://api.example.com/books", F{"name": "bookA"})

// list
Post("http://api.example.com/books", Form{"name": List{"bookA", "bookB"}})
Post("http://api.example.com/books", F{"name": L{"bookA", "bookB"}})
```

#### Basic Auth
```go
// DELETE /books HTTP/1.1
// Host: api.example.com
// Authorization: Basic dXNlcjpwYXNzd29yZA==
Delete("http://api.example.com/books", User{"user", "password"})
Delete("http://api.example.com/books", U{"user", "password"})
```

#### Multipart
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
Post("http://api.example.com/books", MultiPart{"name": "bookA", "file": f})
Post("http://api.example.com/books", MP{"name": "bookA", "file": f})
```

#### Mix
Due to Sugar's flexible design, different types of parameters can be freely combined.
```go
Patch("http://api.example.com/books/:id", Path{"id": 123}, Json{`{"name":"bookA"}`}, User{"user", "password"})
```

#### Apply
You can use Apply() to preset some values which will be attached to every following request. Call Reset() to clean preset values.
```go
Apply(User{"user", "password"})
Get("http://api.example.com/books")
Get("http://api.example.com/books")
Reset()
Get("http://api.example.com/books")
```
```go
Get("http://api.example.com/books", User{"user", "password"})
Get("http://api.example.com/books", User{"user", "password"})
Get("http://api.example.com/books")
```
The latter is equal to the former.


### Response
A request API always returns a value of type `*Response` which also provides some nice APIs.

#### Raw
Raw() returns a value of type `*http.Response` and an `error` which is similar to standard go API.
```go
resp, err := Post("http://api.example.com/books", "bookA").Raw()
...
```

#### ReadBytes
ReadBytes() is another syntax sugar to read bytes from response body.
Notice that this method will close body after reading.
```go
bytes, resp, err := Get("http://api.example.com/books").ReadBytes()
...
```

#### Read
Read() reads different types of response via decoder API.
The following two examples read response body as plain text/JSON according to different content types.
```go
// plain text
var text = new(string)
resp, err := Get("http://api.example.com/text").Read(text)

// json
var books []book
resp, err := Get("http://api.example.com/json").Read(&books)
```

#### Download files
You can also use Read() to download files.
```go
f,_ := os.Create("tmp.png")
defer f.Close()
resp, err := Get("http://api.example.com/logo.png").Read(f)
```

## Extension
There are three major components in Sugar: **Encoder**, **Decoder** and **Plugin**.
- A encoder is used to encode your parameters and assemble requests.
- A decoder is used to decode the data from response body.
- A plugin is designed to work as an interceptor.

### Encoder
You can register your custom encoder which should implement `Encoder` interface.
```go
type MyEncoder struct {
}

func (r *MyEncoder) Encode(context *RequestContext, chain *EncoderChain) error {
    myParams, ok := context.Param.(MyParam)
    if !ok {
	return chain.Next()
    }
    ...
    req := context.Request
    ...
    return nil
}

Default.Encoders.Add(&MyEncoder{})

Get("http://api.example.com/books", MyParam{})
```

### Decoder
You can implement `Decoder` interface so that you can convert a response body to a specific struct.
It is very convenient to get converted value via `func (r *Response) Read(v interface{}) (*http.Response, error)` API.
```go
type MyDecoder struct {
}

func (d *MyDecoder) Decode(context *ResponseContext, chain *DecoderChain) error {
    // decode data from body if a content type named `my-content-type` is set in header
    for _, contentType := range context.Response.Header[ContentType] {
	if strings.Contains(strings.ToLower(contentType), "my-content-type") {
	    body, err := ioutil.ReadAll(context.Response.Body)
	    if err != nil {
		return err
	    }
	    json.Unmarshal(body, context.Out)
	    ...
	    return nil
	}
    }
    return chain.Next()
}

Default.Decoders.Add(&MyDecoder{})
```

### Plugin
Plugin is a new feature since V2. You can do anything as you like before the request is sent or after the response is received.
```go
// Implementation of builtin Logger plugin
Use(func(c *Context) error {
    b, _ := httputil.DumpRequest(c.Request, true)
    log.Println(string(b))
    defer func() {
        if c.Response != nil {
	    b, _ := httputil.DumpResponse(c.Response, true)
	    log.Println(string(b))
	}
    }()
    return c.Next()
})
```
