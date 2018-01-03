# Handling Events

{{quote {author: "Marcus Aurelius", title: Meditations, chapter: true}

You have power over your mind—not
outside events. Realize this, and you will find strength.

quote}}

{{index stoicism, "Marcus Aurelius", input, timeline, "control flow"}}

Some programs work with direct user input, such as mouse and keyboard.
That kind of input isn't available as a whole at the start of your
program—it comes in piece by piece, and the program is expected to
react to it as it occurs.

## Event handlers

{{index polling, button, "real-time"}}

Imagine an interface where the only way to find out whether a key on
the keyboard is being pressed is to read the current state of that
key. To be able to react to keypresses, you would have to constantly
read the key's state so that you'd catch it before it's released
again. It would be dangerous to perform other time-intensive
computations since you might miss a keypress.

Some primitive machines do handle input like that. A step up would be
for the hardware or operating system to notice the keypress and put it
in a queue. A program can then periodically check the queue for new
events and react to what it finds there.

{{index responsiveness, "user experience"}}

Of course, it has to remember to look at the queue, and to do it
often, because any time between the key being pressed and the program
noticing the event will cause the software to feel unresponsive. This
approach is called _((polling))_. Most programmers prefer to avoid it
when possible.

{{index "callback function", "event handling"}}

A better mechanism is for the underlying system to give our code a
chance to react to events as they occur. Browsers do this by allowing
us to register functions as _handlers_ for specific events.

```{lang: "text/html"}
<p>Click this document to activate the handler.</p>
<script>
  addEventListener("click", () => {
    console.log("You knocked?");
  });
</script>
```

{{index "click event", "addEventListener method"}}

The `addEventListener` method registers its second argument to be
called whenever the event described by its first argument occurs.

## Events and DOM nodes

{{index "addEventListener method", "event handling"}}

Each ((browser)) event handler is registered in a context. When you
call `addEventListener` as we did before, you are calling it as a
method on the whole ((window)) because in the browser the ((global
scope)) is equivalent to the `window` object. Every ((DOM)) element,
as well as some other types of objects, has its own `addEventListener`
method, which allows you to listen specifically on that element.

```{lang: "text/html"}
<button>Click me</button>
<p>No handler here.</p>
<script>
  let button = document.querySelector("button");
  button.addEventListener("click", () => {
    console.log("Button clicked.");
  });
</script>
```

{{index "click event", "button (HTML tag)"}}

That example attaches a handler to the button node. Clicks on the
button cause that handler to run, but clicks on the rest of the
document do not.

{{index "onclick attribute", encapsulation}}

Giving a node an `onclick` attribute has a similar effect. But a node
has only one `onclick` attribute, so you can register only one handler
per node that way. The `addEventListener` method allows you to add any
number of handlers, so you won't accidentally replace a handler that
has already been registered.

{{index "removeEventListener method"}}

The `removeEventListener` method, called with arguments similar to as
`addEventListener`, removes a handler.

```{lang: "text/html"}
<button>Act-once button</button>
<script>
  let button = document.querySelector("button");
  function once() {
    console.log("Done.");
    button.removeEventListener("click", once);
  }
  button.addEventListener("click", once);
</script>
```

{{index [function, "as value"]}}

The function given to `removeEventListener` has to be the exact same
function value that was given to `addEventListener`. So to unregister
a handler, you'll want to give the function a name (`once`, in the
example) to be able to pass the same value to both methods.

## Event objects

{{index "button property", "event handling"}}

Though we have ignored it so far, event handler functions are passed
an argument: the _((event object))_. This object holds additional
information about the event. For example, if we want to know _which_
((mouse button)) was pressed, we can look at the event object's
`button` property.

```{lang: "text/html"}
<button>Click me any way you want</button>
<script>
  let button = document.querySelector("button");
  button.addEventListener("mousedown", event => {
    if (event.button == 0) {
      console.log("Left button");
    } else if (event.which == 1) {
      console.log("Middle button");
    } else if (event.which == 2) {
      console.log("Right button");
    }
  });
</script>
```

{{index "event type", "type property"}}

