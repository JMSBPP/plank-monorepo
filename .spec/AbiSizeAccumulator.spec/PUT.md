> EXTEND IF NEEDED
# [PUT](https://hackage.haskell.org/package/binary-0.8.9.3/docs/src/Data.Binary.Put.html) :: \(\mathrm{B}_p (n) \, \subset \mathrm{B}(n)    \)


- `BufferRange` is a half-open span of bytes inside a buffer

```haskell
-- A range of bytes in a buffer: the pointer to the first byte of the
-- range, and the pointer to the first byte after the range.
data BufferRange = BufferRange
  !(Ptr Word8)   -- first byte of the range
  !(Ptr Word8)   -- first byte after the range
```

- `BuildSignal` is a three-way report

```haskell
data BuildSignal a
  = Done        !(Ptr Word8) a
  | BufferFull  !Int !(Ptr Word8) (BuildStep a)
  | InsertChunk !(Ptr Word8) ByteString (BuildStep a)
```


- `BuildStep` is one fill of the output buffer. It is the function a `Builder` composes.

```haskell
type BuildStep a = BufferRange -> IO (BuildSignal a)
```

- `Builder` is a monoid which is a *description* sequence of bytes. **It represents a byte recipe**

```haskell
newtype Builder = Builder (forall r. BuildStep r -> BuildStep r)
```
- `PairS` is a pair of a result and a **byte recipe**

```
data PairS a = PairS a !Builder
```
> what does ! mean ?
>> A constructor field that makes it strict. This is the function that builds the value of a type

- `PutM` is a *writer monad* whose log is a bytestring `Builder`

> what is a monad ? what is a writer monad ? 
>> Is a type constructor that sequences computations, each of which yields a result
> what is a writer monad ? 
>> Is a monad dotted with a log monoid, in this case note Builder is a log monoid 

```
newtype PutM a = Put { unPut :: PairS a }
```

- `Put` is the `()` case of `PutM`
> what does it mean to be a '()' case of a monad another type PutM ? what are the semantics of a writer monad?
>> () is the result type fixed to (). The result carries no information. Althou competition still runs. Is somewhat similar to declaring 'never' on a plank evem fucntion, computation runs, but returns nothing 

```
type Put = PutM ()
```
