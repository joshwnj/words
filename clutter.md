Clutter
====

- clutter in software is all of those things you thought you were going to need, or things that you *did* need at some point but not anymore.  Now it's a loose cable on the floor waiting for someone to trip on.

- turns out removing stuff is way harder than adding stuff. Particularly if it's been there for a while, it gets really hard to see what other parts of the system might be depending on it.  So you leave it for fear of breaking something, or because _"ya never know"_ you might need that thing again at some point...

- **golden rule**: unless you're certain this is going to be a core feature, don't write a single line of code until you've got a solid plan for how you're going to remove it.

- so how can you make it easy to remove things?
 - control dependencies
 - dependencies must travel in only 1 direction, from feature to core.  Feature depends on core, core must never depend on the feature.  When you have core depending on that feature, your feature can sneakily influence all other features even if they are following the rules (dependency infection!). So if you removed your feature now it could bring everything crashing down.
 - if you can identify that a feature is depending on things but not affecting them (1-way dependency), and nobody else is expecting this feature to exist, then you can remove it.  One less bug-nest!
 - a plugin/hook architecture is not essential to do this but can give a lot of good options.
 