The information stored in an event object differs per type of event.
We'll discuss different types later in the chapter. The object's
`type` property always holds a string identifying the event (such as
`"click"` or `"mousedown"`).

## Propagation

{{index "event propagation", "parent node"}}

{{indexsee bubbling, "event propagation"}}

{{indexsee propagation, "event propagation"}}

Event handlers registered on nodes with children will also receive
some events that happen in the children. If a button inside a
paragraph is clicked, event handlers on the paragraph will also
receive the click event.

{{index "event handling"}}

But if both the paragraph and the button have a handler, the more
specific handler—the one on the button—gets to go first. The event is
said to _propagate_ outward, from the node where it happened to that
node's parent node and on to the root of the document. Finally, after
all handlers registered on a specific node have had their turn,
handlers registered on the whole ((window)) get a chance to respond to
the event.

{{index "stopPropagation method", "click event"}}

At any point, an event handler can call the `stopPropagation` method
on the event object to prevent handlers "further up" from receiving
the event. This can be useful when, for example, you have a button
inside another clickable element and you don't want clicks on the
button to activate the outer element's click behavior.

{{index "mousedown event"}}

The following example registers `"mousedown"` handlers on both a
button and the paragraph around it. When clicked with the right mouse
button, the handler for the button calls `stopPropagation`, which will
prevent the handler on the paragraph from running. When the button is
clicked with another ((mouse button)), both handlers will run.

```{lang: "text/html"}
<p>A paragraph with a <button>button</button>.</p>
<script>
  let para = document.querySelector("p");
  let button = document.querySelector("button");
  para.addEventListener("mousedown", () => {
    console.log("Handler for paragraph.");
  });
  button.addEventListener("mousedown", event => {
    console.log("Handler for button.");
    if (event.button == 2) event.stopPropagation();
  });
</script>
```

{{index "event propagation", "target property"}}

Most event objects have a `target` property that refers to the node
where they originated. You can use this property to ensure that you're
not accidentally handling something that propagated up from a node you
do not want to handle.

It is also possible to use the `target` property to cast a wide net
for a specific type of event. For example, if you have a node
containing a long list of buttons, it may be more convenient to
register a single click handler on the outer node and have it use the
`target` property to figure out whether a button was clicked, rather
than register individual handlers on all of the buttons.

```{lang: "text/html"}
<button>A</button>
<button>B</button>
<button>C</button>
<script>
  document.body.addEventListener("click", event => {
    if (event.target.nodeName == "BUTTON") {
      console.log("Clicked", event.target.textContent);
    }
  });
</script>
```

## Default actions

{{index scrolling, "default behavior", "event handling"}}

Many events have a default action associated with them. If you click a
((link)), you will be taken to the link's target. If you press the
down arrow, the browser will scroll the page down. If you right-click,
you'll get a context menu. And so on.

{{index "preventDefault method"}}

For most types of events, the JavaScript event handlers are called
_before_ the default behavior takes place. If the handler doesn't want
this normal behavior to happen, typically because it has already taken
care of handling the event, it can call the `preventDefault` method on
the event object.

{{index expectation}}

This can be used to implement your own ((keyboard)) shortcuts or
((context menu)). It can also be used to obnoxiously interfere with
the behavior that users expect. For example, here is a link that
cannot be followed:

```{lang: "text/html"}
<a href="https://developer.mozilla.org/">MDN</a>
<script>
  let link = document.querySelector("a");
  link.addEventListener("click", event => {
    console.log("Nope.");
    event.preventDefault();
  });
</script>
```

Try not to do such things unless you have a really good reason to. For
people using your page, it can be unpleasant when the behavior they
expect is broken.

Depending on the browser, some events can't be intercepted at all. On
Chrome, for example, the ((keyboard)) shortcut to close the current
tab (Ctrl-W or Command-W) cannot be handled by JavaScript.

## Key events

{{index keyboard, "keydown event", "keyup event", "event handling"}}

When a key on the keyboard is pressed, your browser fires a
`"keydown"` event. When it is released again, you get a `"keyup"`
event.

