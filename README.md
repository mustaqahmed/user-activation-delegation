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
v2](https://github.com/mustaqahmed/user-activation-v2) in version 72, before
which the concept of _user activation visibility_ was not defined.  With UAv2, a
user activation in a `window` object makes the activation state visible by
default to all container `window` objects (that is, those in the ancestor frames
in the frame tree, including the frame being interacted with).  We encountered a
few regressions where the developers would like to override this default
visiblity.  To support such use-cases, we are proposing here an API that allows
passing on the activation state to any `window` in the frame tree.


## Proposed API for activation transfer

### Example usage

Suppose a page has a button on the top frame, and it has a subframe that wants
to open a popup when that button on the top frame is clicked.  The default
visibility rule in UAv2 means that the subframe would be unable to open a popup
because it cannot see the user activation in its parent frame.  To let the
subframe open a popup, the top frame will transfer its own user ativation to the
subframe after a click, as follows:

```javascript
// Script for top frame
function clickHandler() {
    let targetWindow = frames[0];
    targetWindow.postMessage("I_got_a_click", {transferUserActivation: true});
}

let elem = document.getElementById("button");
elem.addEventListener("click", clickHandler);


// Script for subframe
function messageReceiver(e) {
    if (e.source === window.parent && e.data === "I_got_a_click")
        window.open("about:blank");
}

window.addEventListener("message", messageReceiver);
```


### Proposed IDL

The changes below are on top of `PostMessateOptions` added through [JS API for
querying User Activation states](https://github.com/dtapuska/useractivation).
```webidl
partial dictionary PostMessageOptions {
  boolean transferUserActivation = false;
};
```


### Design doc

- [Main
  design](https://docs.google.com/document/d/1NKLJ2MBa9lA_FKRgD2ZIO7vIftOJ_YiXXMYfRMdlV-s/edit?usp=sharing)
  with security considerations.


### Alternates explored

We also explorerd the possibility of adding an `<iframe>` attribute to allow
activation visibility in selected subframes.  This API won't be useful when the
receiver frame is not a descedant subframe (e.g. a sibling frame in the frame
tree), or when the sender frame would transfer conditionally.


## Demo

_TODO: Work in progress._


## Related links

- [User Activation v2](https://github.com/mustaqahmed/user-activation-v2): The
spec explainer for UAv2.

- [JS API for querying User Activation
  states](https://github.com/dtapuska/useractivation): An this is independent but
  somewhat related to UAv2.
