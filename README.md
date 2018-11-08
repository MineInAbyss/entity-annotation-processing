# Event Annotations

Ever wanted to catch all bukkit events that extend a specific event?
Have you been dismayed to find that bukkit cries if you actually try to
do this with a normal EventHandler? Ever wanted to do all of that without adding
additional dependencies to your runtime?


Probably not. But if you have... introducing...

## ExpandEventHandler
`ExpandEventHandler` is a fancy dancy annotation you can throw on your should
be working with bukkit event handler instead of the standard `EventHandler` annotation.

Heres how it looks in the real world:
```
@ExpandEventHandler
void onPlayerEvent(PlayerEvent event) {
    //Amazingly sound logic and flawless code
    ...
    //and so on
}
```

Wow! Amazing. So cool!

But hey, some events are stupid right? Like `org.bukkit.event.player.PlayerInteractAtEntityEvent`

Oh dear. We don't want that one. lets get rid of it:
```
@ExpandEventHandler(exclude = {PlayerInteractAtEntityEvent.class})
void onPlayerEvent(PlayerEvent event) {
    //Amazingly sound logic and flawless code
    ...
    //and so on
}
```

Yay!

Some events are also trash even according to bukkit/spigot. Those darn deprecated events...
So lets go ahead and add them all to the exclude list one by one by one by...

Just kidding. Those already are exlcuded. Maybe we want them though:
```
@ExpandEventHandler(includeDeprecated = true)
void onPlayerEvent(PlayerEvent event) {
    //Amazingly sound logic and flawless code
    ...
    //and so on
}
```

And thats all!

## Actually Use the Damn Thing
Great. Now you know how to use it... but how do you like... actually use it???

Its pretty straightforward. Build the thing from source using these 600 dependencies:

*
*
*

Ok not really. Just download the provided jar or build the project using gradle and then compile
your plugin with the jar on the classpath.

Heres an example with gradle:

    compileOnly fileTree(dir: 'path/to/directory/containing/jar/', include: '*.jar')

(Note compileOnly - this tool is only used for compiling your plugin, and for nothing else)

## How it works
It just does... I think? I've tested it like twice.

But in all seriousness what we are actually doing is using an annotation processor that
processes the `ExpandEventHandler` mentioned before.

The processor takes the argument type from the annotated method, and uses [ClassGraph](https://github.com/classgraph/classgraph) to
find all of its concrete subclasses. (Thanks to Trigary for pointing me towards [the post he wrote](https://www.spigotmc.org/threads/listening-to-all-events-listing-all-events.337466/) explaining
how to do the ClassGraph magic)

Finally, we use Javacs non public barely documented oh god this sucked "Read Only" Abstract Syntax Tree API
to modify (heh read only) the AST before javac turns it into bytecode, injecting handlers for each event type
that delegate to your original method.

And then boom! Your class compiles and has many many methods you didn't have to write.

### Example Decompiled Bytecode Snippet
```
public void onPlayerEvent(PlayerEvent event) {
    //astounding plugin code
}

@EventHandler
public void onPlayerItemBreakEvent(PlayerItemBreakEvent event) {
    this.onPlayerEvent(event);
}

@EventHandler
public void onPlayerItemConsumeEvent(PlayerItemConsumeEvent event) {
    this.onPlayerEvent(event);
}

@EventHandler
public void onPlayerItemDamageEvent(PlayerItemDamageEvent event) {
    this.onPlayerEvent(event);
}

@EventHandler
public void onPlayerItemHeldEvent(PlayerItemHeldEvent event) {
    this.onPlayerEvent(event);
}

@EventHandler
public void onPlayerItemMendEvent(PlayerItemMendEvent event) {
    this.onPlayerEvent(event);
}

@EventHandler
public void onPlayerJoinEvent(PlayerJoinEvent event) {
    this.onPlayerEvent(event);
}

@EventHandler
public void onPlayerKickEvent(PlayerKickEvent event) {
    this.onPlayerEvent(event);
}

...
and so on
...
```