---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - 

toc_footers:
  - <a href='#'>Sign Up for a Developer Key (coming soon)</a>

includes:
  

search: true
---

# Introduction

Welcome to the <a href="https://www.veracitylive.com/">veracity</a> SDK documentation. You have begun your travels into the wonderful world of AI-based student integrity, and we're here to guide you!

Within this guide, you'll find explanations, code examples, and general usage guidelines for our SDK. Please note that at this time, we only support JavaScript applications that are run in a standard, standalone web browser. While other platforms such as Electron or UIWebViews *may* work, we do no make any guarantees, and any use of our SDK in those contexts is unsupported.

As you continue to read through this guide, you may have questions. Please email those to support@veracitylive.com and we'll get back to you as soon as possible.

# Overview

The veracity platform is designed to provide comprehensive, AI-based student integrity services. The SDK is designed to allow you to quickly and easily embed these capabilities into your course or test taking platform.

# Security

> Creating a secure token

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

These configuration sub-objects are listed below, including which components are required vs optional. Generally speaking, however, you need to identify only the student, institution, course, and session being taken. Everything else is essentially extra.

# Configuring the SDK

### The Client Configuration ###

> Client Configuration

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

> Server Configuration

```javascript
{
  studentId: 'some-guid-or-other-id-goes-here',
  studentAlias: 'John Doe',
  studentCode: 'johndoe',
  institutionId: 'some-guid-or-other-id-goes-here',
  institutionAlias: 'Goliath National Bank',
  institutionCode: 'goliath',
  programId: 'some-guid-or-other-id-goes-here',
  programAlias: 'Teller Training',
  programCode: 'teller-training',
  courseId: 'some-guid-or-other-id-goes-here',
  courseAlias: 'Teller 101',
  courseCode: 'teller-101',
  sessionId: 'some-guid-or-other-id-goes-here',
  sessionAlias: 'Teller 101 Final Exam',
  sessionCode: 'teller-101-final',
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

A large selection of these properties are grouped by `student`, `institution`, `program`, `course`, and `session`. The general definition for these is as follows:

  * Student: The student launching the test, course, etc
  * Institution: The institution to which this student belongs
  * Program: The over-arching program the test or course or material to which the course belongs
  * Course: The course to which the session belongs
  * Session: An individual launch of a course

The JSON displayed on the right shows an example of a teller taking their bank training. The program would be their teller training courses ("Teller Training"), the course itself would be a specific training course ("Teller 101"), and a session would be the test they're taking ("Teller 101 Final Exam")

For another example, consider an electrical engineering student getting their degree at NC State. The instituion would be "NC State", the program would be "Electrical Engineering", the course would be "ECE101", and the session would be "Final Exam".

The "ID" properties for the student, institution, course, and session are all for your use, for your own tracking and reporting purposes. They will NOT be used within veracity for any purpose whatsoever.

The "Alias" properties for the student, institution, course, and session will be used to display information within the veracity portal, and may be used for reporting and analysis.

The "Code" properties for the student, institution, course, and session will not be used for display, but may be used for reporting and analysis.

Property | Description | Default
--------- | -----------| ------
studentId | The ID of the student, as specified by the originating system | `Guid.newGuid()`
studentAlias | A human-readable name for this student | *none, required field*
studentCode | A code for the student, often simply the username | `''`
institutionId | The ID of the institution, as specified by the originating system | `Guid.newGuid()`
institutionAlias | A human-readable name for this institution, often the full name or doing-business-as name | *none, required field*
institutionCode | A code for the institution, in the case of multi-tenanted solutions this can be the subdomain | `''`
programId | The ID of the program, as specified by the originating system | `Guid.newGuid()`
programAlias | A human-readable name for this program | *none, required field*
programCode | A code for the program, commonly an SKU or internal reference code | `''`
courseId | The ID of the course, as specified by the originating system | `Guid.newGuid()`
courseAlias | A human-readable name for this course | *none, required field*
courseCode | A code for the course, commonly an SKU or internal reference code | `''`
sessionId | The ID of the session, as specified by the originating system | `Guid.newGuid()`
sessionAlias | A human-readable name for this session | *none, required field*
sessionCode | A code for the session, commonly an SKU or internal reference code | `''`

In addition, there are a handful of other properties that can be defined.

> Signals

```javascript
[
  'keyboardactivity',
  'mouseactivity', 
  'browserfocus',
  'browservisibility',
  'identity',
  'attention',
  'multiplefaces',
  'multiplevoices'
]
```

Property | Description | Default
--------- | -----------| ------
vertical | The industry vertical to which this student and course belong. Useful in reporting. | *none*
stakes | The type of test metrics to be applied | `'medium'`
signals | An array of signals to be tracked. Possible values can be seen in the JSON to the right. The settings will be applied to the signals based on the `stakes` parameter and the settings specified in the veracity console. | []


### The Signature ###

```javascript
var signature = '=AE1234567';
```

The `signature` property is simply used to store the signature corresponding to the `server` config object. On the veracity servers, the `signature` will be re-generated by the veracity platform using the API key and the `config` object, and validated against the incoming `signature` value.

<aside class="warning">
Remember — do NOT generate your signature client-side! This opens you up to potential cheaters!
</aside>

# Starting the Client

> Starting the Client

```javascript
signalTracker
  .start()
  .then(p=>{
    // continue with launching your test or course
  })
  .catch(err=>{
    // log the error and redirect the student out
  })
