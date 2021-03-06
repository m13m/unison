
```ucm
.> builtins.merge

  Done.

```
Standard helpful definitions

```unison
use io2

stdout : Handle
stdout = stdHandle StdOut

putLn : Text ->{IO} ()
putLn t =
  putBytes stdout (toUtf8 (t ++ "\n"))
  ()

map : (a ->{e} b) -> [a] ->{e} [b]
map f l = let
  go acc = cases
    [] -> acc
    x +: xs -> go (acc :+ f x) xs
  go [] l
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      map    : (a ->{e} b) -> [a] ->{e} [b]
      putLn  : Text ->{IO} ()
      stdout : Handle

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    map    : (a ->{e} b) -> [a] ->{e} [b]
    putLn  : Text ->{IO} ()
    stdout : Handle

```
Loops that access a shared counter variable, accessed in transactions.
Some thread delaying is just accomplished by counting in a loop.
```unison
use io2

count : Nat -> ()
count = cases
  0 -> ()
  n -> count (drop n 1)

inc : TVar Nat ->{IO} Nat
inc v =
  atomically 'let
    x = TVar.read v
    TVar.write v (x+1)
    x

loop : '{IO} Nat -> Nat -> Nat ->{IO} Nat
loop grab acc = cases
  0 -> acc
  n ->
    m = !grab
    count (m*10)
    loop grab (acc+m) (drop n 1)

body : Nat -> TVar (Optional Nat) -> TVar Nat ->{IO} ()
body k out v =
  n = loop '(inc v) 0 k
  atomically '(TVar.write out (Some n))
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      body  : Nat -> TVar (Optional Nat) -> TVar Nat ->{IO} ()
      count : Nat -> ()
      inc   : TVar Nat ->{IO} Nat
      loop  : '{IO} Nat ->{IO} Nat ->{IO} Nat ->{IO} Nat

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    body  : Nat -> TVar (Optional Nat) -> TVar Nat ->{IO} ()
    count : Nat -> ()
    inc   : TVar Nat ->{IO} Nat
    loop  : '{IO} Nat ->{IO} Nat ->{IO} Nat ->{IO} Nat

```
Test case.

```unison
spawn : Nat ->{IO} Result
spawn k = let
  out1 = TVar.newIO None
  out2 = TVar.newIO None
  counter = atomically '(TVar.new 0)
  forkComp '(Right (body k out1 counter))
  forkComp '(Right (body k out2 counter))
  p = atomically 'let
    r1 = TVar.read out1
    r2 = TVar.swap out2 None
    match (r1, r2) with
      (Some m, Some n) -> (m, n)
      _ -> !STM.retry
  max = TVar.readIO counter
  match p with (m, n) ->
    sum : Nat
    sum = max * drop max 1 / 2
    if m+n == sum
    then Ok "verified"
    else Fail (display m n sum)

display : Nat -> Nat -> Nat -> Text
display m n s =
  "mismatch: " ++ toText m ++ " + " ++ toText n ++ " /= " ++ toText s

nats : [Nat]
nats = [89,100,116,144,169,188,200,233,256,300]

tests : '{IO} [Result]
tests = '(map spawn nats)
```

```ucm

  I found and typechecked these definitions in scratch.u. If you
  do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      display : Nat -> Nat -> Nat -> Text
      nats    : [Nat]
      spawn   : Nat ->{IO} Result
      tests   : '{IO} [Result]

```
```ucm
.> add

  ⍟ I've added these definitions:
  
    display : Nat -> Nat -> Nat -> Text
    nats    : [Nat]
    spawn   : Nat ->{IO} Result
    tests   : '{IO} [Result]

.> io.test tests

    New test results:
  
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  ◉ tests   verified
  
  ✅ 10 test(s) passing
  
  Tip: Use view tests to view the source of a test.

```