```{lang: "text/html", focus: true}
<p>This page turns violet when you hold the V key.</p>
<script>
  addEventListener("keydown", event => {
    if (event.key == "v") {
      document.body.style.background = "violet";
    }
  });
  addEventListener("keyup", event => {
    if (event.key == "v") {
      document.body.style.background = "";
    }
  });
</script>
```

{{index "repeating key"}}

Despite its name, `"keydown"` fires not only when the key is
physically pushed down. When a key is pressed and held, the event
fires again every time the key _repeats_. Sometimes—for example if you
want to increase the acceleration of a ((game)) character when an
arrow key is pressed and decrease it again when the key is
released—you have to be careful not to repeat the effect every time
the key repeats or you'd end up with an unintentionally supersonic
character.

{{index "key property"}}

The example looked at the `key` property of the event object. This is
how you can identify which key is being pressed or released.
Generally, the `key` string corresponds to the thing that pressing
that key would type, but for special keys like Enter, it holds a
specific string that names the key (`"Enter"`, in this case). If you
hold shift while pressing a key, that might also influence the name of
the key—`"v"` becomes `"V"`, `"1"` may become `"!"`, if that is what
pressing Shift-1 produces on your keyboard.

{{index "modifier key", "shift key", "control key", "alt key", "meta key", "command key", "ctrlKey property", "shiftKey property", "altKey property", "metaKey property"}}

Modifier keys such as Shift, Ctrl, Alt, and Meta (Command on Mac)
generate key events just like normal keys. But when looking for key
combinations, you can also find out whether these keys are held down
by looking at the `shiftKey`, `ctrlKey`, `altKey`, and `metaKey`
properties of keyboard and mouse events.

```{lang: "text/html", focus: true}
<p>Press Ctrl-Space to continue.</p>
<script>
  addEventListener("keydown", event => {
    if (event.key == " " && event.ctrlKey) {
      console.log("Continuing!");
    }
  });
</script>
```

{{index "button (HTML tag)", "tabindex attribute"}}

The ((DOM)) node where a key event originates depends on the element
that has ((focus)) when the key is pressed. Normal nodes cannot have
focus (unless you give them a `tabindex` attribute), but things such
as ((link))s, buttons, and form fields can. We'll come back to form
((field))s in [Chapter ?](forms). When nothing in particular has
focus, `document.body` acts as the target node of key events.

When the user is typing text, using key events to figure out what is
being typed is problematic. Some platforms, most notably the ((virtual
keyboard)) on ((Android)) ((phone))s, don't fire key events. But even
when you have an old-fashioned keyboard, some types of text input
don't match key presses in a straightforward way, such as ((IME))
("Input Method Editor") software used by people whose scripts don't
fit on a keyboard, where multiple key strokes are combined to create
characters.

To notice when something was typed, elements that you can type into,
such as the `<input>` and `<textarea>` tags, fire `"input"` events
whenever the user changed their content. To get the actual content
that was typed, it is best to directly read it from the page. [Chapter
?](forms) will show how.

## Mouse clicks

{{index "mousedown event", "mouseup event", "mouse cursor"}}

Pressing a ((mouse button)) also causes a number of events to fire.
The `"mousedown"` and `"mouseup"` events are similar to `"keydown"`
and `"keyup"` and fire when the button is pressed and released. These
happen on the DOM nodes that are immediately below the mouse pointer
when the event occurs.

{{index "click event"}}

After the `"mouseup"` event, a `"click"` event fires on the most
specific node that contained both the press and the release of the
button. For example, if I press down the mouse button on one paragraph
and then move the pointer to another paragraph and release the button,
the `"click"` event will happen on the element that contains both
those paragraphs.

{{index "dblclick event", "double click"}}

If two clicks happen close together, a `"dblclick"` (double-click)
event also fires, after the second click event.

{{index pixel, "clientX property", "clientY property", "pageX property", "pageY property", "event object"}}

To get precise information about the place where a mouse event
happened, you can look at its `clientX` and `clientY` properties,
which contain the event's ((coordinates)) (in pixels) relative to the
top-left corner of the window, or `pageX` and `pageY`, which are
relative to the top-left corner of the whole document.

