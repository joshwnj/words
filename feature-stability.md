2 stages of dev
====

When building a new feature we go through 2 distinct stages:

1. **pre-stable:** fast iterations, expectation that requirements will change.
1. **post-stable:** unless you're doing a new (unstable) version, the only dev will be:
 - fixing bugs that were missed earlier.
 - making very minor implementation changes (not requirements changes) to allow a good integration with other parts of the system.

There needs to be a mental shift that happens when we go from pre-stable to post-stable. It needs to happen across dev, product, management.

What happens when we don't make the shift?
----

- constantly building unstable features on top of other unstable features, so the potential for bugs is magnified.
- it's hard to make good architectural decisions because there's always uncertainty about which things are expected to change.
- by the time we've launched, everyone is too scared to touch it in case something breaks. So we get stuck with messy, unoptimised code.
 - this is compounded when we don't have enough automation built in.

Why doesn't it happen?
----

- fear of breaking something.
- management doesn't realise how essential it is (so we don't plan for the time required to do it).
- requirements aren't locked down tight enough, so there's a lingering feeling that something might change.

What are some things we can do to encourage it?
----

- start using the language of "unstable" and "stable".  Hopefully when management hears us talking about launching something that is "complete but unstable" they will ask the question "how do we stabilise"?
- stabilisation ceremony. Some event which marks the point in time for everyone involved, to help remember that it's passed over from one form to another.
- when we get a request to add / remove / change something in a stable feature, make it clear that this affects the stability and will need to be "restabilised" afterwards.
