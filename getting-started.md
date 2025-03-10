---
layout: default
title: Getting Started
---
# Getting Started

Getting started is easy, here's a suggested order of actions to take.

1. Say, "Hi 👋!" on the [project
   discussions](https://github.com/orgs/gost-dom/discussions/7)
   - Obviously, this is purely optional, but knowing about the types of projects
     this is used for helps setting the direction and prioritize features.
2. Run a recent Go release. This is tested on Go 1.23, but will eventually
   require 1.24.
3. Add Gost-DOM to your problem `go get github.com/gost-dom/browser`
4. Write a test for the next feature in your web application
5. Make the test pass
6. Go to 4.

## A simple example

### Your server

It is assumed you have a web server serving HTML with JavaScript, and there's a
root `http.Handler` exported. The one you would pass to a function like
`http.ListenAndServe`.

```go
package server

import "net/http"

var RootHandler http.Handler

func init() {
    mux := http.NewServeMux()
    configureRoutes(mux)
    RootHandler = mux
}
```

It is strongly adviced to keep the _creation and configuration_ of the root
handler separated from the actual starting of the server and listening on a TCP
port. Gost-DOM doesn't require you to start the server, as it can call your
`http.Handler` directly.

This minimizes latency and removes the hassle of managing ports.

```go
package server_test

import (
    "testing"

    "myapp/server"

    "github.com/gost-dom/browser"
)

func TestWebBrowser(t *testing.T) {
    b := browser.NewFromHandler(server.RootHttpHandler)
    window, err := b.Open("http://example.com/")
    assert.NoError(t, err)
    // Interact with the document
    win.Document().GetElementById("test-button").Click()
    // Inspect the state of the document
    resultField := win.Document().GetElementById("output-element")
    assert.Equal(t, "The button was clicked", resultField.TextContent())
}
```

**Note:** When creating a browser with an HTTP handler instance, no _real_ TCP
requests will be created. This means any JavaScript used must be served locally;
CDN will not work.

## The API

The `window` returned from `Browser.Open` is an implementation of a [HTML DOM
API Window](https://developer.mozilla.org/en-US/docs/Web/API/Window). The
methods correspond to operations and attributes specified in the Web API
specifications; adapted to Go.

- Functions start with upper-case letters, to be exported.
- Attribute getters become functions with the same name. E.g., `form.method`
     becomes `form.Method()`
- Attribute setters becomes functions prefixed with `Set`. `form.method =
  "post"` becomes `form.SetMethod("post")`
- Functions thay may throw an error in JavaScript return an extra `error` value.
  E.g., `querySelect` throws an error if the pattern is invalid. Go's version
  has two return values `QuerySelector(pattern string) (Element, error)`.

So your existing knowledge about navigating and manipulating the DOM applies to
Gost as well.
