# What is a package set?

[epistemic status: don't know if this is remotely correct, hoping to put it out there so others can correct me]

So there was a fun conversation on the #idris channel in the Functional Programming Slack about package sets today. I couldn't find a good description of them online, so I thought I'd try piece together that conversation and some random googling into something coherent.

An example of a package manager that uses a package set is purescript's [Spago](https://github.com/purescript/spago) and here's what they say:

> A package set is a collection of packages, such that there is only one entry (i.e. version) for a given package in the set.
> 
> If you use a package manager based on package-sets, this means that when you want to install a package:
>
> - it must be in the package set
> - its dependencies and all the transitive dependencies must be in the package set

See more [here](https://github.com/purescript/package-sets#what-is-a-package-set), and their [CONTRIBUTING](https://github.com/purescript/package-sets/blob/master/CONTRIBUTING.md) guide also has some info.

To give an example of "a collection of packages [with] only one entry": say there's packages AA at `v3.0`, BB at sha `12ab77c`, and CC at `lte`, where CC depends on BB. This would be version `1` of the package set. Should AA upgrade to `v4.0`, then that would create a version `2` of the package set. As JSON it might look like the following:

```
{ package-set-version : 1, packages : [ { name : "AA", version: "v3.0" }, { name : "BB", version : "12ab77c" }, { name : "CC," version : "lte" }] }

upgrades to -> 

{ package-set-version : 2, packages : [ { name : "AA", version: "v4.0" }, { name : "BB", version : "12ab77c" }, { name : "CC," version : "lte" }] }
```
You then take either version `1` or `2` of the package set, and build your project against those package versions.

So I guess the package set community works to keep their packages in line with the rest of the set.

## Life in package set land

I still have a lot of open questions about package sets in practice, I'll just lay them out here:

### As a developer, how do I use the package set?

Probably the package set has some tooling around it, but I think essentially you pick a version of the package set and if you restrict yourself the those packages at those versions all should "just work".

### As the maintainer of a package in the package set, how do I get in the package set?

If the package set is basically a list of packages at their versions, if you build you project against that and it succeeds, it should be possible to release a new version of the package set which just appends your package.

### As the maintainer of a package in the package set, how do update my package?

As long as changes are compatible with the rest of the set you should be ok to change and update the package set. If you want to make an incompatible change, guess you would have to append the new functionality, deprecate the interface you want to break, upgrade the things that depend on you, then remove the old functionality.

### As the maintainer of a package in the package set, how do I use a newer version of a dependency?

Probably the same steps as updating your package, if it's backwards compatible all is good. If it's not backwards compatible...?

### How to confirm that a package set is correct?

Don't know, compile all projects in the set one by one and see if it succeeds?

### What happens with multiple projects on same machine that depend on different versions of a package?

I assume there's tooling that knows project X depends on version `1` of the package set, and project y depends on version `2` of the package set, it keeps the package sets separate from one another.

### and multiple projects on same machine that depend on different versions of the _language_?

Not sure. The python 2 to 3 ecosystem upgrade was quite a dance, not sure what the equivalent is for package sets.

### What if you need a different version than in the package set?

There is presumable some way to override the package set versions on a per project basis. If you depend on a project CC that depends on a project BB, but you depend on a newer version of BB, then I'm not sure what happens. You can always wait till CC updates to use the newer BB, and then grab both newer versions from a later package set, but not sure what happens till then. That's probably a problem with many package managers though.

### Are package sets and package managers different?

Not sure, maybe it comes down to version resolution. You can make a package manage that's powered by a package set, so if you depend on AA and BB, it'll look at the version of the package set and then grab the appropriate versions of AA and BB.

Other package managers might depend on semantic versions, so if you depend on `AA >= 3.0` and `BB == 1.0` it'll solve some contraints and decide on the appropriate versions, then grab those.
