---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - 

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the veracity SDK documentation. You have begun your travels into the wonderful world of remote proctoring, and we're here to guide you!

Within this guide, you'll find explanations, code examples, and general usage guidelines for our SDK. Please note that at this time, we only support JavaScript applications that are run in a standard, standalone web browser. While other platforms such as Electron or UIWebViews *may* work, we do no make any guarantees, and any use of our SDK in those contexts is unsupported.

As you continue to read through this guide, you may have questions. Please email those to support@veracitylive.com and we'll get back to you as soon as possible.

# Overview

The veracity platform is designed to provide comprehensive, AI-based student integrity services. The SDK is designed to allow you to quickly and easily embed these capabilities into your course or test taking platform.

# Authentication

> Creating an authentication token

```javascript
var config = {
  // configuration for the SDK goes here
  // anything that needs to be signed by the server for integrity
  // see "The Server Configuration" in the "Initializing the SDK" section below
};

// sign the configuration
var signature = veracity.authorize('replace-this-with-your-key', config)

// return both the signature and the configuration used to generate it to the client
return {
  config: config,
  signature: signature
}
```

> Make sure to replace `replace-this-with-your-key` with your API key.

In order to utilize the SDK, you must first digitally sign your request. This ensures that the request is coming from you, and only from you, has exactly the parameters you have requested, and that nobody has tampered with the request. veracity uses a simple JWT token to validate all requests, which is digitally signed using your API key. You can find your key in the veracity portal under the "Settings" panel.

Please note that you can have multiple keys active at once, allowing you to deprecate older keys, transition keys, or invalidate keys that have been erroneously made available to the wrong people. Keep your keys safe from prying eyes!

<aside class="notice">
To ensure your key stays safe and your requests un-tampered, your tokens should ALWAYS be generated on the server, after your users have authenticated and authorized themselves in your system. They can then be passed to the veracity SDK on the client to configure the SDK according to the server-specified requirements.
</aside>

The sample code here shows JavaScript that can be used in a nodejs server application, but it simply generates a JWT token signed with your API key. Once you have created this token,

<aside class="notice">
You must replace <code>replace-this-with-your-key</code> with your personal API key.
</aside>

# Adding the SDK to Your App

> Reference the JavaScript

```javascript
<script src="/main.js" data-html2canvas-ignore="true"></script>
```

> Reference the CSS

```javascript
<link rel="stylesheet" type="text/css" href="/signaltracker.css"/>
```

To add the SDK into your platform, you need to add both the core JavaScript SDK, as well as a reference to the application CSS. We'll go over what CSS classes are used later so you can style up the user interface components.

# Initializing the SDK

## Creating the Signal Tracker

> Creating the Signal Tracker

```javascript
var signalTracker = new veracity.SignalTracker({
    client: {},
    server: {},
    signature: ''
});
```

To use the SDK, you start with the creation of a SignalTracker object. This object is the core component of the SDK and is where all events and methods exist.

The SignalTracker takes a single object as a parameter, a configuration that defines how the tracker will be used. The configuration contains three sub-properties:

1. `client`
1. `server`
1. `signature`

### The Client Configuration ###

```javascript
{
  maskQuery: 'body',
  contentQuery: 'body',
  onKick: function(o){}
}
```

The `client` configuration property is used for configuration that applies to the *current client-side implementation* of the SDK. This includes things like the event to fire when the user is kicked by a proctor, or when an anomaly is detected. It also includes CSS-style selectors that are used to locate elements on the page, for example the elements that should masked when using the user's webcam to take pictures or the selection of what to screen share.

In general, these properties are low-risk properties that don't impact the integrity of the signal tracking system.

Property | Description | Default
--------- | -----------| ------
maskQuery | The CSS-style selector to use when masking out the page to display the camera overlay when taking a self-portrait | `'body'`
contentQuery | The CSS-style selector to use as the capture source when capturing the screen during a session | `'body'`
onKick | The handler that is invoked due to a proctor-initiated kick event. The only argument is an object with a 'reason' property describing why the user was kicked out. | Redirect to /

### The Server Configuration ###

```javascript
{
  studentId: 'some-guid-or-other-id-goes-here',
  studentAlias: 'John Doe',
  studentCode: 'johndoe',
  companyId: 'some-guid-or-other-id-goes-here',
  companyAlias: 'Goliath National Bank',
  companyCode: 'goliath',
  courseId: 'some-guid-or-other-id-goes-here',
  courseAlias: 'Teller Training',
  courseCode: 'teller-training',
  sessionId: 'some-guid-or-other-id-goes-here',
  sessionAlias: 'Bank Secrecy Act Final Exam',
  sessionCode: 'bank-security-act-final',
  vertical: 'Finance',
  stakes: 'medium',
  signals: [
    'keyboardactivity',
    'mouseactivity', 
    'browserfocus',
    'browservisibility',
    'identity',
    'attention',
    'multiplefaces',
    'multiplevoices'
  ]
}
```

The `server` configuration property is used to configure components of the system that require a signature from the server to ensure integrity. When the signature is generated on the server, it is created using a specific `config` object. The full `config` that was used on the server MUST be passed as the `server` configuration property to the SignalTracker when it is created. The signed configuration is verified by veracity to ensure that there has not been any tampering with the configuration, thus ensuring all ids, requirements, names, etc, are exactly what the platform requested. 

The "ID" and "Code" properties for the student, company, course, and session are all for your use, for tracking and reporting purposes. They will NOT be used within veracity for display, etc.

The "Alias" properties for the student, company, course, and session will be used to display information within veracity. 

Property | Description | Default
--------- | -----------| ------
studentId | The ID of the student, as specified by the originating system | `Guid.newGuid()`
studentAlias | A human-readable name for this student | *none, required field*
studentCode | A code for the student, often simply the username | `''`

### The Signature ###

```javascript
var signature = '=AE1234567';
```

The `signature` property is simply used to store the signature corresponding to the `server` config object. On the veracity servers, the `signature` will be re-generated by the veracity platform using the API key and the `config` object, and validated against the incoming `signature` value.

<aside class="warning">
Remember — do NOT generate your signature client-side! This opens you up to potential cheaters!
</aside>

# Putting It Together
