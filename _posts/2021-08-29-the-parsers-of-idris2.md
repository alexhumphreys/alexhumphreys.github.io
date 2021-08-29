[epistemic status: Not very confident on a lot of this. High level should be ok, but lot of details could be wrong]

So the idris2 repo has a lot of parsers, and I spent the last few weeks being confused by which is which. After a lot of grepping, I think this is where they all are and their general purpose. This is mostly just notes from what I found when rewriting the [idrall](https://github.com/alexhumphreys/idrall) parser, but maybe it's useful to others.

The date of writing this is August 29, 2021, and I'm basing this on [idris2 commit `1e00277`](https://github.com/idris-lang/Idris2/tree/1e0027721f11fb51f7bcea86f39e5c27133a72bd)

Using the bash command [`fd parser`](https://github.com/sharkdp/fd) to get the parsers, let's go through them one by one:

## 1. `src/Libraries/Text/Parser`

A total parser combinator library used by idris2 itself internally. This parser has a lexer, and good support for positioning.

Note: the following I'm unsure about:

I think it also has 2 tokenizers. There's `src/Libraries/Text/Lexer/Tokenizer.idr` and `src/Libraries/Text/Lexer/Core.idr`. The difference is `Tokenizer.idr` can support nested grammars, as is required by Idris' grammer for string interpolation, and the `Core.idr` one does not.

Check out here to see how parsing idris2 source [uses the `Tokenizer.idr` type `Tokenizer`](https://github.com/idris-lang/Idris2/blob/1e0027721f11fb51f7bcea86f39e5c27133a72bd/src/Parser/Lexer/Source.idr#L316). Compare to how parsing the .ipkg files (a much more simple grammer) uses the [`TokenMap` type from `Core.idr`](https://github.com/idris-lang/Idris2/blob/1e0027721f11fb51f7bcea86f39e5c27133a72bd/src/Parser/Lexer/Package.idr#L100).

Both of these result in token that can then be consumed by the parser `src/Libraries/Text/Parser`.

## 2. `libs/contrib/Text/Parser`

Copy of `src/Libraries/Text/Parser`, made available to other packages via contrib. This is duplicated because idris2 does not want to have a dependency on contrib. It can also get out of sync with `src/Libraries/Text/Parser`

## 3. `libs/contrib/Data/String/Parser`

Simple parser combinator library, doesn't have a lexer. To see this in use check [its tests here](https://github.com/idris-lang/Idris2/blob/1e0027721f11fb51f7bcea86f39e5c27133a72bd/tests/chez/chez027/StringParser.idr)

## 4. `src/Parser`

Uses `src/Libraries/Text/Parser` module to write a collection of smaller parsers to be used by idris2 when parsing idris2 source code. Running `grep -r Rule src/Parser` shows that most of the parsers here return basic Types rather than PTerm:

```
src/Parser/Rule/Source.idr:constant : Rule Constant
src/Parser/Rule/Source.idr:intLit : Rule Integer
src/Parser/Rule/Source.idr:onOffLit : Rule Bool
src/Parser/Rule/Source.idr:strLit : Rule String
src/Parser/Rule/Source.idr:symbol : String -> Rule ()
src/Parser/Rule/Source.idr:keyword : String -> Rule ()
...
```

## 5. `src/Idris/Parser.idr`

Uses the parses in `src/Parser` to parse to something like an idris2 AST? Also not sure on this, but these parsers match higher level idris2 constructs:

```
$ grep -r Rule src/Parser
src/Idris/Parser.idr:recordConstructor : OriginDesc -> Rule Name
src/Idris/Parser.idr:ifaceDecl : OriginDesc -> IndentInfo -> Rule PDecl
src/Idris/Parser.idr:implDecl : OriginDesc -> IndentInfo -> Rule PDecl
src/Idris/Parser.idr:fieldDecl : OriginDesc -> IndentInfo -> Rule (List PField)
src/Idris/Parser.idr:typedArg : OriginDesc -> IndentInfo -> Rule (List (Name, RigCount, PiInfo PTerm, PTerm))
src/Idris/Parser.idr:recordParam : OriginDesc -> IndentInfo -> Rule (List (Name, RigCount, PiInfo PTerm,  PTerm))
src/Idris/Parser.idr:recordDecl : OriginDesc -> IndentInfo -> Rule PDecl
src/Idris/Parser.idr:directiveDecl : OriginDesc -> IndentInfo -> Rule PDecl
src/Idris/Parser.idr:import_ : OriginDesc -> IndentInfo -> Rule Import
...
```

## 6. `src/Idris/Parser/`

Surprise! This module only contains one file `src/Idris/Parser/Let.idr`, which just parses `let` declarations.

## 7. `src/Idris/IDEMode/Parser.idr`

I think this is used to parse the protocol that the IDE mode uses idris2 uses to communicate with editors.

## 8. `src/TTImp/Parser.idr`

Not sure...

## 9. `libs/contrib/Language/JSON/Parser.idr`

A JSON parser. Uses `libs/contrib/Text/Parser` if you need a more simple example of the `src/Libraries/Text/Parser` module in action.

## Conclusion

That's what I found, hope it's usefult to someone! If you find any mistakes or important omissions, feel free to open an issue or PR at [this blog's repo](https://github.com/alexhumphreys/alexhumphreys.github.io).
