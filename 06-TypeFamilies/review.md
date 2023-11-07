# TypeFamilies Review

Type families allow us to express type computation with
some strong limitations:
- There are no let bindings (you need another decl)
- There is no guarding or value-level computation
- There are no Higher-Order Functions
(Some languages built on Haskell escape these
limitations, like Coq, Idris and Agda.)

Even with these limitations, they can do quite a bit.

``` haskell
-- a family that enforces a list of constraints
type family Every (c :: Type -> Constraint) (x :: [Type]) :: Constraint where
  Every _ '[] = ()
  Every f (h ': t) = (f h, Every f t)
  -- | a note about tuples of constraints:
  --   ghc automatically flattens tuples of
  --   constraints into a 1-deep n-tuple:
  --     ((a x, ), (), b y, (c z, d)) ~>
  --          (a x, b y, c z, d)
```

Though, due to those limitations, helper families are
almost always necessary.

``` haskell
type family Insert (x :: Nat) (t :: Tree) :: Tree where
  Insert e 'Empty = Node 'Empty e 'Empty
  Insert e ('Node l a r) = Insert' (Compare e a) e ('Node l a r)

-- a helper for recursion with ordering (can't use if/case)
type family Insert' (ord :: Ordering) (x :: Nat) (t :: Tree) where
  Insert' 'LT x ('Node l a r) = 'Node (Insert x l) a r
  Insert' 'EQ _ ('Node l a r) = 'Node l a r
  Insert' 'GT x ('Node l a r) = 'Node l a (Insert x r)
```

## Open and Incomplete Families

Interestingly, you may create Open families like so:
``` haskell
type family Open (x :: Type) :: Type

type instance Open Int = String
type instance Open (Maybe Bool) = IO Int
-- ...
```

Or incomplete families:
``` haskell
type family Subtract (x :: Nat) (y :: Nat) :: Nat where
  Subtract     x   'Z    = x
  Subtract ('S x) ('S y) = Subtract x y

-- Subtract 'Z ('S 'Z) ~> Subtract 'Z ('S 'Z)
-- no definition for negative numbers means they simply
-- don't reduce any further
```

## With GHCi

To begin you must set all your extensions as active:
```
> :set -XDataKinds -XTypeFamilies ...
```

Then you can ask ghci for the kinds with `:kind!`
```
> ... load defn of Nat and Subtract
> :kind! Subtract ('S 'Z) 'Z
'S 'Z :: Nat
```