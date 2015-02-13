For Directories and Glory
====

Greetings adventurer! In today's expedition we're going to pick a popular nodejs library and dive into its depths to see what we can find.

If this all goes to plan we'll come away not only understanding the internal workings of a commonly-used module, we'll also discover some interesting patterns and techniques along the way.

Starting out
----

Get ready to dig into [substack's](https://github.com/substack) excellent [mkdirp](http://npmjs.org/mkdirp), a handy library for recursively creating directories (just like `mkdir -p` in linux). If you're like me, you've probably used this library many more times than you actually realize - it is a dependency of about 1000 other modules in npm!  So understanding modules like this will naturally lead us on to enlightenment elsewhere.

If we look in [index.js](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js) we're dealing with just under 100 lines. That feels pretty achievable. Let's go.


Aliasing the export
----

The first thing that stands out here is on [line 4](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L4): `module.exports = mkdirP.mkdirp = mkdirP.mkdirP = mkdirP;`

At first glance it looks a bit odd, but it's quite easy to understand if we read it from right to left:

- <code>module.exports = mkdirP.mkdirp = mkdirP.mkdirP = </code><span style="background:yellow; padding: 0 5px;"><b><code>mkdirP</code></b></span> is the main function that we're exporting. This is defined on [line 6](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L6).
- <code>module.exports = mkdirP.mkdirp = <span style="background:yellow; font-weight: bold; padding: 0 5px;"><b>mkdirP.mkdirP = mkdirP</b></span></code>: what we're doing here is adding a new property to the `mkdirP` function (remember that in javascript functions are objects). This new property is an alias back to the original function. Why would we do this? I can't speak for the author's intention here, but it could be to allow different forms of `require`.  For example, the alias means we can get the same function in 2 different ways: `require('path/to/module')` or `require('path/to/module').mkdirP`. Maybe someone would prefer the second form for being more explicit.
- finally there is one more alias: `mkdirP.mkdirp = mkdirP.mkdirP = mkdirP`. This alias gives us the same function but with an all-lowercase name.

So, first hill climbed. Onward to the function itself.


Function arguments and alternate signatures
----

The `mkdirP` function starts out with a technique that, while widely used, can be confusing at first glance:

_[(Line 7)](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L7)_

```js
if (typeof opts === 'function') {
    f = opts;
    opts = {};
}
```

You might read this literally as _"if the options object is actually a function, set function `f` to be the options object and then erase the options object."_ And reading it that way you would be rightly confused.

But once you've seen it a few times you'll be able to see between the lines and recognise it as a technique for optional function arguments. You could picture it as 2 signatures for the same function:

- `function mkdirP (p, opts, f)`
- `function mkdirP (p, f)`

With that in mind we can ditch the confusing literal reading from before, and learn to read this as _"if the second argument is a function, the caller must be using the `mkdirP (p, f)` signature. Treat the second argument as `f` and set `opts` to its default value (an empty object)"_.

Further on you'll see more code dealing with alternate signatures, like in [lines 11 to 13](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L11-L13).


Dependency injection
----

The next interesting thing we'll talk about is on [line 16](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L16): `var xfs = opts.fs || fs;`

This allows us to substitute a custom `fs` object in place of the [core module](http://nodejs.org/api/fs.html). As long as we provide an object with equivalent functions for `mkdir`, `mkdirSync`, `stat` and `statSync` it will go ahead as normal. The most obvious use for this would be for testing (so that a real filesystem is not needed) and we can see [mock-fs](https://www.npmjs.com/package/mock-fs) being used in some of the test-cases. But there's no reason why you couldn't also use this for some kind of virtual filesystem.


Binary operations
----

On [line 19](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L19) we see one of the less-common parts of the javascript language: binary operations.

```js
mode = 0777 & (~process.umask());
```

You may well write javascript for years and never need to use syntax like this. But it's good to be aware of it for the sake of understanding what's going on. To break it down:

- we start with a number representing "full access" (`0777`. The `0` at the start tells javascript this is an octal number)
- `process.umask()` gives us another number which relates to the filesystem permissions of the user (whichever user executed the `node` process).
- `~` is a binary operation which inverts the value.
- `&` is an operator that combines the two numbers in a certain way (binary `AND`).

So putting that all together: we get the user's permission mask, invert it, and apply the mask to a full set of permissions. The value we end up with after applying the mask is what will be set on any files or directories we create.

_If you want to read more, Mozilla MDN has a good reference of [bitwise operators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_AND). And there's also some good reading on wikipedia about [file system permissions](http://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation)._


Making the directory
----

Finally we get to the part of the function that actually makes the directory:

_[(Line 26)](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L26)_

```js
xfs.mkdir(p, mode, function (er) {
    if (!er) {
        made = made || p;
        return cb(null, made);
    }
```

This reads as _"Create a directory, and if there's no error, we can return here (via callback)"_.  The second argument to the callback, `made`, will be set to the path of the first directory to be successfully created (`p`).

[Line 31](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L31) continues with error handling: `switch (er.code) {`

Note that in this module we actually expect errors to occur. The `ENOENT` error means that `mkdir` failed to create the directory because a parent directory didn't exist. We handle that by recursively calling `mkdirP` again, but this time attempting to create the parent directory (`path.dirname(p)`):

_[(Line 32)](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L32)_

```js
case 'ENOENT':
    mkdirP(path.dirname(p), opts, function (er, made) {
        if (er) cb(er, made);
        else mkdirP(p, opts, cb, made);
    });
    break;
```

This recursion will continue stepping up to the parent until it successfully creates a directory, at which time it steps back down to create the child directory we originally asked for.


Homework
----

We only got halfway through the module, but the rest of the code mostly deals with a [synchronous version of the same thing](https://github.com/substack/node-mkdirp/blob/0.5.0/index.js#L54-L97) (using `fs.mkdirSync` instead of `fs.mkdir`).  Take a look and see if you can recognise the same patterns that we saw earlier!

Another great way to gain more insight into a 3rd-party module is to read some of the history.  Pick a spike on the [contributors graph](https://github.com/substack/node-mkdirp/graphs/contributors) and see what you can learn from commits made in that time.  You can also learn more about the design goals and particular challenges the authors faced by reading over past and present [issues](https://github.com/substack/node-mkdirp/issues) and [pull requests](https://github.com/substack/node-mkdirp/pulls).  And of course, if you get stumped on the meaning of a certain part of the code you can always ask :)

Wrapping up
----

Thus concludes our adventure into the land of `mkdirp`!  I wonder how many times I've depended on this code without having understood in depth how it works??  My hope is that by now, you also can see that this need no longer be the case. Nearly everything in software development is a matter of making a tradeoff. So the more we understand the libraries we use, the more informed our decisions will be when it comes time to trade.
