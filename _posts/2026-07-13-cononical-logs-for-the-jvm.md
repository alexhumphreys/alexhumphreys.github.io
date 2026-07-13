# A Canonical Logs Library for the JVM

I've been getting really into canonical-logs (aka wide events) for observability. I'm not a JVM pro, so I got Claude to whip up a library to help, you can find it at https://github.com/alexhumphreys/canonical-log

The basic idea is making "unit of work in progress" a small core, and it accumulates counter increments, new fields with values/durations etc, and plays nice with virtual threads and kotlin coroutines. Then, test the hell out of this for concurrency issues.

With that in place, can write packages that use that just hook into various tools, ideally transparently, and instrument. These can be kept simple as the core takes care of the spicy details.

Right now there's support for spring and dropwizard, jdbc, kafka, jobrunr, and a handful of other tools. So play around with it, check out the code and the tests, see if it's interesting!
