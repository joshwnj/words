Anti-pattern: URLs as an indirect lookup
====

Consider this ridiculous scenario:

- imagine you had an employee called Arnold.
- Arnold leaves and you have to hire someone to fill his job.
- you hire Betty but insist everyone call her Arnold. That way all the staff won't have to learn a new name, and we won't have to print a new batch of business cards.
- hmm now that just created more confusion because when people talk about "Arnold" they don't know whether to say "he" or "she"...
- i know, let's wipe everyone's memory about Arnold and then we can just tell them Arnold is a girl's name. Problem solved.

Now, that would never happen.  But we do something similarly nonsensical all the time:

- you have an image for your company's logo
- that image has an URL, and you code it into all your html pages in an `<img>` tag
- company logo changes!

*What do you do?*

- obvious/quickest solution is to replace the old image file with the new one.  Hey that worked great! I didn't have to change any of my html code, because the URL is the same, just the contents have changed.
- but actually, this sucks. For the same reason as telling someone they need to change their name to "avoid confusion" and "keep it simple".  Just creates more confusion with memory (cache).

*Ok, what's a better solution?*

- let's use a new URL. That way there will be *no confusion* because when I refer to a URL it can mean only 1 thing.

*Hmm but now i have to edit all my htmls ;_;*

- yes you could do that, **or** take the opportunity to refactor
- instead of referring to the URL directly in your html, use a lookup
- notice that what we're doing here is moving the indirect lookup down a layer, from the URL into the app. That's where it belongs!
- bonus round: build step.

Recap
----

The internet might sometimes feel like a gigantic key-value store, but when we blindly use it that way we run into unnecessary problems at the user level which ought to have been dealt with at the app level.

URLs are special creatures. They don't only give us an address to find an object, they are also strongly tied to the identity of the object. Ambiguity of meaning is great for poetry, disasterous for computers. Make sure an URL means 1-and-only-1 thing, both in the present and future.
