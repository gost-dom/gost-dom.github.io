---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Home
---

<div class="d-flex flex-row align-items-center mb-5">
    <img src="/assets/gost-logo.svg" height="64" width="64" alt="Gost-DOM logo"/>
<div class="main-heading flex-fill">
<h1>Gost-DOM</h1>
The go-to solution for a web application TDD workflow
</div>
</div>

Gost-DOM is a headless browser written in Go to help build provide a fast
feedback loop of the HTTP for web application development in Go. It features
a DOM implemented in Go and a V8 JavaScript engine exposing the DOM to
client-scripting; as well as a subset of the [browser
APIs](https://developer.mozilla.org/en-US/docs/Web/API). 

Gost-DOM is specifically written with [HTMX](https://htmx.org/) in mind.

## How it works

Gost-DOM can consume the HTTP handler directly, bypassing the TCP stack,
avoiding the overhead, as well as complexity in test code for managing server
startup, and available ports.

You can construct an instance of the
[`Browser`](https://pkg.go.dev/github.com/gost-dom/browser#Browser) passing an
`http.Handler` as argument.[^1]

Any window opened from the browser will be instantiated with the _default_
script engine, currently V8.[^2] The window provides a subset[^3] of the
[DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model),
allowing developers to write the tests using a familiar syntax, the DOM.

<div class="card p-2 my-2" markdown="1">

```go
// server.go
var MyRootHttpServer = http.DefaultServeMux

func init() {
    http.HandleFunc("GET /", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("<body><h1>My title</h1><p>Lorem ipsum</p></body>"))
    })
}
```

</div>
<div class="card p-2 my-2" markdown="1">

```go
// server_test.go
func TestMyServer(t *testing.T) {
    browser := browser.New(MyRootHttpServer)
    win, err := browser.Open("/")
    // ignore error, means invalid CSS selector
    pageTitle, _ := win.Document().QuerySelector("h1")
    assert.Equal(t, "My title", pageTitle.TextContent())
}
```
</div>

## Selling points

Gost-DOM is an efficient tool for testing, partially because it allows bypassing
the TCP stack and consume the `ServeHTTP` function directly.

<ul class="gd-feature-list">
            <li><span class="gd-feature-list__heading">Speed</span> - By
        eliminating the TCP stack, fetching resources are normal function calls.
        </li>
            <li><span class="gd-feature-list__heading">No hassle managing TCP ports</span> - Automating startup/shutdown of a test-managed server is often a painpoint for web application testing. With Gost-DOM, this layer doesn't exist.
    </li>
            <li><span class="gd-feature-list__heading">Isolation</span> - <span markdown="1">Each `Browser` instance has it's own V8 engine; they cannot interfere with one another.</span>
            </li>
            <li><span class="gd-feature-list__heading">Parallelism</span> - <span markdown="1">Isolation facilitates parallel test execution, as long as _your code_ supports
                parallel test runs.</span></li>
            <li><span class="gd-feature-list__heading">Mocking</span> - <span>With Gost-DOM you treat the SUT as Go-code like any other test; facilitating replacing real dependencies with test doubles.</span></li>
    <li><span class="gd-feature-list__heading">Time travel</span> - <span markdown="1">
            Gost-DOM has a virtual clock controllable by test code, affecting the behaviour of `setTimeout` and `setInterval`. Your tests control the passing of time, e.g. for throttled or debounced behaviour, supporting:
        </span>
        <ul>
            <li><span class="gd-feature-list__heading">Advance time sufficiently</span> - Ensure all effects have executed by advancing time several seconds.</li>
            <li><span class="gd-feature-list__heading">Advance time precisely</span> - Advance time precisely to verify the exact throttling behaviour.</li>
        </ul>
    </li>
</ul>

## Better testing using the Shaman library.

An unrelease side project, Shaman, is in the works. This provides helpers on top
of Gost allowing test cases to be more expressive, and simulate user behaviour.

```Go
func TestMyServer(t *testing.T) {
    window := OpenWindow()
    scope := scope.New(window.Document())
    form := scope.SubScope(scope.ByRole(ariarole.Form))
    form.Find(ByRole(ariarole.TextBox), ByName("Email")).Type("smith@example.com")
    form.Find(ByRole(ariarole.Button), ByName("Reset password")).Click()
    // Assert something happened.
}
```

### Locate elements like users do

When a user interacts with a page, the locate the elements to interact with by
contextual information for the elements. 

A visual user relies on visual affinity of elements to bring context, e.g. the
"text" next to an input field signifies what to type here.

A user relying on a screen reader depends on proper semantic meaning in the
codes to being the same context. They rely on the [accessibility
name](https://developer.mozilla.org/en-US/docs/Glossary/Accessible_name) of the
element to bring the same meaning.

In both cases, the user sees input fields with names. Often, test code relies on
implementation details, such as element attributes like `id` or `data-testid`.
Shaman promotes writing tests that express how a user interact with the
application, yieling benefits:

- Promote accessible design by default.
- Makes the tests resilient to code refactoring.
- Makes the tests resilient to changes to the visual design.

### Sponsors only

At the time of the first pre-release, Shaman will be an exclusive to sponsors at the appropriate
sponsorship tiers; It might eventually be made public.

Hiding Shaman from the public hurts in the heart, but I need to provide a boon
to attract sponsors. I hope to make it publically available, but I need to
provide some boon for to encourage sponsorships.

---

[^1]: You _can_ create an instance of [`html.Window`]() directly, and can
    _potentially_ yield better performance in the current release. But the the
    browser is it brings sensible defaults, and it's less likely to undergo a
    breaking change.

[^2]: Work is in progress to support [goja](https://github.com/dop251/goja) as a
    pure Go alternative to V8, eliminating the need for CGo.

[^3]: While we may work towards full compliance, the priority is writing a tool
    for testing _modern_ web applications. Adding support for old deprecated
    standards is low on the priority list.