{{index "border-radius (CSS)", "absolute positioning", "drawing program example"}}

{{id mouse_drawing}}

The following implements a primitive drawing program. Every time you
click the document, it adds a dot under your mouse pointer. See
[Chapter ?](paint) for a less primitive drawing program.

```{lang: "text/html"}
<style>
  body {
    height: 200px;
    background: beige;
  }
  .dot {
    height: 8px; width: 8px;
    border-radius: 4px; /* rounds corners */
    background: blue;
    position: absolute;
  }
</style>
<script>
  addEventListener("click", event => {
    let dot = document.createElement("div");
    dot.className = "dot";
    dot.style.left = (event.pageX - 4) + "px";
    dot.style.top = (event.pageY - 4) + "px";
    document.body.appendChild(dot);
  });
</script>
```

## Mouse motion

{{index "mousemove event"}}

Every time the mouse pointer moves, a `"mousemove"` event is fired.
This event can be used to track the position of the mouse. A common
situation in which this is useful is when implementing some form of
mouse-((dragging)) functionality.

{{index "draggable bar example"}}

As an example, the following program displays a bar and sets up event
handlers so that dragging to the left or right on this bar makes it
narrower or wider:

```{lang: "text/html"}
<p>Drag the bar to change its width:</p>
<div style="background: orange; width: 60px; height: 20px">
</div>
<script>
  let lastX; // Tracks the last observed mouse X position
  let rect = document.querySelector("div");
  rect.addEventListener("mousedown", event => {
    if (event.button == 0) {
      lastX = event.clientX;
      addEventListener("mousemove", moved);
      event.preventDefault(); // Prevent selection
    }
  });

  function moved(event) {
    if (event.buttons == 0) {
      removeEventListener("mousemove", moved);
    } else {
      let dist = event.clientX - lastX;
      let newWidth = Math.max(10, rect.offsetWidth + dist);
      rect.style.width = newWidth + "px";
      lastX = event.clientX;
    }
  }
</script>
```

{{if book

The resulting page looks like this:

{{figure {url: "img/drag-bar.png", alt: "A draggable bar",width: "5.3cm"}}}

if}}

{{index "mouseup event", "mousemove event"}}

Note that the `"mousemove"` handler is registered on the whole
((window)). Even if the mouse goes outside of the bar during resizing,
we still want to update its size as long as the button is held.

{{index "buttons property", "button property", "bitfield"}}

We must stop resizing the bar when the mouse button is released. For
that, we must use the `buttons` property (note the plural), which
tells us about the buttons that are currently held down. When this is
zero, no buttons are down. When buttons are held, its value is the sum
of the codes for those buttons—the left button has code 1, the right
button 2, and the middle one 4. That way, you can check if a given
button is pressed by taking the remainder of the value of `buttons`
and its code. Note that the order of these codes is different from the
one used by `button`, where the middle button came before the right
one. As mentioned, consistency isn't really a strong point of the
browser's programming interface.

## Scroll events

{{index scrolling, "scroll event", "event handling"}}

Whenever an element is scrolled, a `"scroll"` event is fired on it.
This has various uses, such as knowing what the user is currently
looking at (for disabling off-screen ((animation))s or sending ((spy))
reports to your evil headquarters) or showing some indication of
progress (by highlighting part of a table of contents or showing a
page number).

The following example draws a ((progress bar)) in the top-right corner
of the document and updates it to fill up as you scroll down:

```{lang: "text/html"}
<style>
  #progress {
    border-bottom: 2px solid blue;
    width: 0;
    position: fixed;
    top: 0; left: 0;
  }
</style>
<div id="progress"></div>
<script>
  // Create some content
  document.body.appendChild(document.createTextNode(
    "supercalifragilisticexpialidocious ".repeat(1000)));

  let bar = document.querySelector("#progress");
  addEventListener("scroll", () => {
    let max = document.body.scrollHeight - innerHeight;
    bar.style.width = `${(pageYOffset / max) * 100}%`;
  });
</script>
```

