# I hate modern CI

Going old school internet with a blog rant. For a better description see [Gregory Szorc's post](https://gregoryszorc.com/blog/2021/04/07/modern-ci-is-too-complex-and-misdirected/).

So modern CI I'd describe as something where you check a yaml file into your repo, and when you push it some service reads that file and runs some tasks. Gitlab's gonna take most of the heat, but only because that's what I use every day, the same is probably true for Github/Travis/etc...

My main complaint is that these CI platforms are optimised for simple workflows, and anything beyond that is basically spaghetti code made worse by the fact that it's in YAML and untestable.

Let's talk through how this happens:

You start a new project, you're in a hurry to get something running, so you reach for a CI that can do something like this:

```
compile -> test -> push to docker ☺️
```

The details/language/etc aren't important, maybe you make a deb package instead of a docker container, whatever. Modern ci does this very well, hence the smiling emoji.

So the project matures a bit, you want to deploy it somewhere, and add a post deploy smoke test:

```
compile -> test -> push to docker -> deploy -> smoke test ☺️
```

Still looking good. Lets add another env:

```
compile -> test -> push to docker
-> deploy staging -> smoke test
-> deploy prod -> smoke test 😐
```

Smile's beginning to crack, that deploying/testing those two environments is very similar, but you basically have yaml anchors (which suck) or janky homegrown features (like [`extends`](https://docs.gitlab.com/ee/ci/yaml/#extends), get ready for how that doesn't play well with lists) to represent this. If this was a programming language, you'd have maybe an `environments` list that has `["staging", "prod"]` and a steps list `["deploy", "smoke-test"]`, and you could traverse both on your way to living good a life. But we're in yaml town, so we gotta use janky tools or hope the CI people implement some abstractions.

Let's add automated rollback:

```
compile -> test -> push to docker
-> deploy staging -> smoke test -> rollback on failure
-> deploy prod -> smoke test -> rollback on failure 🤨
```

Surprise on gitlab, when any step fails, _all_ `on_failure` tasks run. So if your staging smoke test fails, get ready to rollback both staging and production! Programming languages let you handle errors with monads or exceptions or checking return codes, here our options are much more limited. On gitlab we can add [`needs`](https://docs.gitlab.com/ee/ci/yaml/#needs) to make sure that only the correct failure runs, but our config is getting pretty complicated...

Let's add some cleanup task that always runs:

```
compile -> test -> push to docker
-> deploy staging -> smoke test -> rollback on failure -> cleanup
-> deploy prod -> smoke test -> rollback on failure -> cleanup 😔
```

Surprise on gitlab! If say deploy staging fails, your cleanup job runs, good times. Let's say you go to the UI and hit the retry button on deploy staging. It passes, the pipeline continues, but the cleanup job doesn't run again because _it has already run_. So guess you can't use that retry button anymore.

All these problems get worse as you add environments, and since this file is checked into the repo, it won't help you if you have 15 services that have _mostly_ the same CI but with minor differences ([`include`](https://docs.gitlab.com/ee/ci/yaml/#include) is pretty limited if you want to make changes). I haven't even gotten to the [`rules`](https://docs.gitlab.com/ee/ci/yaml/#rules) keyword, which is wildly complicated, but also a list so you can say goodbye to [`extends`](https://docs.gitlab.com/ee/ci/yaml/#extends) helping you abstract it. Or that projects only have one pipeline, so if you need to trigger it remotely expect _all_ the above to happen, no easy way to express "in case of X redeploy prod".

So modern CI may start simple but very quickly you run into stuff like:

- `rules` is incompatible with `when`, and plays badly with `extends`
- any kind of specific failure is real tough to handle.
- _when_ steps run and _what commands are run_ are conflated: got one service that deploys to K8S and another to EC2? Get ready to recreate all that crazy pipeline failure logic.
- `include` dumps all jobs into your pipeline, if you only have 2 envs, but someone else has 3, good luck sharing stuff.
- left it out above for brevity, but writing a shell script in a YAML file is a thankless task.

I'm tired. I'm tired of pretending this isn't programming. There's actions, happening in an order, governed by rules as to when they happened, recovering from failure... We have a way to tell computers how to do this, and it's programming languages. Then maybe we'd realise we need some proper tooling: an ability to run locally (shout out to [CircleCI which has this](https://circleci.com/docs/2.0/local-cli-getting-started/#testing-a-job-before-pushing-to-a-vcs)), see which jobs would get run when, see if failures are handled at the correct time, proper abstractions for repetition...

This is normally when I'd recommend [Dhall](https://dhall-lang.org/), which would help with the [inherent yaml issues](https://noyaml.com/), but it would be tough to express this logic in dhall, eg: it has `if` statements that will generate the correct config, but this logic would need an if statement that is around at runtime so the pipeline behaves correctly.

If I had infinite time/ability to solve this what would I do? I'd probably start breaking stuff up. Right now you write YAML, hand it to the CI gods and something may or may not run somewhere. As [Gregory Szorc](https://gregoryszorc.com/blog/2021/04/07/modern-ci-is-too-complex-and-misdirected/) pointed out, CI systems are basically remote code execution platforms. So I'd offer _that_ as an API.

Once that's separate you can now have different ways of interacting with that API:

- classic thing that watches repo for yaml file, and when it finds it calls the API.
- make a domain specific programming language (DSL) for expressing CI jobs, write an interpreter for that which then uses the API.
- write a command line tool that reads that DSL and can run it locally, or at least let you step through it debugger style.
- as Gregory said, write a Gradle/SBT/Bazel/whatever plugin that calls the API.
- who knows what else.

If you have a good API, people will find interesting ways to use it. It sounds like [Taskcluster](https://docs.taskcluster.net/docs) from Mozilla may be that, and I intend on looking into it, and the ability to write github actions that can be shared is a good step in this direction.

So I'm hopefully some people will do something interesting in this space, but if we keep being satisfied with YAML+janky features that don't play well together, CI life won't really get better.
