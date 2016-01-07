nodemcu-timer
========================
Current version: v1.0.0 (stable)
<br />

## Table of Contents

1. [Overiew](#Overiew)  
2. [Installation](#Installation)  
3. [APIs](#APIs)  

<a name="Overiew"></a>  
## 1. Overview  

The [**nodemcu**](https://github.com/nodemcu/nodemcu-firmware) `tmr` module has 7 timers (id=0~6) for you to schedule your things with `tmr.alarm()`. You would face a problem of managing callbacks with the specified timer ids to start or stop the scheduled alarms. This is the reason why **nodemcu-timer** comes out. With **nodemcu-timer**, you don't have to worry about whether a timer is available or not. It is a soft timer utility that allows you to schedule tasks with Javascript(/node.js) style APIs in your **nodemcu** project, i.e., `setTimeout()` and `clearTimeout()`.  
  
**nodemcu-timer** uses a single timer to simulate the behavior of `setTimeout()`, `setInterval()`, and `setImmediate()`. Only one task executes when a tick fires (excepts those scheduled by [`setImmediate()`](#API_setImmediate)), thus **nodemcu-timer** does not guarantee that the callback will fire at exact timing but as close as possilbe. When a callback is _setImmediate_, it will be executed at right next tick immediately even if there are other tasks being due at the same time. The internal timer will automatically start when a scheduled task enqueues, and automatically stopped when there is no task in queues. **nodemcu-timer** uses only a single timer (id=6 by default) internally, and you are free to use other 5 timers that `tmr` provides.

[**Note**]  
This module internally polls its task queues every 2ms. When you call `setTimeout()` and `setInterval()`, it is better to give the `delay` with an even number, e.g., `setTimeout(callback, 2000)` will fire the callback in 2 seconds. It is okay for `delay` to be odd, it will minus 1 to be even implicitly and will result in a timing error as small as 1ms. I think this small error is negligible if you are scheduling your task with an interval up to few hundreds of ms.  
(I didn't use a tick of 1ms, because `tmr` is a bad ass when you give him a repeat interval of 1ms. If you do so, nodemcu will crash and I don't know why.)


<a name="Installation"></a>
## 2. Installation

> $ git clone https://github.com/simenkid/nodemcu-timer.git
  
Just include the file `timer.lua` or use the minified one `timer_min.lua` in your project.  
If you are with the **nodemcu** on ESP8266, it would be good for you to compile `*.lua` text file into `*.lc` to further lower memory usage.  

<a name="APIs"></a>
## 3. APIs

* [timer.setTimeout()](#API_setTimeout)
* [timer.clearTimeout()](#API_clearTimeout)
* [timer.setInterval()](#API_setInterval)
* [timer.clearInterval()](#API_clearInterval)
* [timer.setImmediate()](#API_setImmediate)
* [timer.clearImmediate()](#API_clearImmediate)
* [timer.set()](#API_set)

*************************************************
### timer utility
Exposed by `require 'nodemcu-timer'`  
  
```lua
local timer = require 'timer'  -- or 'timer_min'
```

<br />

<a name="API_setTimeout"></a>
### setTimeout(callback, delay[, ...])
Schedules a one-time callback after `delay` ms. This API returns a timer-object(`tobj`) for possible use with clearTimeout(). You can also pass parameters to the callback via the variadic arguments.

  
**Arguments:**  

1. `callback` (_function_): The function to be scheduled.
2. `delay` (_number_): Time in milisecond, an even number would be better.
3. `...` (_variadic arguments_): The arguments pass to your callback.

**Returns:**  
  
* (_object_) Timer-object.

**Examples:**

```lua
-- fires after 10 seconds
local tobj = timer.setTimeout(function ()
    print('I am fired')
end, 10000)

```
  
********************************************

<a name="API_clearTimeout"></a>
### clearTimeout(tobj)
Removes the timeout timer-object from triggering.
  
**Arguments:**  

1. `tobj` (_object_): Timer-object to remove.

**Returns:**  
  
* (_none_) nil

**Examples:**

```lua
local greet = function ()
    print('Bonjour!')
end

local tmout = timer.setTimeout(greet, 5000)

timer.clearTimeout(tmout)
```
  
********************************************

<a name="API_setInterval"></a>
### setInterval(callback, delay, ...)
Schedules a callback to be executed every `delay` ms. This API returns a timer-object(`tobj`) for possible use with clearInterval(). You can also pass parameters to the callback via the variadic arguments.  

If `delay` is larger than 2147483647 ms (~25 days) or less than 2, **nodemcu-timer** will use 2 as the `delay`. 
  
**Arguments:**  

1. `callback` (_function_): The function to be scheduled.
2. `delay` (_number_): Time in milisecond, an even number would be better.
3. `...` (_variadic arguments_): The arguments pass to your callback.
  
**Returns:**  
  
* (_Object_) Timer-object.

**Examples:**

```lua
local tobj = timer.setInterval(function ()
    print('Hello')
end, 2000)
```
  
********************************************

<a name="API_clearInterval"></a>
### clearInterval(tobj)
Removes a time-object of interval from repeatly triggering.
  
**Arguments:**  

1. `tobj` (_Object_): Timed-object to remove.
  
**Returns:**  
  
* (_none_) nil

**Examples:**

```lua
local count = 0
local repeater = timer.setInterval(function ()
    count = count + 1
    print(count ' times triggered!')
end, 1000)

timer.clearInterval(repeater)
```
  
********************************************

<a name="API_setImmediate"></a>
### setImmediate(callback, ...)
Schedules a callback to be immediately executed at next tick, its priority is higher than those tasks set by `setTimeout()` and `setInterval()`. This API returns a timer-object(`tobj`) for possible use with clearImmediate(). You can also pass parameters to the callback via the variadic arguments.  

The callbacks for immediate execution enqueues in the order in which they were created. All immediate callbacks in the queue will be invoked right away when a tick fires. If you queue an immediate callback from inside an executing callback, that immediate callback won't be invoked until the next tick comes. Remember that do not shcedule a long task for immediate execution as possible.

  
**Arguments:**  

1. `callback` (_function_): The function to be scheduled.
2. `...` (_variadic arguments_): The arguments pass to your callback.
  
**Returns:**  
  
* (_Object_) Timer-object.

**Examples:**

```lua
local tobj = timer.setImmediate(function ()
    print(gpio.read(0))
end)
```
  
********************************************

<a name="API_clearImmediate"></a>
### clearImmediate(tobj)
Removes an immediate time-object from triggering. 
  
**Arguments:**  

1. `tobj` (_Object_): Timed-object to remove.

**Returns:**  
  
* (_none_) nil

**Examples:**

```lua
local tobj = timer.setImmediate(function ()
    print(gpio.read(0))
end)

timer.clearImmediate(tobj)
```
  

********************************************

<a name="API_set"></a>
### set(tid)
Change the internal timer accroding to the specified timer id (default is 6). Be aware of that all callbacks in queue will be cleared when you change to a new timer. It is suggested to use this function within your code of initialization only.

 You can use the property `timer.id` to know which timer is used as an internal one.
  
**Arguments:**  

1. `tid` (_number_): Id of the timer to use. If it is the same with the current id, nothing will happen.

**Returns:**  
  
* (_none_) nil

**Examples:**

```lua
print(timer.id)    -- 6

timer.set(2)
print(timer.id)    -- 2
```

  
<br />
********************************************
<br />
## License  
MIT