{{index "unit (CSS)", scrolling, "position (CSS)", "fixed positioning", "absolute positioning", percent, "repeat method"}}

Giving an element a `position` of `fixed` acts much like an `absolute`
position but also prevents it from scrolling along with the rest of
the document. The effect is to make our progress bar stay in its
corner. It is resized to indicate the current progress. We use `%`,
rather than `px`, as a unit when setting the width so that the element
is sized relative to the page width.

{{index "innerHeight property", "innerWidth property", "pageYOffset property"}}

The global `innerHeight` binding gives us the height of the window,
which we have to subtract from the total scrollable height—you can't
keep scrolling when you hit the bottom of the document. (There's also
an `innerWidth` to go along with `innerHeight`.) By dividing
`pageYOffset`, the current scroll position, by the maximum scroll
position and multiplying by 100, we get the percentage for the
progress bar.

{{index "preventDefault method"}}

Calling `preventDefault` on a scroll event does not prevent the
scrolling from happening. In fact, the event handler is called only
_after_ the scrolling takes place.

## Focus events

{{index "event handling", "focus event", "blur event"}}

When an element gains ((focus)), the browser fires a `"focus"` event
on it. When it loses focus, the element gets a `"blur"` event.

{{index "event propagation"}}

Unlike the events discussed earlier, these two events do not
propagate. A handler on a parent element is not notified when a child
element gains or loses focus.

{{index "input (HTML tag)", "help text example"}}

The following example displays help text for the ((text field)) that
currently has focus:

```{lang: "text/html"}
<p>Name: <input type="text" data-help="Your full name"></p>
<p>Age: <input type="text" data-help="Age in years"></p>
<p id="help"></p>

<script>
  let help = document.querySelector("#help");
  let fields = document.querySelectorAll("input");
  for (let field of Array.from(fields)) {
    field.addEventListener("focus", event => {
      let text = event.target.getAttribute("data-help");
      help.textContent = text;
    });
    field.addEventListener("blur", event => {
      help.textContent = "";
    });
  }
</script>
```

{{if book

In this screenshot, the help text for the age field is shown.

{{figure {url: "img/help-field.png", alt: "Providing help when a field is focused",width: "4.4cm"}}}

if}}

{{index "focus event", "blur event"}}

The ((window)) object will receive `"focus"` and `"blur"` events when
the user moves from or to the browser tab or window in which the
document is shown.

## Load event

{{index "script (HTML tag)", "load event"}}

When a page finishes loading, the `"load"` event fires on the window
and the document body objects. This is often used to schedule
((initialization)) actions that require the whole ((document)) to have
been built. Remember that the content of `<script>` tags is run
immediately when the tag is encountered. This is often too soon, such
as when the script needs to do something with parts of the document
that appear after the `<script>` tag.

{{index "event propagation", "img (HTML tag)"}}

Elements such as ((image))s and script tags that load an external file
also have a `"load"` event that indicates the files they reference
were loaded. Like the focus-related events, loading events do not
propagate.

{{index "beforeunload event", "page reload", "preventDefault method"}}

When a page is closed or navigated away from (for example by following
a link), a `"beforeunload"` event fires. The main use of this event is
to prevent the user from accidentally losing work by closing a
document. Preventing the page from unloading is not, as you might
expect, done with the `preventDefault` method. Instead, it is done by
returning a string from the handler. The string will be used in a
dialog that asks the user if they want to stay on the page or leave
it. This mechanism ensures that a user is able to leave the page, even
if it is running a ((malicious script)) that would prefer to keep them
there forever and force them to look at dodgy weight loss ads.

{{id timeline}}

## Events and the event loop

{{index "requestAnimationFrame function", "event handling", timeline, "script (HTML tag)"}}

In the context of the event loop, as discussed in [Chapter ?](async),
event handlers are similar to notifications of other asynchronous
events. They are scheduled when the event occurs, but must wait for
other scripts that are running to finish before they get a chance to
run.

The fact that events can only be processed when nothing else is
running means that, if the event loop is tied up with other work, any
interaction with the page (which happens through events) will be
delayed until there's time to process it. So if you schedule too much
work, either with long-running event handlers or with short-running
ones that fire very often, the page will become slow and cumbersome to
use.

