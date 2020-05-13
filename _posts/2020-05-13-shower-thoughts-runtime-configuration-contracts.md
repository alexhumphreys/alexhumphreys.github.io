# Shower thoughts: runtime configuration contracts

So I've been thinking about the problem of configuring microservices. When you have a application that is split it into microservices, each of the services becomes simpler, but the configuration and interactions between them become more complicated.￼￼ often services have configurations dependencies on other running services. ￼I'm wondering if Dhall's combination of `assert` and HTTP imports￼ can be of use￼ here.

Let's take for an example, you maintain two services: service A and service B. Service B can be run in two, authentication on and off.￼￼ Service A makes API requests against service B, So depending on the runtime configuration of B, service A will maybe need to be configured to ￼add authentication￼ headers when making requests.

So what can happen is service B has authentication￼ turned on￼ at runtime, service A is deployed without authentication turned on, and so there is an error the first time it makes a request to service B.￼ 

Having to wait until the first request to find out that this error has occurred is lame. Is there a better way to do this?

￼What I've been thinking is taking a contract testing approach to configuration.￼ So the maintainer of service A can add an endpoint like /config.dhall￼ to service B. That endpoint could read it's own config values and return something like eg `{ authEnabled : True }`

Now when service A starts, it can read its own configuration and create a check for what it depends on, like:

`let B = http://serviceB.example.com/config.dhall

let test = assert : B.authEnabled === True

in test`

Now Service A at start-up can check whether its dependencies are configured the way it needs. Plus an added bonus is it checks that it has the correct URL for service B and that it can reacher over the network.￼￼

If there is an error, ￼I guess the maintainers of service a can then choose to feel to boot, or a log a message with what's wrong. This error would hopefully be more informative/timely ￼then whatever ￼ error would occur later on when service A attempts to make a request to service B without authentication.￼
￼
This is a kind of simple example, but I've been configuring microservices enough to have run into similar situations and it's more often then I'd like￼. ￼

This example as soon as you maintain both services, but I can imagine even for some external dependencies like S3 bucket, you could just write a Dhall file to that S3 bucket with the buckets relevant information,￼ and then request that file from your service￼.

Of course this doesn't have to use Dhall, but it's a simple enough language and expressive for these kind of cases. So if your language has binding you'd be able to play around with this idea straight away.￼￼

Not sure if this is useful, there's probably a bunch of issues i haven't thought of, but it seems like an interesting idea, would like to see where it goes.
