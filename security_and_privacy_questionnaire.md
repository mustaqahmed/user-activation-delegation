# Answers to [Security and Privacy Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/)

### 3.1 Does this specification deal with personally-identifiable information?

No.


### 3.2 Does this specification deal with high-value data?

No.


### 3.3 Does this specification introduce new state for an origin that persists across browsing sessions?

No.


### 3.4 Does this specification expose persistent, cross-origin state to the web?

No.


### 3.5 Does this specification expose any other data to an origin that it doesn’t currently have access to?

No by default.

The proposed spec allows a frame to transfer its own activation state to a
target frame of a different origin through the new dictionary entry in
`WindowPostMessageOptions`.  This is not a concern because of the following
reasons:
- By default no transfer takes places.  The sender has to explicitly ask for a
  transfer when it needs.
- The sender frame is in full control here; a receiver frame of a rogue origin
  can't get the state without sender's action.
- If the sender wants, it can already expose to a target frame the fact that
  user is interacting (or has interacted) with it (say, through a traditional
  `postMessage()`).


### 3.6 Does this specification enable new script execution/loading mechanisms?

No.


### 3.7 Does this specification allow an origin access to a user’s location?

No.


### 3.8 Does this specification allow an origin access to sensors on a user’s device?

No.


### 3.9 Does this specification allow an origin access to aspects of a user’s local computing environment?

No.


### 3.10 Does this specification allow an origin access to other devices?

No.


### 3.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

No.


### 3.12 Does this specification expose temporary identifiers to the web?

No.


### 3.13 Does this specification distinguish between behavior in first-party and third-party contexts?

No.


### 3.14 How should this specification work in the context of a user agent’s "incognito" mode?

This would work in the "incognito" mode in the same way as in the "regular" mode.


### 3.15 Does this specification persist data to a user’s local device?

No.


### 3.16 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

There is no privacy concerns here: the only user info that gets exposed to the
receiver frame is whether the user has interacted with a page.  This info can be
exposed already through a traditional `postMessage()`, see Q3.5 above.

We don't see any security risk in the proposed spec.  We have already considered
possible security issues in the general idea of activation transfer (see
[here](https://docs.google.com/document/d/1NKLJ2MBa9lA_FKRgD2ZIO7vIftOJ_YiXXMYfRMdlV-s/edit#heading=h.ujagf0s90yap)),
and tweaked the design accordingly.


### 3.17 Does this specification allow downgrading default security characteristics?

No.