For cases where you _really_ do want to do some time-consuming thing
in the background without freezing the page, browsers provide
something called _((web worker))s_. A worker is a JavaScript process
that runs alongside the main script, on its own timeline.

Imagine that squaring a number is a heavy, long-running computation
that we want to perform in a separate ((thread)). We could write a
file called `code/squareworker.js` that responds to messages by
computing a square and sending a message back:

```
addEventListener("message", event => {
  postMessage(event.data * event.data);
});
```

To avoid the problems of having multiple ((thread))s touching the same
data, workers do not share their ((global scope)) or any other data
with the main script's environment. Instead, you have to communicate
with them by sending messages back and forth.

This code spawns a worker that uses that script, sends it a few
messages, and outputs the responses.

```{test: no}
let squareWorker = new Worker("code/squareworker.js");
squareWorker.addEventListener("message", event => {
  console.log("The worker responded:", event.data);
});
squareWorker.postMessage(10);
squareWorker.postMessage(24);
```

{{index "postMessage method", "message event"}}

The `postMessage` function sends a message, which will cause a
`"message"` event to fire in the receiver. The script that created the
worker sends and receives messages through the `Worker` object,
whereas the worker talks to the script that created it by sending and
listening directly on its ((global scope)). Only values that can be
represented as JSON can be sent as messages—and the other side will
receive a _copy_ of them, rather than the value itself.

## Timers

{{index timeout, "setTimeout function"}}

We saw the `setTimeout` function in [Chapter ?](async). It schedules
another function to be called later, after a given amount of
milliseconds. This page turns from blue to yellow after two seconds:

```{lang: "text/html"}
<script>
  document.body.style.background = "blue";
  setTimeout(() => {
    document.body.style.background = "yellow";
  }, 2000);
</script>
```

{{index "clearTimeout function"}}

Sometimes you need to cancel a function you have scheduled. This is
done by storing the value returned by `setTimeout` and calling
`clearTimeout` on it.

```
let bombTimer = setTimeout(() => {
  console.log("BOOM!");
}, 500);

if (Math.random() < 0.5) { // 50% chance
  console.log("Defused.");
  clearTimeout(bombTimer);
}
```

{{index "cancelAnimationFrame function", "requestAnimationFrame function"}}

The `cancelAnimationFrame` function works in the same way as
_clearTimeout_—calling it on a value returned by
`requestAnimationFrame` will cancel that frame (assuming it hasn't
already been called).

{{index "setInterval function", "clearInterval function", repetition}}

A similar set of functions, `setInterval` and `clearInterval` are used
to set timers that should _repeat_ every _X_ milliseconds.

```
let ticks = 0;
let clock = setInterval(() => {
  console.log("tick", ticks++);
  if (ticks == 10) {
    clearInterval(clock);
    console.log("stop.");
  }
}, 200);
```

## Debouncing

{{index optimization, "mousemove event", "scroll event", blocking}}

Some types of events have the potential to fire rapidly, many times in
a row (the `"mousemove"` and `"scroll"` events, for example). When
handling such events, you must be careful not to do anything too
time-consuming or your handler will take up so much time that
interaction with the document starts to feel slow and choppy.

{{index "setTimeout function"}}

If you do need to do something nontrivial in such a handler, you can
use `setTimeout` to make sure you are not doing it too often. This is
usually called _((debouncing))_ the event. There are several slightly
different approaches to this.

{{index "textarea (HTML tag)", "clearTimeout function", "keydown event"}}

In the first example, we want to react when the user has typed
something, but we don't want to do it immediately for every input
event. When they are ((typing)) quickly, we just want to wait until a
pause occurs. Instead of immediately performing an action in the event
handler, we set a timeout. We also clear the previous timeout (if any)
so that when events occur close together (closer than our timeout
delay), the timeout from the previous event will be canceled.

```{lang: "text/html"}
<textarea>Type something here...</textarea>
<script>
  let textarea = document.querySelector("textarea");
  let timeout;
  textarea.addEventListener("input", () => {
    clearTimeout(timeout);
    timeout = setTimeout(() => console.log("Typed!"), 500);
  });
</script>
```

