# Privacy considerations for activation Transfer through postMessages

This document is a supplement to our [proposed
API](https://mustaqahmed.github.io/user-activation-v2) for transferring user
activation through postMessages.  We discuss here possible privacy implications
of the API.


## Why user privacy is relevant here

Browsers handle lots of sensitive data on behalf of the user, so any API that is
even remotely related to user data should be careful about the privacy
implications.  The proposed API here transfers user interaction information to
another frame (which could be a cross-origin frame), allowing the receiver frame
to call [user activation gated
APIs](https://mustaqahmed.github.io/user-activation-v2/#classifying-user-activation-gated-apis).
So it is natural to ask if the proposed API can possibly leak some information
to the receiver frame or allow the receiver frame to perform any
privacy-sensitive task without the user's knowledge.


## Why the proposed API is safe from privacy perspective

The ability to transfer user activation state to another frame doesn't impact
user privacy because this neither passes on any sensitive data nor enables
collection of new sensitive data.  Below is our explanation.

### Won't transfer sensitive information

The information that is passed to the receiver frame via a transfer is
essentially two bits: whether the user is interacting with the source frame, and
whether the user has ever interacted with the source frame.  None of these bits
is sensitive from privacy perspective.


### There are other existing ways to transfer the same information

The two bits of information mentioned above can be communicated to the receiver
frame using well-known existing APIs.

For example, the sender frame can add the
following event handler at `Window` for all events representing user interaction
(`mousedown`, `pointerdown`, `touchend`, `click` etc):

```javascript
function eventHandler(event) {
  if (event.isTrusted)
    receiverWindow.postMessage("user_interaction_happened");
}
```

Thus the receiver frame can deduce and maintain the two bits of information (as
well as additional information like the precise timing of the user interaction)
without using the proposed transfer API.

In other words, the proposed API doesn't enable the sender to reveal any new
information that it can't communicate today.


### There are other existing ways to activate a target frame

In addition to passing the two bits mentioned above, the transfer API also
activates the target frame.  That means a frame can get activated without the
user realizing it.  (We will show in the next section that such activation is
okay.)  There are existing ways to achieve this today.  For example, the top
frame can have an opaque element that hides an `<iframe>` behind it.  If the
opaque element has the style `pointer-events:none`, all clicks on the element
would go to the `<iframe>` behind it, activating that frame without user's
knowledge.


### Activating a cross-origin frame doesn't mean unnoticed data collection

We investigated [the activation-gated
APIs](https://docs.google.com/document/d/1mcxB5J_u370juJhSsmK0XQONG2CIE3mvu827O-Knw_Y)
in Chrome to see if the transfer API could somehow enable any of them to collect
sensitive data from the target (possibly cross-origin) frame without user's
knowledge.  We didn't find a single case like that.

Almost all the APIs in the list rely on user activation to prevent annoying
behavior by a script, and they are unrelated to privacy (e.g. popups,
full-screen, vibration etc.).  The remaining APIs need a closer look.

The remaining APIs always reject calls when there is no user activation, and
show permission dialogs _in most cases_ when there is user activation.  Let's
first rule out the easy case: when a permission dialog is shown, these APIs are
clearly safe from privacy perspective since the user gets notified about the
usage.

What remains is the case that the permission dialog is _not_ shown.  Is there a
chance that the transfer target can access the API without user's knowledge?
The answer is "no" even in this case.  For ease of discussion, we will pick the
[Geolocation
API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API) since
this is most privacy-sensitive among those remaining APIs (but the argument
below is valid for any other APIs there).

Assume that a site contains a cross-origin subframe that uses the Geolocation
API.  The Geolocation access from the subframe here is allowed without any
permission dialog only when the user has previously allowed it (by hitting
"allow" on a past permission dialog), which means the user already trusts this
site.  We verified this behavior in both Chrome and Firefox using mouse clicks
on [this
demo](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API#Live_Result).
The same argument then follows for activation transfer: the only case when the
target frame can silently access Geolocation after a transfer from the top frame
is that user already allowed the access before.

In other words, the transfer operation doesn't create a way for unnoticed data
collection.
