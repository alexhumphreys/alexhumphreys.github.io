# What is a package set?

Big thanks to [shmish111](https://github.com/shmish111) for basically writing most of this.

Package sets came up on the #idris channel in the Functional Programming Slack today, and I wasn't sure what they were. So I thought I'd try answer that here.

Package sets offer an alternative way of finding dependency package versions, as opposed to say [Semantic Versioning](https://semver.org/) based approaches. Let's look at an example.

A package manager that uses a package set is purescript's [Spago](https://github.com/purescript/spago) and here's what they say:

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

Generally the package set community works to keep their packages in line with the rest of the set.

## Life in package set land

Here's some questions I had about package sets, which [shmish111](https://github.com/shmish111) helpfully answered:

### As a developer, how do I use the package set?

Most (all) package sets are associated with a package manager, Haskell has stack and stackage, purescript has spago and [package sets](https://github.com/purescript/package-sets), nix has nix and nixpkgs etc. Note that the package set can be used separately, this is often the case in haskell where people use stackage with cabal and nix, not just stack.

### Are package sets and package managers different?

Yes but they are almost always presented together. Again, with Haskell it’s presented separately though, there has been a great effort to make sure people understand that you can use the package set (stackage) without the package manager (stack). This is because there was already a de facto package manager around

With a package manager that's powered by a package set, if you depend on AA and BB, it'll look at the version of the package set and then grab the versions of AA and BB listed there.

Package managers not based on a package set might use say [Semantic Versioning](https://semver.org/). In that case, you say that you depend on `AA >= 3.0` and `BB == 1.0`, the package manager will try solve those contraints and decide on the appropriate versions AA and BB, then grab those.

### As the maintainer of a package in the package set, how do I get in the package set?

Generally you just fork the package set, add your package and create a pull request with your update.

### As the maintainer of a package in the package set, how do update my package?

The same process as above however much of this can be automated away. For example, you could provide the package set with a git branch `stable` that is always stable, the automated tooling behind the package set can then try to update the package set itself whenever it sees a change to `stable`. In the event of a failure it can contact the package maintainer explaining.

### As the maintainer of a package in the package set, how do I use a newer version of a dependency?

You use the package set version which you intend to be merged into. If some dependency isn’t in that package set then you need to collaborate with the maintainer of that dependency to get it into the package set first.

### How to confirm that a package set is correct?

In the case of [`smoke-hill`](https://github.com/shmish111/smoke-hill), it just builds all packages, so this is as good as the idris type checker. I think stackage also runs the tests of each package, this would be ideal. This is all automated in a CI system where the package set is published, for example https://travis-ci.com/github/shmish111/smoke-hill

Note that this may seem like a big job but with suitable caching it should be quite easy, most updates would be very small builds.

### What happens with multiple projects on same machine that depend on different versions of a package?

You generally include a file in your project that says which version of a package set you want to use.

### and multiple projects on same machine that depend on different versions of the _language_?

Most package sets include the language, specifying the package set version specifies the language version. Package managers take care of this

### What if you need a different version than in the package set?

There is usually some way to override the package set versions on a per project basis. If you depend on a project CC that depends on a project BB, but you depend on a newer version of BB, then I'm not sure what happens. You can always wait till CC updates to use the newer BB, and then grab both newer versions from a later package set, but not sure what happens till then. That's probably a problem with many package managers though.

In a more general sense, you can usually tell the package manager where the package set definition is located, it will likely default to somewhere on the internet but you could specify a local definition.

## Conclusion

So that's a brief introduction to package sets. If I think of more questions I'll update above. Got any package set questions, or maybe I'm missing some important details? Feel free to open an [issue](https://github.com/alexhumphreys/alexhumphreys.github.io/issues) or [pull request](https://github.com/alexhumphreys/alexhumphreys.github.io/pulls).

## Links

- [Stackage](https://www.stackage.org/), [Stack](https://docs.haskellstack.org/en/stable/README/)
- [nix-pkgs](https://github.com/NixOS/nixpkgs), [nix](https://nixos.org/)
- [purescript package-sets](https://github.com/purescript/package-sets), [spago](https://github.com/purescript/spago)
- [smoke-hill](https://github.com/shmish111/smoke-hill) 
