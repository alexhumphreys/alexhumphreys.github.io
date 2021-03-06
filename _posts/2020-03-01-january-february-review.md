_[mostly of interest to myself as a future reference]_

I set myself a goal of learning how programming languages work this year, and to use exercise as an excuse to write [Idris](https://www.idris-lang.org/index.html). Here's what I've been up to so far. 

Started off with this python tutorial on how to [interpret a simple imperative language](http://www.jayconrod.com/posts/37/a-simple-interpreter-from-scratch-in-python--part-1-). Got a bit stuck on the meaning of a lot of basic terms (expression, term, factor...) so checked out this tutorial on [basic maths terms](http://zonalandeducation.com/mmts/expressions/termsAndFactors.html).

The python tutorial uses parser combinators for parsing, and I wasn't sure how they worked. I learned a good bit by going through the start of this paper on [monadic parser combinators](http://www.cs.nott.ac.uk/~pszgmh/monparsing.pdf).

After that I took a step back from the python tutorial and [implemented a calculator](https://github.com/alexhumphreys/pl-playground/blob/master/calculator/calc.idr) by copying [this repo](https://github.com/steshaw/idris-calc). That made a bunch of things about expressions start to make sense, so I got brave and ported haskell's [buildExpressionParser](https://hackage.haskell.org/package/attoparsec-expr-0.1.1.2/docs/Data-Attoparsec-Expr.html) to Idris. Can see it [here](https://github.com/alexhumphreys/pl-playground/tree/01f62d8fa36baef3ddcaa4978ba8447c5df3ce22/BuildExpressoinParser).

With that under my belt I went back and finished parsing the imperative language, with the help of [this Haskell tutorial](https://wiki.haskell.org/Parsing_a_simple_imperative_language), the results are [up here](https://github.com/alexhumphreys/pl-playground/tree/01f62d8fa36baef3ddcaa4978ba8447c5df3ce22/Imperative).

That was mostly parsing, and there's a few more fun steps in how programming languages work, like type checking and evaluation. Still shaky on these areas, but as of the end of February I'm starting to explore them via lambda calculus.

Had a good talk with an old friend who knows this stuff, and he gave me a whirlwind tour of untyped, simply typed and dependently typed lambda calculus, along with using the [dhall typechecker](https://github.com/dhall-lang/dhall-haskell/blob/master/dhall/src/Dhall/TypeCheck.hs) for examples. I should write up this talk sometime, was really helpful.

I've started reading pu on [David Christiansen's](http://davidchristiansen.dk/tutorials/implementing-types-hs.pdf) and [Andras Kovaks'](https://github.com/AndrasKovacs/elaboration-zoo) work on typechecking/normalisation. So far I have copied this [parser/evaluator](https://github.com/AndrasKovacs/elaboration-zoo/tree/89bd75a2fe3fb9aa8e505bb89723acd8f0e730a9/01-eval) for untyped lambda calculus. I've used it to play with arithmetic and Boolean logic, so I can get a better handle on lambda calculus in general. 

Currently working on explicitly writing out a few lambda calculus substitutions, much as they do in the middle of [this](https://blog.brunobonacci.com/2017/10/08/lambda-calculus-and-boolean-logic/) post. Just posted some [boolean expansions](https://alexhumphreys.github.io/2020/03/01/january-february-review.html), will do some arithmetic ones soon. 

Once I get that down, next stop is the [typechecker from elaboration zoo](https://github.com/AndrasKovacs/elaboration-zoo/tree/master/02-typecheck). This has given me a bit of a goal to aim for with all this, but more on that later. 