```
Once you have created the client, it's time to start it up. The `start()` function returns a promise, because depending on what elements you have requested, it may need to run async. If there is a failure initializing, the promise will fail and the `catch` will run and give an appropriate error code (see below). If the start is successful, the `then` will exeucte and you can continue with your test or course launch.

<aside class="warning">
The `start` MUST be called AFTER the document has loaded (the DOMContentLoaded event has fired). Failure to ensure this is the case can result in the veracity platform not executing properly. See "Putting It Together" for a full example.
</aside>

Once the tracker is started, it will continue until the page is reloaded or you call `signalTracker.stop()`.

<aside class="note">
PLEASE NOTE: there is no guarantee as to how long the start() operation will take. Sometimes access to webcams are required, or self-portraits need to be taken, so if you have any code relying on timing to execute, be careful!
</aside>

# Changing Styles
The veracity SDK sometimes needs to display information to the end user (student), such as communication from the proctor (or vice-versa), a display to take a self portrait, and so on. These CSS classes are defined in signaltracker.css, and all begin with .veracity*. If you wish to override them, you may do so by simply adding a new `<style>` block to your html that has a slightly more specific CSS target. We do not deliberately change the CSS classes without notifying our customers of the pending change.

We do NOT recommend changing positioning or display elements, as this can break functionality of the system; colors, fonts, and backgrounds are typically fine.

# Putting It Together

Now that you have the SDK figured out, here's a complete sample to show you how it all looks together.

> Putting It Together - Server

```javascript

// this code lives on your backend server, and should be loaded up in your test or course when it launches
var config = {
  studentId: 'some-guid-or-other-id-goes-here',
  studentAlias: 'John Doe',
  studentCode: 'johndoe',
  institutionId: 'some-guid-or-other-id-goes-here',
  institutionAlias: 'Goliath National Bank',
  institutionCode: 'goliath',
  programId: 'some-guid-or-other-id-goes-here',
  programAlias: 'Teller Training',
  programCode: 'teller-training',
  courseId: 'some-guid-or-other-id-goes-here',
  courseAlias: 'Teller 101',
  courseCode: 'teller-101',
  sessionId: 'some-guid-or-other-id-goes-here',
  sessionAlias: 'Teller 101 Final Exam',
  sessionCode: 'teller-101-final',
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
var token = veracity.authorize('replace-this-with-your-key', config)
return {
  token: token,
  config: config
}
```

First, sign your request, and return that information to the client. You can use a custom endpoint, load this up inline in your page, whatever method makes sense for your test/course launch mechanism.

For this example, we will assume the endpoint is "/veracity"

> Putting It Together - Client

```javascript
<script src="/main.js" data-html2canvas-ignore="true"></script>
<link rel="stylesheet" type="text/css" href="/signaltracker.css"/>
<script data-html2canvas-ignore="true">
  window.addEventListener('DOMContentLoaded', function(){
    // load up the server config
    $.ajax({
      url: "/veracity"
    }).done(function(data) {
      // create the signal tracker
      var signalTracker = new veracity.SignalTracker({
        client: {
          maskQuery: 'body',
          contentQuery: '.body-container',
          onKick: function(o){
              alert('You have been suspended from this course. ' + o.reason)
              window.top.location.href = '/'
          }
        },
        // pass the server config and signature
        server: data.config,
        signature: data.token
      });

      // start tracking!
      signalTracker
        .start()
        .then(p=>{
          // continue any sort of course or test timer initiation
        })
        .catch(function(err){
            window.top.location.href = '/'
        })
    });
  });
</script>
```

Now we've loaded up the config, created our signal tracker, and started it up. That's it - everything else is take care of on the veracity portal.

Happy tracking!