{{index "sloppy programming"}}

Giving an undefined value to `clearTimeout` or calling it on a timeout
that has already fired has no effect. Thus, we don't have to be
careful about when to call it, and we simply do so for every event.

{{index "mousemove event"}}

We can use a slightly different pattern if we want to space responses
so that they're separated by at least a certain length of ((time)) but
want to fire them _during_ a series of events, not just afterward. For
example, we might want to respond to `"mousemove"` events by showing
the current coordinates of the mouse, but only every 250 milliseconds.

```{lang: "text/html"}
<script>
  let scheduled = null;
  addEventListener("mousemove", event => {
    if (!scheduled) {
      setTimeout(() => {
        document.body.textContent =
          `Mouse at ${scheduled.pageX}, ${scheduled.pageY}`;
        scheduled = null;
      }, 250);
    }
    scheduled = event;
  });
</script>
```

## Summary

Event handlers make it possible to detect and react to events
happening in the world of our web page. The `addEventListener` method
is used to register such a handler.

Each event has a type (`"keydown"`, `"focus"`, and so on) that
identifies it. Most events are called on a specific DOM element and
then _propagate_ to that element's ancestors, allowing handlers
associated with those elements to handle them.

When an event handler is called, it is passed an event object with
additional information about the event. This object also has methods
that allow us to stop further propagation (`stopPropagation`) and
prevent the browser's default handling of the event
(`preventDefault`).

Pressing a key fires `"keydown"` and `"keyup"` events. Pressing a
mouse button fires `"mousedown"`, `"mouseup"`, and `"click"` events.
Moving the mouse fires `"mousemove"` events.

Scrolling can be detected with the `"scroll"` event, and focus changes
can be detected with the `"focus"` and `"blur"` events. When the
document finishes loading, a `"load"` event fires on the window.

Only one piece of JavaScript program can run at a time. Thus, event
handlers and other scheduled scripts have to wait until other scripts
finish before they get their turn.

## Exercises

### Balloon

{{index "balloon (exercise)", "arrow key"}}

Write a page that display a ((balloon)) (using the "balloon"
((emoji)), 🎈). When you press the up arrow, it should inflate (grow)
ten percent, and when you press the down arrow, it should deflate
(shrink) 10%.

{{index "font-size (CSS)"}}

You can control the size of text (emoji are text) by setting the
`font-size` CSS property (`style.fontSize`) on its parent element.
Remember to include a unit in the value, for example pixels (`10px`).

The key names of the arrow keys are `"ArrowUp"` and `"ArrowDown"`.
Make sure the keys only change the balloon, without scrolling the
page.

When that works, add a feature where, if you blow up the balloon past
a certain size, it explodes. In this case, exploding means that it is
replaced with an "💥" emoji, and the event handler is removed (so that
you can't inflate or deflate the explosion).

{{if interactive

```{test: no, lang: "text/html", focus: yes}
<p>🎈</p>

<script>
  // Your code here
</script>
```

if}}

{{hint

{{index "keydown event", "key property", "balloon (exercise)"}}

You'll want to register a handler for the `"keydown"` event, and look
at `event.key` to figure out whether the up or down arrow key was
pressed.

The current size can be kept in a binding, so that you can base the
new size on it. It'll be helpful to define a function that updates the
size—both the binding and the style of the balloon in the DOM, so that
you can call it from your event handler, and possibly also once when
starting, to set the initial size.

{{index "replaceChild method", "textContent propery"}}

You can change the balloon to an explosion by replacing the text node
with another one (using `replaceChild`), or by setting the
`textContent` property of its parent node to a new string.

hint}}

### Mouse trail

{{index animation, "mouse trail (exercise)"}}

In JavaScript's early days, which was the high time of ((gaudy home
pages)) with lots of animated images, people came up with some truly
inspiring ways to use the language.

One of these was the _mouse trail_—a series of images that would
follow the mouse pointer as you moved it across the page.

