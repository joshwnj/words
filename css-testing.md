CSS testing
====

Thoughts and notes on the topic of automated CSS testing.  Goal is to bookmark a lot of available knowledge and discussions, summarize it and nut out some conclusions.

Consistency & Correctness
----

- we use tests for both these things, but the solve different problems
- consistency means we have confidence that what I see in one browser is the same in others. But it doesn't tell us whether it's correct or not.
- correctness is a measure of how close we are to spec.
- correctness is much more important but consistency is much easier to define & measure.

Kinds of tests
----

It doesn't seem like there is one kind of testing pattern that suits all stages of a project. So here we're trying to classify the different kinds of tests you might want to do and get an idea of how they can fit together.

### Detect changes

- something changed visual appearance since the last time we ran the test.
- this test involves a human component, somebody has to confirm whether the changes were intended or not.
- helpful for detecting accidental cascades.
- we can run this as part of the deploy / release process, so all visual changes are approved before being released.
 - eg. should be obvious from looking at the release notes, if there are changes on Page B but the release notes only talk about Page A.

### Detect cross-browser differences

- compare a master copy to a range of browsers and tell whether there are any significant differences.
- if we can make this cheap enough it could be run continuously during development to get instant feedback.

### Compare to spec

- this test is for proving the quality of our master copy, so only needs to be run in one browser.


Example Workflow
----

- get the spec passing in the master browser.
 - benefit of having non-screenshot tests here is that with screenshots you only get useful feedback when you're very near to complete and getting the final pixels in place.
- cross-browser comparisons can be run during this whole process so that inconsistencies are caught close to the source.
- once specs are passing and cross-browser consistency is sufficient, we can tag a release. At this point we run a "detect changes" test to make sure that all changes are intended and approved.


Bookmarks
----

_TODO: filter though these and add some comments to each one_

- [http://dalekjs.com/]()
- [https://github.com/Huddle/PhantomCSS]()
- [http://csste.st/]()
