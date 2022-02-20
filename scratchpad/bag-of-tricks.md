# SRE Bag of tricks

https://silo.pub/pound-for-pound-a-biography-of-sugar-ray-robinson.html

> “You don’t look like you used to, Edna Mae told me,” Sugar recalled. “You look like you’re trying to knock out everybody. You’re so anxious to fight again, you just want to show everybody how great you once were by knocking out all of your opponents. But that’s not how you were great. You’re not using the bag of tricks that made you great. That was your gift, the tricks were what God blessed you with—the tricks, the science. You don’t use that anymore. You don’t look like the Ray Robinson anymore.”

## Philosophical

- systems have emergent behaviour
- bayes
- model building
- falsification
- chesterton's fence: https://wiki.lesswrong.com/wiki/Chesterton%27s_Fence

## Some tricks

- Always suspect versions. You code, your environment, your dependencies, check for changes.
- fast feedback loop, get whatever setup you need to test theories quickly
- make it do anything. if something is broken (not responding to network traffic, restarting, timeout, no output, whatever), try to make it somehow respond to anything. bluetooth radio story.
- start inside, more out, network testing. esp in k8s. hit it on localhost, then other container in same pod, then different pod, then ingress/elb, then within VPC then outside VPC. (try both with IP address or DNS names)
- hack in a fix to buy time. eg, story: change a weird CNAME to an A record with hardcoded IPs.
- actually read error messages. actually google error messages.
- logs fluffier: search slack/github (especially the org) for error messages
- logs fluffier again: just kind of free search your brain for the last mention you came across for anything associated with any of the concepts in the error message
- all the stuff here: https://wizardzines.com/networking-tools-poster/
- retry stuff (with some caveats eg. 409/database migrations)
- increase the timespan on monitoring graphs (see if there is anything that breaks from any pattern)
- if there was some error that went unnoticed for a few days, go back to slack for that day and see what people were up to.
- order of suspicion is least shared to most shared. thing you just changed -> your app -> your configuration -> network -> small time dependencies (other apps in your company) -> big time dependencies (AWS/k8s/rails).
- get a second opinion on metrics. does the graph of CPU usage in Grafana match what htop is saying?
- lots of sources of info on changes: `git blame`, commit messges, PR descriptions and comments, slack, Linear tickets, check them all for hints.
