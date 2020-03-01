Still trying to wrap my head around lambda calculus, so I thought I'd write out a few expansions/substitutinos for practice. First example is taken from [this blog post](https://blog.brunobonacci.com/2017/10/08/lambda-calculus-and-boolean-logic/) by Bruno Bonacci, with the syntax changed to match the evaluator I've been working on [here](https://github.com/alexhumphreys/pl-playground/tree/e2658e94da304d0d8fdf233c8602e8cac1aaee14/lambda-calc)

```
let TRUE = (\\x y. x) in
let FALSE = (\\x y. y) in
let AND = (\\x y. (x (y TRUE FALSE) FALSE)) in
let OR  = (\\x y. (x TRUE (y TRUE FALSE))) in
let NOT = (\\b.  (b FALSE TRUE)) in
...
```

### Example one: `AND`

```
AND TRUE FALSE

-- expand AND
(\\x y. (x (y TRUE FALSE) FALSE)) TRUE FALSE
-- substiture last two parameters
(TRUE (FALSE TRUE FALSE) FALSE)
-- expand first TRUE
((\\x y. x) (FALSE TRUE FALSE) FALSE)
-- return first parameters
(FALSE TRUE FALSE)
-- expand first FALSE
((\\x y. y) TRUE FALSE)
-- return second parameter
(FALSE) -- expected value for (AND TRUE FALSE)
```

### Example two: `OR`

```
OR TRUE FALSE
-- expand OR
(\\x y. (x TRUE (y TRUE FALSE))) TRUE FALSE
-- substitue last two parameters
(TRUE TRUE (FALSE TRUE FALSE))
-- expand first TRUE
((\\x y. x) TRUE (FALSE TRUE FALSE))
-- return first parameter
(TRUE) -- expected value of OR TRUE FALSE
```

### Example three: `OR` again

```
OR FALSE TRUE
-- expand OR
(\\x y. (x TRUE (y TRUE FALSE))) FALSE TRUE
-- substitue last two parameters
(FALSE TRUE (TRUE TRUE FALSE))
-- expand first FALSE
((\\x y. y) TRUE (TRUE TRUE FALSE))
-- return second parameter
(TRUE TRUE FALSE)
-- expand first TRUE
((\\x y. x) TRUE FALSE)
-- return first parameter
(TRUE) -- expected value of OR FALSE TRUE
```

### Example four: `NOT`

```
NOT TRUE
-- expand NOT
(\\b. (b FALSE TRUE)) TRUE
-- substitute TRUE paramenter
(TRUE FALSE TRUE)
-- expand first TRUE
((\\x y. x) FALSE TRUE)
-- return first parameter
(FALSE) -- expected value of NOT TRUE
```
