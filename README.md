# global-event-system-ue4
A loosely coupled internal global event system (GES) plugin for the Unreal Engine. Aims to solve cross-map and cross-blueprint communication for reliable and inferable event flow.

Questions? See https://github.com/getnamo/global-event-system-ue4/issues

## Quick Install & Setup ##
 1. [Download Latest Release](https://github.com/getnamo/global-event-system-ue4/releases)
 2. Create new or choose project.
 3. Browse to your project folder (typically found at Documents/Unreal Project/{Your Project Root})
 4. Copy *Plugins* folder into your Project root.
 5. Plugin should be now ready to use.

## How to use - Basics and API

As the name implies, there are globally available functions that you can use to emit and bind events. At this time there are two variants for emitting (no parameters and one wildcard parameter) and one for binding events to your local functions. Each emit is a multi-cast to all valid bound receivers. If the parameters don't match you'll be warned in the log with fairly verbose messages while emitting to all other valid targets. Consider optionally attaching interfaces to your receivers to keep message types structured.

### Emit Event

#### ```GESEmitEvent```

##### Param: Pinned
Whether the event should trigger for listeners added after the event has fired. Useful to communicate state.

##### Param: Domain
A string type similar to a namespace with reverse-DNS like structure encouraged, but not enforced. By default there is a ```global.default``` prefilled which can be changed to any valid utf8 string.

##### Param: Event
This is an abstract name and is considered unique for that domain. You'll need the same domain and event name to match in your emitting function to receive the event.

#### ```GESEmitEventOneParam```

##### Additional Param: Parameter Data
Wildcard Property, will accept any single property type e.g. *int, float, byte, string, name, bool, struct,* and *object*. Wrap arrays and maps in a custom struct to emit more complex data types. Break pin to set a new type of value. Keep in mind that the receiving listeners need to match the property type to receive the data.

![emit](https://i.imgur.com/8nXb5ya.png)

### Bind Event

```GESBindEvent```
##### Param: Domain
A string type similar to a namespace with reverse-DNS like structure encouraged, but not enforced. By default there is a ```global.default``` prefilled which can be changed to any valid utf8 string.

##### Param: Event
This is an abstract name and is considered unique for that domain. You'll need the same domain and event name to match in your emitting function to receive the event.

##### Param: Receiving Function
The final parameter is your local function name. This will be called on the graph context object (owner of graph e.g. the calling actor).

![bind](https://i.imgur.com/WzHhEeG.png)

Then make your custom event or blueprint function with a matching name and matching parameters.

### Options

There are some simple options to toggle some log messages and detailed struct type checking.

![options](https://i.imgur.com/22tC4lI.png)

## Examples

Keep in mind that you can start using GES incrementally for specific tasks or parts of large projects instead of replacing tpo many parts at once. Below are some simple examples where GES could be useful.

### Cross-map reference pinning
Let's say you had two actors in two different sub-maps and you wanted one actor to know that it has spawned from e.g. some dynamic process.

![actor ready](https://i.imgur.com/BLUFoFs.png)
In the spawned actor you could emit a reference to itself.

![listen actor](https://i.imgur.com/IP0XTtC.png)

and in the other actor you could bind to that event to do something with that information. In this case the event would be received whether it was pinned or not because the receiver is assumed to be spawned in before the emitting actor. But what if you couldn't control the delay?

![delayed bind](https://i.imgur.com/UfQYsJa.png)

This is the case where pinning the event would help as now when the receiving actor binds to the event, it will auto-matically receive the last emit even though it was called after the event was emitted. From a developer perspective you can now just handle the receiving logic and not worry about whether you need to add delays or loop through all actors in the map. By using signaling from various sources with logic filtering you can ensure that the order of your events remains predictable, only start x when part y and z in the map have happened.

### Flow muxing and Loose coupling

You can add a simple actor to the map which listens to various GES events. When for example two of those events have fired you can fire off another event which is a composite logic of the source events e.g. ANDGate or much more complex logic if we decide to use variable state.

![](https://i.imgur.com/ickckJe.png)

Blueprints which would listen to the SAReady event, don't even have to care where the source came from and you could easily swap out this logic actor for maybe another type without changing any other code; an example of the loose coupling enabled by GES. The actor is replaceable, there is no additional boilerplate that needs to be changed if replaced.

## When not to use GES
- There are some performance considerations to keep in mind. While the overall architecture is fairly optimized, it can be more expensive than a simple function call due to function and type checking. Consider it appropriate for signaling more than a hammer to use everywhere.

- If your objects have a tight coupling or it's easily accesible in a tree hierarchy pattern I would use standard methods instead of GES.

- Background threads. Current version is not thread safe.
