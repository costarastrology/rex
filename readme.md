http://hackage.haskell.org/package/rex

Provides a quasi-quoter for regular expressions which yields a tuple, of 
appropriate arity and types, representing the results of the captures.  Allows 
the user to specify parsers for captures as inline Haskell.  Can also be used to
provide typeful pattern matching in function definitions and case patterns.

To build / install:

```
./Setup.hs configure --user
./Setup.hs build
./Setup.hs install
```

See the haddock or `Text/Regex/PCRE/Rex.hs` for documentation.

Some examples (verbatim from Test.hs):

```haskell
math x = mathl x 0

mathl [] x = x
mathl [rex|^  \s*(?{ read -> y }\d+)\s*(?{ s }.*)$|] x = mathl s y
mathl [rex|^\+\s*(?{ read -> y }\d+)\s*(?{ s }.*)$|] x = mathl s $ x + y
mathl [rex|^ -\s*(?{ read -> y }\d+)\s*(?{ s }.*)$|] x = mathl s $ x - y
mathl [rex|^\*\s*(?{ read -> y }\d+)\s*(?{ s }.*)$|] x = mathl s $ x * y
mathl [rex|^ /\s*(?{ read -> y }\d+)\s*(?{ s }.*)$|] x = mathl s $ x / y
mathl str x = error str

-- math "1 + 3" == 4.0
-- math "3 * 2 + 100" == 106.0
-- math "20 / 3 + 100 * 2" == 213.33333333333334
```


```haskell
peano :: String -> Maybe Int
peano = [rex|^(?{ length . filter (=='S') } \s* (?:S\s+)*Z)\s*$|]

--  peano "S Z" == Just 1
--  peano "S S S S Z" == Just 4
--  peano "S   S   Z" == Just 2
```

```haskell
parsePair :: String -> Maybe (String, String)
parsePair = [rex|^<\s* (?{ }[^\s,>]+) \s*,\s* (?{ }[^\s,>]+) \s*>$|]

--  parsePair "<-1, 3>" == Just ("-1","3")
--  parsePair "<-4,3b0>" == Just ("-4","3b0")
--  parsePair "< a,  -30 >" == Just ("a","-30")
--  parsePair "< a,  other>" == Just ("a","other")
```


```haskell
-- From http://www.regular-expressions.info/dates.html
parseDate :: String -> Maybe (Int, Int, Int)
parseDate [rex|^(?{ read -> y }(?:19|20)\d\d)[- /.]
                (?{ read -> m }0[1-9]|1[012])[- /.]
                (?{ read -> d }0[1-9]|[12][0-9]|3[01])$|]
  |  (d > 30 && (m `elem` [4, 6, 9, 11]))
  || (m == 2 &&
       (d ==29 && not (mod y 4 == 0 && (mod y 100 /= 0 || mod y 400 == 0)))
    || (d > 29)) = Nothing
  | otherwise = Just (y, m, d)
parseDate _ = Nothing

--  parseDate "1993.8.10" == Nothing
--  parseDate "1993.08.10" == Just (1993,8,10)
--  parseDate "2003.02.28" == Just (2003,2,28)
--  parseDate "2003.02.27" == Just (2003,2,27)
```

```haskell
onNull a f [] = a
onNull _ f xs = f xs

nonNull = onNull Nothing

disjunct [rex| ^(?:(?{nonNull $ Just . head -> a} .)
             | (?{nonNull $ Just . head -> b} ..)
             | (?{nonNull $ Just . last -> c} ...))$|] =
  head $ catMaybes [a, b, c]

--  disjunct "a" == 'a'
--  disjunct "ab" == 'a'
--  disjunct "abc" == 'c'
```
