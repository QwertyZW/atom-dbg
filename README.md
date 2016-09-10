# dbg package

An interactive debugger frontend

![Debug screenshot](http://i.imgur.com/Nv5StoZ.png)

Supports:

* Breakpoints
* Continue/pause/stop/step-over/step-in/step-out
* Call stack display
* Local variable display
* Program output (via https://atom.io/packages/output-panel)

## Commands:

`dbg:continue` - Continue  
`dbg:pause` - Pause  
`dbg:pause-continue` - Toggle pause/continue  
`dbg:stepOver` - Step over the current line of code  
`dbg:stepIn` - Step into the current procedure  
`dbg:stepOut` - Complete and step out of the current procedure  
`dbg:stop` - Stop debugging  
`dbg:toggle-breakpoint` - Toggle a breakpoint on the currently active line  
`dbg:clear-breakpoints` - Clear all breakpoints  

## Service: `dbg`

Returns an object with the following events and functions:

### Events:

`start` - A debug session has begun  
`stop` - A debug session has ended  

### Functions:

`debug(options):Promise<Bool>` - Attempt to start a new debug session (any existing session will be `stop()`ed first)

Example: `dbg.debug({ path: './myProg', args: ['testmode'] });`  
> `options` - An object with the following properties:

>> `debugger` - Optional. The name of the dbg provider to use. (This can be omitted to auto-detect)  
>> `path` - Optional. The path to the file to debug  
>> `args` - Optional. An array of arguments to pass to the file being debugged  
>> `cwd` - Optional. The working directory to use when debugging  
>> `...` - Optional. Custom debugger arguments  

> If a `debugger` is not specified, every dbg provider will be queried with the options object provided;  
> The first provider to respond as being compatible will be selected  
> Note that the options supported vary depending on dbg providers, but `path`, `args` and `cwd` are recommended as defaults if relevant to the target  

> Returns a promise that fulfills to `true` if a valid dbg provider is found, or `false` if not

`stop()` - Cancel the currently active debug session  

`continue()` - Continue  
`pause()` - Pause  
`pause_continue()` - Pause or continue (toggle)

`stepOver()` - Step over the current line of code  
`stepIn()` - Step into the current procedure  
`stepOut()` - Complete and step out of the current procedure  

`addBreakpoint(path:String, line:Int)` - Add breakpoint at the specified location  
> `path` - The absolute file path  
> `line` - The line number (1-based)  

`removeBreakpoint(path:String, line:Int)` - Remove a breakpoint at the specified location  
> `path` - The absolute file path  
> `line` - The line number (1-based)  

`toggleBreakpoint(path:String, line:Int)` - Toggle a breakpoint at the specified location  
> `path` - The absolute file path  
> `line` - The line number (1-based)  

`getBreakpoints()` - Return an array of all breakpoints. Each array item has the following properties:  
> `path` - The absolute file path  
> `line` - The line number (1-based)  

`hasBreakpoint(path:String, line:Int)` - Toggles a breakpoint at the specified location  
> `path` - The absolute file path  
> `line` - The line number (1-based)  

## Consumed service: `dbgProvider`

To create a dbg provider provide a `dbgProvider` service with the following properties:

`name` - The name of this provider  
`description` - A brief one-line description of this provider  

`canHandleOptions(options):Promise<Bool>` - A function that returns a promise that fulfilling as `true` if it is compatible with the provided debug options, or `false` if it is not
> `options` - The options parameter passed to `debug()` of the `dbg` service

`debug(options, api)` - A function to begin debugging  
> `options` - The options parameter passed to `debug()` of the `dbg` service  
> `api` - An object containing the following properties:
>> `breakpoints` - An array of all breakpoints currently present  
>> Each item in the array contains the following properties:
>>> `path` - The absolute file path  
>>> `line` - The line number (1-based)  

>> `ui` - An object containing the following functions for controlling the dbg frontend:  
>>> `setStack(stack:Frame[])` - Set the stacktrace display to the specified items.  
>>> Frames are ordered from bottom to top, with index 0 being the entrypoint of the program and the highest index being the current executing location

>>> Each `Frame` should have the following properties:  
>>>> `name` - The friendly name of this location (`"main()"` etc)  
>>>> `local:Bool` - Specifes if the location is locally accessible, or a system location (hidden by default unless at the top of the stack)  
>>>> `path` - The absolute or relative filepath  
>>>> `line` - The line number (1-based)  

>>> `setVariables(variables:Variable[])` - The list of variables visible in the currently active frame

>>>  Each `Variable` should have the following properties:
>>>> `name` - The name of the variable  
>>>> `type` - Optional. The type of the variable (as a String)  
>>>> `value` - Optional. The value of the variable (as a String)  

>>> `setFrame(frameIndex:Int)` - Set this index in the stack as the currently active frame, and navigate the Atom to this file and location if it exists  
>>> If this frame in the stack is a system location then all system frames will be shown

>>> `running()` - Mark the debug as current running  
>>> `paused()` - Mark the debug as currently paused  
>>> `stop()` - Abort the current debug in process (this will call `stop()` for the `dbg` service internally and then `stop()` on this packages `dbgProvider`)  

>>> `showWarning(warning:String)` - Display the specified warning  
>>> `showError(error:String)` - Display the specified error  

`stop()` - This function is called when debugging has ceased  
Once this has been called the `api` property passed to `start()` should no longer be accessed  

`continue()` - This function is called when the user request the debugger continue  
`pause()` - This function is called when the user request the debugger pause  

`selectFrame()` - This function is called when the user selects a frame from the stacktrace  

`stepOver()` - This function is called when the user requests a step over  
`stepIn()` - This function is called when the user requests step into  
`stepOut()` - This function is called when the user requests step out  

`addBreakpoint(breakpoint:Breakpoint)` - This function is called if the user adds a breakpoint during an active debug session  
The breakpoint contains the following properties:
> `path` - The absolute file path  
> `line` - The line number (1-based)  

`removeBreakpoint(breakpoint:Breakpoint)` - This function is called if the user removes a breakpoint during an active debug session  
The breakpoint contains the following properties:
> `path` - The absolute file path  
> `line` - The line number (1-based)  
