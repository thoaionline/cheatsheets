# Haskell Cheatsheet

## Type classes

### Semigroup

Elements that can be concatenated together

```
class Semigroup a where
    (<>) :: a -> a -> a
    sconcat :: [[Data.List.Nonempty|Nonempty]] a -> a
    stimes :: Integral b => b -> a -> a
```

Laws

```
(a <> b) <> c == a <> (b <> c)
```

### Monoid

Semigroup with a default element `mempty`.

```
class Semigroup m => Monoid m where
  mempty :: m

  -- defining mappend is unnecessary, it copies from Semigroup
  mappend :: m -> m -> m
  mappend = (<>)

  -- defining mconcat is optional, since it has the following default:
  mconcat :: [m] -> m
  mconcat = foldr mappend mempty
```

Laws

```
x <> mempty = x
mempty <> x = x
```

### Functor

A type that can be `fmap` over. (e.g. List, Maybe,)

```
class Functor f where
    -- alias: <$> - map over
    fmap :: (a -> b) -> f a -> f b
    
    -- project into
    (<$) :: a -> f b -> f a

    -- load
    ($>) :: f a -> b -> f b
``` 

A trick to remember the `<$` and `$>` operators is to imagine it as a party popper 🎉 where the small guy pops glitters over the rich ($) crowd!

The `<$>` looks like some wisdom seed (money in a bracket) being imparted to the crowd.

### Applicative

A structure

```
class Functor f => Applicative f where
    -- convert to structure
    pure :: a -> f a

    -- apply a formula
    (<*>) :: f (a -> b) -> f a -> f b
```

Did you notice that `<*>` looks like a spaceship? Yeah, because the secret has a container around it. This is an ET visiting earth!

### Monad

Chainable calculation

```
class Applicative m => Monad m where
    -- "bind" - calculate, then take the result and apply to another calculation
    (>>=) :: m a -> (a -> m b) -> m b

    -- "then" (discarding first result)
    (>>) :: m a -> m b -> m b
    
    -- wraping result
    return :: a -> m a
```

In above examples, treat `m` as a box. Read the `>>=` (bind) as "grab a value out of a box, run some calculation, put it in another box".

It is beneficial to deep-dive into how `do` blocks are only syntactic sugar for `>>=` and `>>`, but keep it easy and use `do` syntax when you first start, then rewrite your code to use `>>=` and `>>` where appropriate.

### Foldable

A Monoid that can be reduced/folded. Can be derived automatically for `X a` recursive types.

```
class Foldable (t :: * -> *) where
    fold :: Monoid m => t m -> m

    -- fold with preliminary transform from ordinary element to monoid (e.g. Int -> Sum Int)
    foldMap :: Monoid m => (a -> m) -> t a -> m
```

`t` is the foldable structure of monoids. Example: `fold [Sum 1, Sum 2]`.

### Traversable

A structure that can be traversed (iterated over).

```
class (Functor t, Foldable t) => Traversable t where
    traverse :: Applicative f => (a -> f b) -> t a -> f (t b)
    -- default implemantion
    traverse f = sequenceA . fmap f

    sequenceA :: Applicative f => t (f a) -> f (t a)
    sequenceA = traverse id
```

Reminder: Applicative `f` is a container. `a -> f b` is an action. `traverse` apply an action to each element of a structure and drop the transform structure into a new container.

`sequenceA` flip the structure into the container, e.g. a `[Maybe Int]` into `Maybe [Int]`.

### State

Quick reminder https://wiki.haskell.org/State_Monad
Explainer: https://williamyaoh.com/posts/2020-07-12-deriving-state-monad.html

```
newtype State s a = State { runState :: s -> (s, a) }

get    :: State s s
put    :: s -> State s ()
modify :: (s -> s) -> State s ()
```

Example:

```
addCounterToString :: String -> State Int String
addCounterToString s = modify (+ 1) >> get >>= \v -> return (s ++ " " ++ show v)

multipleRuns :: String -> State Int String
multipleRuns str = do
  a <- addCounterToString str
  b <- addCounterToString a
  return b
 
-- ("Hey 4 5",5)
runState (multipleRuns "Hey") 3
-- 5
execState (multipleRuns "Hey") 3
-- "Hey 4 5"
evalState (multipleRuns "Hey") 3
```

### Reader

Example
```
import Control.Monad.Reader

hello :: Reader String String
hello = do
    name <- ask
    return ("hello, " ++ name ++ "!")

bye :: Reader String String
bye = do
    name <- ask
    return ("bye, " ++ name ++ "!")

convo :: Reader String String
convo = do
    c1 <- hello
    c2 <- bye
    return $ c1 ++ c2

main = print . runReader convo $ "adit"
```

### Writer

```
half :: Int -> Writer String Int
half x = do
  tell $ "Halving " ++ (show x) ++ "!"
  return $ x `div` 2

heyhey :: IO ()
heyhey = do
  let writer :: Writer String Int
      writer = do
        let x = 10
        y <- half x
        z <- half y
        return z
      (result, log) = runWriter (writer)
  print (show result)
  print log
  return ()
```