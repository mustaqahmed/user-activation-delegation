# Activation Transfer through postMessages
An API to allow developers transfer user activation state to any target `window`
in the frame tree.


## Introduction

### What's a user activation?

The term _user activation_ means the state of a browsing session with respect to
user actions: an "active" state typically implies either the user is currently
interacting with the page through some input mechanism (typing, clicking with
mouse etc.), or the user has completed some interaction since the page got
loaded.  Browsers control access to "abusable" of APIs through user activation,
e.g., popups are usually blocked unless the user is actively interacting with
the page.


### Why activation transfer?

Chrome shipped [User Activation
v2](https://mustaqahmed.github.io/user-activation-v2) in version 72, before
which the concept of _user activation visibility_ was not defined.  With UAv2, a
user activation in a `window` object makes the activation state visible by
default to all container `window` objects (that is, those in the ancestor frames
in the frame tree, including the frame being interacted with).  We encountered a
few regressions where the developers would like to override this default
visibility.  To support such use-cases, we are proposing here an API that allows
passing on the activation state to any `window` in the frame tree.

Note, however, that our proposed API makes sense without User Activation v2 too,
even though we have [spec
limitations](https://github.com/whatwg/html/issues/1903) with the notion of user
activation.


## Proposed API for activation transfer

### Proposed spec change

[Here](https://github.com/whatwg/html/pull/4369) is the initial pull request for
whatwg/html.

### Example usage

Suppose a page has a button on the top frame, and it has a subframe that wants
to open a popup when that button on the top frame is clicked.  The default
visibility rule in UAv2 means that the subframe would be unable to open a popup
because it cannot see the user activation in its parent frame.  To let the
subframe open a popup, the top frame will transfer its own user activation to the
subframe after a click, as follows:

```javascript
// Script for top frame
function clickHandler() {
    let targetWindow = frames[0];
    targetWindow.postMessage("handle_click", {transferUserActivation: true});
}

let elem = document.getElementById("button");
elem.addEventListener("click", clickHandler);
```

```javascript
// Script for subframe
function messageReceiver(e) {
    if (e.source === window.parent && e.data === "handle_click")
        window.open("about:blank");
}

window.addEventListener("message", messageReceiver);
```


### How the transfer affects activation states

* After a transfer, the sender `window` loses the activation state, so can’t
  call any activation-gated API until it sees another user activation.  This is
  done to discourage abusive propagation of activation.

* After a transfer, the receiver `window` gets the activation state the sender
  had before.  If the receiver already had activation at the time of transfer,
  its new activation state will effectively be the union of its old state and
  the transferred-in state.  To be precise:
  - Its sticky activation bit will be set if it was already set before, or the
    sender had the bit set before (or both).
  - Its transient activation bit will now be set if it was already set before,
    or the sender had the bit set before (or both), with an expiry time that is
    larger of the two individual expiry times.

* The activation states of other frames (i.e., those except the sender and the
  receiver) are not affected by the transfer.


### Proposed IDL

The changes below are on top of `WindowPostMessateOptions` added through [JS API
for querying User Activation states](https://github.com/dtapuska/useractivation).
```WebIDL
partial dictionary WindowPostMessageOptions {
  boolean transferUserActivation = false;
};
```


### Design doc

- [Main
  design](https://docs.google.com/document/d/1NKLJ2MBa9lA_FKRgD2ZIO7vIftOJ_YiXXMYfRMdlV-s/edit?usp=sharing)
  with security considerations.

- [Privacy considerations](https://github.com/mustaqahmed/user-activation-delegation/blob/master/privacy_considerations.md).

### Alternates explored

We also explored the possibility of adding an `<iframe>` attribute to allow
activation visibility in selected subframes.  This API won't be useful when the
receiver frame is not a descendant subframe (e.g. a sibling frame in the frame
tree), or when the sender frame would transfer dynamically (e.g. only certain
clicks would be delegated instead of all).


## Demo

Work in progress.  First need to complete an implementation behind a flag in
Chrome.  Note that the API can't be done through a polyfill.


## Related links

- [User Activation v2](https://mustaqahmed.github.io/user-activation-v2): The
spec explainer for UAv2.

- [JS API for querying User Activation
  states](https://github.com/dtapuska/useractivation): An this is independent but
  somewhat related to UAv2.
