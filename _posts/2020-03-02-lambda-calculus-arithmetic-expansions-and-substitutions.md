Still playing with lambda calculus, this time I'm writing out a few expansions/substitutions/reductions for arithmetic, specifically the `succ` and `add` functions. Shout out to [this stack overflow answer](https://stackoverflow.com/a/47085844/1776944) for the sweet notation.

### Example one: S3
```
S = (\\w y x. y (w y x))
3 = (\\s z. s (s (s (z))))

0  | S 3
1  | (\\w y x. y (w y x)) 3                 | by def
2  | (\\y x. y (3 y x))                     | beta
3  | (\\y x. y ((\\s z. s(s (s (z)))) y x)) | by def
4  | (\\y x. y (y (y (y (x)))))             | beta
```

### Example two: 2S3

```
2 S 3
2 = (\\s z. s (s (z)))
S = (\\w y x. y (w y x))
3 = (\\s z. s (s (s (z))))

0  | 2 S 3
1  | 2 (\\w y x. y (w y x)) 3                                        | by def
2  | (\\s z. s (s (z))) (\\w y x. y (w y x)) 3                       | by def
3  | ((\\w y x. y (w y x)) ((\\w y x. y (w y x)) (3)))               | beta
4  | ((\\w y x. y (w y x)) ((\\y x. y (3 y x))))                     | beta
5  | ((\\w y x. y (w y x)) ((\\y x. y((\\s z. s (s (s (z)))) y x)))) | by def
6  | ((\\w y x. y (w y x)) ((\\y x. y (y (y (y (x)))))))             | beta
7  | ((\\w y x. y (w y x)) ((\\s z. s (s (s (s (z)))))))             | alpha
8  | ((\\y x. y (((\\s z. s (s (s (s (z)))))) y x)))                 | beta
9  | ((\\y x. y (y (y (y (y (x)))))))                                | beta
```

### Example three: ADD 2 2
```
2 = (\\f x. f (f x))
ADD = (\\m n f x. m f (n f x))

0  | ADD 2 2
1  | (\\m n f x. m f (n f x)) 2 2           | by def
2  | (\\f x. 2 f (2 f x))                   | beta
3  | (\\f x. 2 f ((\\f x. f (f x)) f x))    | by def
3  | (\\f x. 2 f ((\\s z. s (s z)) f x))    | alpha
4  | (\\f x. 2 f (f (f x)))                 | beta
5  | (\\f x. (\\f x. f (f x)) f (f (f x)))  | by def
5  | (\\f x. (\\s z. s (s z)) f (f (f x)))  | alpha
5  | (\\f x. f (f (f (f x))))               | beta
```

Got a bit confused between examples online that were like `2S3` or `ADD 2 3`. The first calls the successor function twice on `3`, while the latter adds `2` and `3`. That makes it pretty clear that `ADD` and `S` are very different functions, but my brain got confused and tried to switch between the examples as if they were the same. Lesson learned about slowing down and reading stuff more carefully. 
