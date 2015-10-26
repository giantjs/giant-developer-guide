<!-- @@@page:manual@@@ -->
<!-- @@@title:Best Practices@@@ -->

Best Practices
==============

## No instantiation in `.postponed` & `.amendPostponed`

Make sure not to instantiate any classes inside a `.postponed` or `.amendPostponed` block that *constructs* a class. You may rely on language features, built-in types, and the class-building tools.

Instantiation within a `.postponed` block might lead to an infinite loop:

- Class "A" instantiates class "B" in order to build itself
- Class "B" has an amendment somewhere else that invokes class "A"

**Result**: Class "A" will never resolve, and you get a stack overflow error.

Instantiation within an `.amendPostponed` block might lead to unexpected behavior in surrogates:

- Class "A" is memoized (eg. singleton)
- Class "B" extends "A" with a surrogate amendment (pointing from "A" to "B")
- Class "A" has an amendment elswhere, in which it's instantiated

**Result**: At the first user-initiated instantiation of "A" the surrogate might be applied last among the other amendments, in one of which instantiation will succeed & yield an instance of "A". Since "A" is memoized, from this on, it will always return the same instance for the same constructor arguments, regardless of the surrogate.

**It's important that**

- Instantiation is always wrapped in an (anonymous) function
- Only built-in types are used as static properties for classes
