# Debugging Idris Code

_[confidence status: there must be a better way than this]_

So I've been playing about with Idris, and this language is *chef's kiss

However right now (Feb 2020) it's primarily a research language, so I haven't found anything by way of a debugger.

Maybe such a thing exists? (If so, please tell me) Either way I found myself with [code](https://github.com/alexhumphreys/pl-playground/commit/71ebebc7e7e683a5b66a46da8b1bdafbda0693f7#diff-4c8e73d3ab3b8b5da109b4869ece941eR128) to parse expressions generically. Basically `buildExpressionParser` from Haskell ported to Idris with the [Lightyear](https://github.com/ziman/lightyear) parser library. What I had compiled but didn't execute correctly. It sounds silly but this was a new experience for me in Idris! So far with Idris I've mostly just been following along examples and easy stuff, and normally things that compile they "just work" because the compiler is _really_ something.

So I had to debug code. I was neck deep in parser combinators and I figured it was some left-recursion, but I needed to say for sure. My plan of just galaxy brain staring at it for hours had failed spectacularly. So I figured I'd try hand the debugging over to something else: javascript.

Idris has a javascript backend, so I could compile the code with `idris -p lightyear --codegen javascript Expr.idr -o expr.js`, then goofily stick it in a html file like so:

```
<html>
  <head>
    <script type = "text/javascript" src = "./expr.js"></script>
  </head>
</html>
```

Then I could open that in a browser, crack open the inspector, and see what's going on!

... well, kind of. There's no source map so this:

```
  table : OperatorTable Integer
  table =
    [ [Infix (do token "*"; pure (*) ) AssocLeft]
    , [Infix (do token "+"; pure (+) ) AssocLeft]]

  term : Parser Integer
  term = parens expr <|> intConst
         <?> "simple expression"
```

becomes this:
```
// Main.table

function Main__table(){
    return new $HC_2_1$Prelude__List___58__58_(new $HC_2_1$Prelude__List___58__58_(new $HC_2_0$Main__Infix($partial_7_12$Prelude__Monad__Lightyear__Core___64_Prelude__Monad__Monad_36_ParserT_32_str_32_m_58__33__62__62__61__58_0(null, null, null, null, null, Lightyear__Strings__token(null, null, "+"), $partial_0_6$Main___123_table_95_83_125_()), $HC_0_1$Main__AssocLeft), $HC_0_0$Prelude__List__Nil), $HC_0_0$Prelude__List__Nil);
}

// Main.term

function Main__term(){
    return $partial_6_8$Lightyear__Core___60__63__62_(null, null, null, null, $partial_6_12$Prelude__Applicative__Lightyear__Core___64_Prelude__Applicative__Alternative_36_ParserT_32_str_32_m_58__33__60__124__62__58_0(null, null, null, null, Lightyear__Combinators__between(null, null, null, null, null, null, Lightyear__Strings__token(null, null, "("), Lightyear__Strings__token(null, null, ")"), Prelude__Foldable__Prelude__List___64_Prelude__Foldable__Foldable_36_List_58__33_foldl_58_0(null, null, $partial_4_6$Main__buildExpressionParser_58_makeParser_58_0(null, null, null, null), Main__term(), Main__table())), Main__intConst()), "simple expression");
}
```

So, pretty bonkers looking javascript, but the function names are nice, which was enough to give me a stack track that looked like `term` was in a loop parsing `"("` and `")"`, which looks like left recursion to me!

With some help from a friend turns out the answer was to [rewrite term](https://github.com/alexhumphreys/pl-playground/commit/cebbcd042354b9a71af83003f30e2ed1f23c3419) in a lazy way like so:

```
term : Parser Integer
term = intConst <|>| parens expr
         <?> "simple expression"
```

But yeah, goofy javascript debugging certainly helped. Almost certain there's a better way, but I'll take what I can get.