{{index "absolute positioning", "background (CSS)"}}

In this exercise, I want you to implement a mouse trail. Use
absolutely positioned `<div>` elements with a fixed size and
background color (refer to the [code](event#mouse_drawing) in the
"Mouse Clicks" section for an example). Create a bunch of such
elements and, when the mouse moves, display them in the wake of the
mouse pointer.

{{index "mousemove event"}}

There are various possible approaches here. You can make your solution
as simple or as complex as you want. A simple solution to start with
is to keep a fixed number of trail elements and cycle through them,
moving the next one to the mouse's current position every time a
`"mousemove"` event occurs.

{{if interactive

```{lang: "text/html", test: no}
<style>
  .trail { /* className for the trail elements */
    position: absolute;
    height: 6px; width: 6px;
    border-radius: 3px;
    background: teal;
  }
  body {
    height: 300px;
  }
</style>

<script>
  // Your code here.
</script>
```

if}}

{{hint

{{index "mouse trail (exercise)"}}

Creating the elements is best done with a loop. Append them to the
document to make them show up. To be able to access them later in
order to change their position, store the trail elements in an array.

{{index "mousemove event", [array, indexing], "remainder operator", "% operator"}}

Cycling through them can be done by keeping a ((counter variable)) and
adding 1 to it every time the `"mousemove"` event fires. The remainder
operator (`% 10`) can then be used to get a valid array index to pick
the element you want to position during a given event.

{{index simulation, "requestAnimationFrame function"}}

Another interesting effect can be achieved by modeling a simple
((physics)) system. Use the `"mousemove"` event only to update a pair
of bindings that track the mouse position. Then use
`requestAnimationFrame` to simulate the trailing elements being
attracted to the position of the mouse pointer. At every animation
step, update their position based on their position relative to the
pointer (and, optionally, a speed that is stored for each element).
Figuring out a good way to do this is up to you.

hint}}

### Tabs

{{index "tabbed interface (exercise)"}}

Tabbed panels are widely used in user interfaces. They allows you to
select an interface panel by choosing from a number of tabs "sticking
out" above an element.

{{index "button (HTML tag)", "display (CSS)", "hidden element", "data attribute"}}

In this exercise you'll implement a simple tabbed interface. Write a
function, `asTabs`, that takes a DOM node and creates a tabbed
interface showing the child elements of that node. It should insert a
list of `<button>` elements at the top of the node, one for each child
element, containing text retrieved from the `data-tabname` attribute
of the child. All but one of the original children should be hidden
(given a `display` style of `none`). The currently visible node can be
selected by clicking the buttons.

When it works, extend it to also style the currently active button
differently.

{{if interactive

```{lang: "text/html", test: no}
<tab-panel>
  <div data-tabname="one">Tab one</div>
  <div data-tabname="two">Tab two</div>
  <div data-tabname="three">Tab three</div>
</tab-panel>
<script>
  function asTabs(node) {
    // Your code here.
  }
  asTabs(document.querySelector("tab-panel"));
</script>
```

if}}

{{hint

{{index "text node", "childNodes property", "live data structure", "tabbed interface (exercise)"}}

One pitfall you'll probably run into is that you can't directly use
the node's `childNodes` property as a collection of tab nodes. For one
thing, when you add the buttons, they will also become child nodes and
end up in this object because it is a live data structure. For
another, the text nodes created for the ((whitespace)) between the
nodes are also in there and should not get their own tabs.

{{index "TEXT_NODE code", "nodeType property"}}

To work around this, start by building up a real array of all the
children in the wrapper that have a `nodeType` of 1.

{{index "event handling", closure}}

When registering event handlers on
the buttons, the handler functions will need to know which tab element
is associated with the button. If they are created in a normal loop,
you can access the loop index binding from inside the function, but
it won't give you the correct number because that binding will have
been further changed by the loop.

{{index "forEach method", "local binding", loop}}

A simple workaround is to use the `forEach` method and create the
handler functions from inside the function passed to `forEach`. The
loop index, which is passed as a second argument to that function,
will be a normal local binding there and won't be overwritten by
further iterations.

hint}}
