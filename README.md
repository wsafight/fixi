<h1>&#x1F6B2; fixi</h1> 

fixi is a minimalist implementation
of [generalized hypermedia controls](https://dl.acm.org/doi/fullHtml/10.1145/3648188.3675127):

```html

<button fx-action="/content" fx-method="get" fx-target="#output" fx-swap="innerHTML">
    Get Content
</button>
<output id="output"></output>
```

This button will issue a `GET` request to the `/content` relative URL and place the HTML returned by that request inside
the output tag below it.

In contrast to [htmx](https://htmx.org) fixi is designed to be as simple as possible while still being useful
for real world projects. This means it does not have many features found in htmx, including:

* [request queueing](https://htmx.org/attributes/hx-sync/)
* [history support](https://htmx.org/docs/#history)
* [extended selector support](https://htmx.org/docs/#extended-css-selectors)
* [extended event support](https://htmx.org/docs/#special-events)
* [attribute inheritance](https://htmx.org/docs/#inheritance)
* [request indicators](https://htmx.org/docs/#indicators)
* [CSS transitions](https://htmx.org/docs/#css_transitions)

fixi takes advantage of some modern JavaScript features not used by htmx:

* the [`fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
* the use of [`MutationObserver`](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) for monitoring when
  new content is added
* [`async` functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
* The [View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API) (used by htmx, but the
  sole mechanism for transitions in fixi)

## Minimalism

fixi is an experiment
in [software minimalism](https://ia601608.us.archive.org/8/items/pdfy-PeRDID4QHBNfcH7s/LeanSoftware_text.pdf).

The goal is to keep the uncompressed, unminified library size under that of
the [preact](https://bundlephobia.com/package/preact) project, when preact is minified & compressed (currently 4.6Kb).

Current size `4012` bytes as determined by:

```bash
ls -l fixi.js | awk  '{print $5}' 
```

Web developers should be able to use fixi unminified in order to debug the library at development and
deployment time.

Like a fixed-gear bike, fixi provides very few bells and whistle:

* No dependencies (including the test suite, which is a self-contained testing system)
* No JavaScript API (beyond the events)
* No build step
* No `package.json`
* No `fixi.min.js` file

The project consists of three total files:

* This `README.md`, which is the documentation
* `fixi.js`, the code for the library
* `test.html`, the test suite for the library.

`test.html` is a stand-alone file that implements its own visual testing infrastructure, mocking for `fetch()`, etc.

## Installing

fixi is not distributed via [NPM](https://www.npmjs.com/).

Instead, it is intended to be [vendored](https://macwright.com/2021/03/11/vendor-by-default), that is copied, into your 
project:

```bash
curl https://raw.githubusercontent.com/bigskysoftware/fixi/refs/tags/0.0.1/fixi.js >> fixi-0.0.1.js
```

The SHA256 of v0.0.1 is 

`yJ5Uz683QbALhlxdK/butdndgiNHtyC+ORh0kVNaNII=`

generated by the following command line script:

```bash
cat fixi.js | openssl sha256 -binary | openssl base64
```

You can also use the JSDelivr CDN for local development or testing:

```html

<script src="https://cdn.jsdelivr.net/gh/bigskysoftware/fixi@0.0.1/fixi.js"
        integrity="sha256-yJ5Uz683QbALhlxdK/butdndgiNHtyC+ORh0kVNaNII="></script>
```

## API

The fixi api consists of six attributes & six events.

### Attributes

| attribute    | description                                                                                                                                                                               | example               |
|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| `fx-action`  | The URL to which an HTTP request will be issued, required                                                                                                                                 | `fx-action='/demo'`   |
| `fx-method`  | The HTTP Method that will be used for the request (case-insensitive), defaults to `GET`                                                                                                   | `fx-method='DELETE'`  |
| `fx-target`  | A CSS selector specifying where to place the response HTML in the DOM, defaults to the current element                                                                                    | `fx-target='#a-div'`  |
| `fx-swap`    | A string specifying how the content should be swapped into the DOM, one of `innerHTML`, `outerHTML`, `beforestart`, `afterstart`, `beforeend` or `afterend`.  `outerHTML` is the default. | `fx-swap='innerHTML'` |
| `fx-trigger` | The event that will trigger a request.  Defaults to `submit` for `form` elements, `change` for `input`-like elements & `click` for all other elements                                     | `fx-trigger='click'`  |
| `fx-ignore`  | Any element with this attribute on it or on a parent will not be processed for `fx-*` attributes                                                                                          | `                     |

#### Requests

fixi works in a fairly straight-forward manner, and I encourage you to look at [the source](fixi.js). 

The main entry point is that processes the initial DOM and any newly added content looking for elements with the `fx-action`
attribute on them.  

When it finds one it will establish an event listener on that element that will dispatch an AJAX request via `fetch()` to
the given URL, and the response HTML will be inserted into the DOM based on the other fixi attributes on the element.

The default header send with fixi requests is `FX-Request`, which will have the value `true`.

If an element is within a form element or has a `form` attribute, the values of that form will be included with the
request.  Otherwise, if the element has a `name`, it's `name` and `value` will be sent with the request.

`GET` & `DELETE` requests will include values via query parameters, other request types will submit them as a form
encoded body.

#### Example

Here is an example using all the attributes available in fixi:

```html

<button fx-action="/demo" fx-method="GET" fx-target="#output" fx-swap="innerHTML" fx-trigger="click">
    Get Content
</button>
<output id="output" fx-ignore>--</output>
```

In this example, the button will issue a `GET` request to `/demo` and put the resulting HTML into the `innerHTML` of the
output element with the id `output`. Because the `output` element is marked as `fx-ignore` and `fx-action` attributes
in the content will be ignored.

### Events

fixi fires the following events, broken into two categories:

* Initialization related
    * `fx:init` - triggered on elements that have a `fx-action` attribute and are about to be initialized by fixi
    * `fx:process` - triggered on new content added to the DOM that is not filtered via the `fx-ignore` attribute
* `fetch()` related
    * `fx:before` - triggered on an element before a `fetch()` request is made
    * `fx:after` - triggered on an element after a `fetch()` finishes normally but before content is swapped
    * `fx:error` - triggered on an element if an exception occurs during a `fetch()`
    * `fx:finally` - triggered after a request no matter what

#### Initialization Events

##### `fx:init`

The `fx:init` event is triggered when fixi is processing a node with an `fx-action` attribute. If it is cancelled via
`preventDefault()`, the element will not be initialized by fixi

##### `fx:process`

The `fx:process` event is triggered when fixi is sees new content added to the DOM that is not marked as `fx-ignore` or
the child of a node marked as `fx-ignore`. If it is cancelled via `preventDefault()`, the element nor any of its
children will not be initialized by fixi.

#### `fetch()` Events

fixi uses the [`fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) to issue HTTP requests. It
triggers events around this call that allow users to configure the request.

##### `fx:before`

The first event triggered is `fx:before`. This event can be used to configure the arguments passed to `fetch()` via the
fixi config object, which can be found at `evt.detail.cfg`.

This config object has the following properties:

* `method` - The HTTP Method that is going to be used
* `action` - The URL that the request is going to be issued to
* `headers` - An Object of name/value pairs to be sent as HTTP Request Headers
* `target` - The target element that will be swapped when the response is processed
* `swap` - The mechanism by which the element will be swapped
* `body` - The body of the request, if present, a FormData object that holds the data of the form associated with the
* `drop` - Whether this request will be dropped, defaults to `true` if a request is already in flight
* `transition` - Whether to use the View Transition API for swaps, defaults to `true`
* `preventTriggerDefault` - A boolean (defaults to true) that, if true, will call `preventDefault()` on the triggering
  event
* `signal` - The AbortSignal of the related AbortController for the request
* `abort()` - A function that can be invoked to abort the pending fetch request

Mutating the `method`, etc. properties of the `cfg` object will change the behavior of the request dynamically. Note
that the `cfg` object is passed to `fetch()` as the second argument of type `RequestInit`, so any properties you want
to set on the `RequestInit` may be set on the `cfg` object (e.g. `credentials`).

You can also set a property on the config, `confirm`, that should be a no-argument function. This function should return
a Promise and can be used to asynchronously confirm that the request should be issued:

```js
function showAsynConfirmDialog() {
    //... a Promise-based confirmation dialog...
}

document.addEventListener("fx:before", (evt) => {
    evt.detail.cfg.confirm = showAsynConfirmDialog;
})
```

Another property available on the `detail` of this event is `requests`, which will be an array of any existing
outstanding requests for the element.

fixi does not implement request queuing like htmx does, but you can implement a simple
"replace existing requests in flight" rule with the following JavaScript:

```js
document.addEventListener("fx:before", (evt) => {
    evt.detail.cfg.drop = false;                       // allow this request to be issued
    evt.detail.requests.forEach((cfg) => cfg.abort()); // abort all existing requests
})
```

If you call `preventDefault()` on this event, no request will be issued.

##### `fx:after`

The  `fx:after` event is triggered after a `fetch()` successfully completes. The config will again be available in the
`evt.detail.cfg` property, and will have two additional properties:

* `response` - The response object from the `fetch()` call
* `text` - The text of the response

At this point you may still mutate the `swap`, etc. attributes to affect swapping, and you may mutate the `text` if you
want to modify it is some way before it is swapped.

Calling `preventDefault()` on this event will prevent swapping from occurring.

##### `fx:error`

The  `fx:error` event is triggered when a network error occurs. In this case the `cfg.txt` will be set to a blank
string, and the `evt.detail.cfg` object is available for modification.

Calling `preventDefault()` on this event will prevent swapping from occurring. Note that `AbortError`s will also prevent
swapping.

##### `fx:finally`

The  `fx:finally` event is triggered regardless if an error occurs or not and can be used to clean up after a request.
Again the `evt.detail.cfg` object is available for modification.

## Examples

Because fixi is minimalistic, the user is responsible for implementing many behaviors they want via events. We have
already seen how to abort an existing request that is already in flight.

Here are some additional examples of useful behaviors implemented using events:

### Disabling an Element During A Request

Here is an example that will use attributes to disable an element when a request is in flight:

```js
function maybeDisable(elt, disabled) {
    if (elt.hasAttribute('ext-disable')) {
        var disableTarget = elt.getAttribute('ext-disable');
        if (disableTarget === '') {
            elt.disabled = disabled;
        } else {
            document.querySelector(disableTarget).disabled = disabled;
        }
    }
}

document.addEventListener("fx:before", (evt) => {
    maybeDisable(evt.target, true);
})
document.addEventListener("fx:finally", (evt) => {
    maybeDisable(evt.target, false);
})
```

### Showing an Indicator During A Request

Here is an example that will use attributes and the `display` property to show an indicator element when a request is
in flight:

```js
function handleIndicator(elt, show) {
    if (elt.hasAttribute('ext-indicator')) {
        var indicator = elt.getAttribute('ext-indicator');
        document.querySelector(indicator).style.display = show ? 'initial' : 'none';
    }
}

document.addEventListener("fx:before", (evt) => {
    handleIndicator(evt.target, true);
})
document.addEventListener("fx:finally", (evt) => {
    handleIndicator(evt.target, false);
})
```

This example can be modified to use classes or other mechanisms for showing indicators as well.

### Debouncing A Request

Here is an implementation of the Active Search example from htmx done in fixi:

```html
<script>
  /**
   * Debounces the debouncedEvent for the given delay, emitting a triggeredEvent if
   * no new occurrences of the debounced event occur in the given delay interval, and
   * resetting the delay if so
   * 
   * @param elt - the elt to listen for the `debouncedEvent` on
   * @param debouncedEvent - the event type to debounce
   * @param triggeredEvent - the event to trigger when the interval `delay` has eclipsed and no new events to debounce have occurred
   * @param delay - the interval to wait for a new debouncedEvent before emitting a triggeredEvent
   */
  function debounce(elt, debouncedEvent, triggeredEvent, delay) {
    let currentTimeout = null;
    elt.addEventListener(debouncedEvent, () => {
      if (currentTimeout != null) {
        clearTimeout(currentTimeout);
      }
      currentTimeout = setTimeout(() => {
        element.dispatchEvent(new CustomEvent(triggeredEvent))
        currentTimeout = null;
      }, delay);
    })
  }

  document.addEventListener("DOMContentLoaded", () => {
    let element = document.querySelector("#search");
    element.addEventListener('search', () => element.dispatchEvent(new CustomEvent('doSearch')));
    debounce(element, 'input', 'doSearch', 200);
  })
</script>
<input id="search" type="search" fx-action="/search" fx-trigger="doSearch" fx-target="#results" fx-swap="innerHTML"/>
<table>
    ...
    <tbody id="results">
    ...
    </tbody>
</table>
```

## LICENCE

```
Zero-Clause BSD
=============

Permission to use, copy, modify, and/or distribute this software for
any purpose with or without fee is hereby granted.

THE SOFTWARE IS PROVIDED “AS IS” AND THE AUTHOR DISCLAIMS ALL
WARRANTIES WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES
OF MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE
FOR ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY
DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN
AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT
OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
```
