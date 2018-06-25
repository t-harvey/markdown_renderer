ZJS API for Timers
==================

* [Introduction](#introduction)
* [Web IDL](#web-idl)
* [Class: Timers](#timers-api)
  * [timers.setInterval(func, delay, extra_args)](#timerssetintervalfunc-delay-extra_args)
  * [timers.setTimeout(func, delay, extra_args)](#timerssettimeoutfunc-delay-extra_args)
  * [timers.clearInterval(intervalID)](#timersclearintervalintervalid)
  * [timers.clearTimeout(timeoutID)](#timerscleartimeouttimeoutid)
* [Sample Apps](#sample-apps)

Introduction
------------
ZJS provides the familiar setTimeout and setInterval interfaces. They are always
available.

Web IDL
-------
This IDL provides an overview of the interface; see below for
documentation of specific API functions.  We have a short document
explaining [ZJS WebIDL conventions](Notes_on_WebIDL.md).

<details>
<summary>Click to show WebIDL</summary>
<pre>
// require returns a Timers object
// var timers = require('timers');
<p>
[ReturnFromRequire]
interface Timers {
    intervalID setInterval(TimerCallback func, unsigned long delay, any... extra_args);
    timeoutID setTimeout(TimerCallback func, unsigned long delay, any... extra_args);
    void clearInterval(long intervalID);
    void clearTimeout(long timeoutID);
};<p>
callback TimerCallback = void (any... callback_args);</pre>
</details>

Timers API
----------
### timers.setInterval(func, delay, extra_args)
* `func` *TimerCallback* A callback function that will take the arguments passed in the variadic `extra_args` parameter.
* `delay` *unsigned long* The `delay` argument is in milliseconds. Currently, the delay resolution is about 10 milliseconds, and if you choose a value less than that it will probably fail.
* `extra_args` *any* The user can pass an arbitrary number of additional arguments that will then be passed to `func`.
* Returns: an `intervalID` object that can be passed to `clearInterval` to stop the timer.

Every `delay` milliseconds, your callback function will be called.

### timers.setTimeout(func, delay, extra_args)
* `func` *TimerCallback* A callback function that will take the arguments passed in the variadic `extra_args` parameter.
* `delay` *unsigned long* The `delay` argument is in milliseconds. Currently, the delay resolution is about 10 milliseconds.
* `extra_args` *any* The user can pass an arbitrary number of additional arguments that will then be passed to `func`.
* Returns: a `timeoutID` that can be passed to `clearTimeout` to stop the timer.

After `delay` milliseconds, your callback function will be called *one time*.

### timers.clearInterval(intervalID)
* `intervalID` *long* This value was returned from a call to `setInterval`.

That interval timer will be cleared and its callback function
no longer called.

### timers.clearTimeout(timeoutID)
* `timeoutID` *long* This value was returned from a call to `setTimeout`.

The `timeoutID` timer will be cleared and its callback function will not be
called.

Sample Apps
-----------
* [Timers sample](../samples/Timers.js)
* [Spaceship2 sample](../samples/arduino/starterkit/Spaceship2.js)
