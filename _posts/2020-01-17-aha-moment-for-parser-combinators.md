So I've been starting to play around with parsing, following a few blogs and papers linked above in the header, and have been trying to write some of the ideas in those posts in Idris. This lead me to the parser combinator library [Lightyear](https://github.com/ziman/lightyear).

The library ain't big on documentation, I think it's expecting a famility with parser combinators in general, and Haskell's parsec in particular. I don't have eihter, so that makes for a rocky start. They do have one full [Json parser](https://github.com/ziman/lightyear/blob/6f56439ea4bdba6a3ad4a11221c6028ed9d74047/examples/Json.idr) which is pretty cool though.

After a few days bashing my head against this, especially the types of the parser functions, and of their return values. It finally started clicking for me with the [json bool parser](https://github.com/ziman/lightyear/blob/6f56439ea4bdba6a3ad4a11221c6028ed9d74047/examples/Json.idr#L99)

```
jsonBool : Parser Bool
jsonBool  =  (char 't' >! string "rue"  *> pure True)
         <|> (char 'f' >! string "alse" *> pure False) <?> "JSON Bool"
```

Breaking it down line by line, the first is the function definition: a function that returns a `Parser` and a `Bool`.

The second line is where it started to click for me: I think what's happening here is "try parse the character 't', if it's a 't' then try parse the string 'rue', if it's a 'rue' then you have a try so return (using the `*>` operator) a `Parser` and the value of `True`, thus matching the type signature". The `pure` there is creating/returning a value from inside the parser monads (glossing over some details I don't understand here).

Then the third line has this funky `<|>` operator. This is like `or`. So, if the first parser fails, try the second parser. Then it functions the same as the line before but with the string `false`. (I think the `<?> "JSON Bool"` at the end is some kind of documentation/output hint, I haven't understood that yet.)

That helped clear up a lot of the return values from these parser combinator functions for me. In Idris, `Bool` is defined as:

```
data Bool = False | True
```

So it's a type `Bool` with two constructors `False` and `True`. So when you have a parser that can return multiple things, you need a type that reflects that. For example, here's a parser that will match the letter 'a' or number '1', and marschall it (using `pure`) into a value in Idris to do something with:

```
data OneOrA = Number1 Nat
            | LetterA Char

oneOrA : Parser OneOrA
oneOrA = do
  c <- anyChar
  (case c of
        '1' => pure (Number1 1)
        'a' => pure (LetterA 'a')
        _   => fail "'1' or 'a' expected")
```

Running that in the REPL like so:

```
*parser> Lightyear.Strings.parse oneOrA "a"
Right (LetterA 'a') : Either String OneOrA
*parser> Lightyear.Strings.parse oneOrA "1"
Right (Number1 1) : Either String OneOrA
*parser> Lightyear.Strings.parse oneOrA "b"
Left "At 1:2:\n\t'1' or 'a' expected" : Either String OneOrA
```

Lightyear is pretty fancy and can do a lot of stuff, but it can be pretty overwhelming if you're not experienced in this area. This helped me get a bit of a foot hold to start experimenting with it more.
