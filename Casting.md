# Casting and converting types

Be warned. This is an area where, frankly, F# has **too many ways to do it**.
Some of them are straight-up redundant: there are multiple syntaxes to write
code with identical semantics.

I consider this excess of options a wart on the language, but none of the
options are _bad_, so it's fine once you know them. It's just an unnecessarily
large number of things to learn.

At the bottom of this page I express my opinions on which of the redundant
casting approaches to settle on.

## Upcasting (treating a child type as its parent type)

```csharp
MemoryStream memStream = new MemoryStream();
Stream stream = memStream; // upcasting is always implicit in C#
IDisposable disposable = stream; // same with casting to an interface type
```

This operation can never fail at runtime, which is why in C# it is done
automatically. In F#, for type inference reasons, it is not always done
automatically. The operators for doing it are separate from the downcasting or
conversion operators though, so that you know not to worry when you see an
upcast as it doesn't have the same potential for runtime failure that a downcast
does.

You perform an upcast with either the keyword `upcast`, or the operator `:>`. If
you are not casting from a child type to a parent type, this will be caught at
compile time.

```fsharp
let memStream : MemoryStream = new MemoryStream()

// you can use `upcast`
let stream1 : Stream = upcast memStream
let disposable1 : IDisposable = upcast stream1

// or `:>` -- the compiled code is the same either way
let stream2 = memStream :> Stream
let disposable2 = stream2 :> IDisposable
```

Note that upcasts *are* implicit when passing arguments to a method.

```fsharp
type Example =
    static member MethodTakingObject(x : obj) = Console.WriteLine(x)

let myInteger : int = 1
Example.MethodTakingObject(myInteger) // no need to explicitly upcast int to obj here
```

## Forced downcasting (treating a parent type as one of its child types)

```csharp
Stream stream = new MemoryStream();
// both the below casts will compile, but the 2nd will fail at runtime
MemoryStream memoryStream = (MemoryStream)stream;
FileStream fileStream = (FileStream)stream; // throws InvalidCastException
```

Again, in F# there are two ways to do this: a `downcast` keyword, and an
operator `:?>`.

```fsharp
let stream : Stream = upcast new MemoryStream()

// you can use `downcast`
let memoryStream1 : MemoryStream = downcast stream
let fileStream1 : FileStream = downcast stream // throws InvalidCastException

// or `:?>` -- the compiled code is the same either way
let memoryStream2 = stream :?> MemoryStream
let fileStream2 = stream :?> FileStream // throws InvalidCastException
```

## Testing type at runtime (C# `is` operator)

```csharp
object x = 1;
var isString = x is string; // false
var isInteger = x is int; // true
```

```fsharp
let x = 1 :> obj
let isString = x :? string; // false
let isInteger = x :? int; // true
```

## Type conditional assignment (C# `as` operator, C# 7 `is` pattern matching)

The old C# way (before C# 7.0):

```csharp
object x = "hello";

var s = x as string;
if (s != null)
{
    Console.WriteLine(s.Length); // yay, we can use this as a string
}
else
{
    var i = x as int?;
    if (i != null)
    {
        Console.WriteLine(i.Value + 1); // yay, we can use this as an int
    }
    else
    {
        Console.WriteLine("Not a string or int");
    }
}
```

The new C# way (since C# 7.0):

```csharp
object x = "hello";
switch (x)
{
    case string s:
        Console.WriteLine(s.Length); // yay, we can use this as a string
        break;
    case int i:
        Console.WriteLine(i + 1);
        break;
    default:
        Console.WriteLine("Not a string or int");
        break;
}
```

The F# way is very similar to the C# 7 way (which it inspired).

```fsharp
let x = "hello" :> obj
match x with
| :? string as s ->
    Console.WriteLine(s.Length) // yay, we can use this as a string
| :? int as i ->
    Console.WriteLine(i + 1)
| _ ->
    Console.WriteLine("Not a string or int")
```

Note that all 3 approaches (the 2 C# ways and the 1 F# way) treat nulls the same
way. A null object will not match the test for `string` (or any other type),
even though technically null is a legal value for a string variable.

## Downcasting null to "non-nullable" F# reference types

F# tries to prevent you from putting nulls in reference types, encouraging the
use of `Option` types instead. By default, types defined in F# don't allow you
to assign `null` to them in your code.

In order to help prevent nulls getting into those types via casting, the
`downcast` and `:?>` operators check for null when downcasting to an F# type
that is not supposed to permit nulls.

```
[<AllowNullLiteral>]
type CSharpyReferenceType() = class end

type FSharpyReferenceType() = class end

let nullObj : obj = null

let compileExample1 : CSharpyReferenceType = null // this compiles
let compileExample2 : FSharpyReferenceType = null // this line does NOT compile

let castExample1 = nullObj :?> CSharpyReferenceType // this succeeds, example1 contains null
let castExample2 = nullObj :?> FSharpyReferenceType // this fails with a NullReferenceException

```

If you would like to avoid this null checking behavior, at the risk of getting a
null in a variable of a supposedly non-nullable type, you can use
`Unchecked.unbox`. This inline function has the type `obj -> 'a`. It compiles to
an `unbox.any` opcode like a C# downcast.

```
let castExample3 : FSharpyReferenceType = Unchecked.unbox nullObj
```

## box and unbox

In addition to `Unchecked.unbox`, there is an unadorned `unbox` function. This
behaves just like the other ways of downcasting, including the null check for F#
reference types. However, be advised that if you pass an input of a type other
than `obj`, the input will be implicitly upcasted, so the compiler cannot detect
some obvious errors in usage.

```fsharp
// the compiler thinks this is OK, since it implicitly upcasts the MemoryStream
// to obj before passing it into `unbox`
let fs1 : FileStream = unbox (new MemoryStream())

// the compiler can tell that this conversion is never going to work, and will
// give error FS0193: Type constraint mismatch.
let fs2 = new MemoryStream() :?> FileStream
```

For symmetry, a `box` function also exists, which converts its input (of any
type) to `obj`. This behaves just like upcasting to `obj`. There is no
significant upside or downside between upcasting to `obj` vs. using `box`.

## Converting between primitive types

In C# syntax, casting and conversion look exactly the same.

* Casting: treating the same (ignoring boxing) value as a different type
* Conversion: translating a value into a new value of a different type

In F# these are completely separate concerns. Thus far, we have covered casting
only. Primitive conversions are handled by built-in inline functions named for
the primitive type they convert to:

```csharp
int x = 1;
short y = (short)x; // explicit conversion
long z = x; // implicit conversion
double q = x; // implicit conversion
```

```fsharp
let x = 1
let y = int16 x
let z = int64 x
let q = double x
```

## Invoking C# conversion operator overloads

In C#, you can define implicit and explicit conversion operators as static
methods. The compiler will call these when you use the cast syntax. Some types
in the framework have these. For example, there is (unfortunately) an implicit
conversion operator from `DateTime` to `DateTimeOffset`.

```csharp
DateTimeOffset myDateTimeOffset = DateTime.UtcNow; // implicit conversion
```

In F# there is no implicit conversion, ever. Nor is there any built-in special
syntax for calling these custom cast operators. So you're stuck calling the
actual .NET method:

```fsharp
let myDateTimeOffset = DateTimeOffset.op_Implicit(DateTime.UtcNow)
```

Depending on the API you're working with, often times there is another way to do
the same thing. In this case I could simply use the constructor overload that
takes a `DateTime`:

```fsharp
let myDateTimeOffset = new DateTimeOffset(DateTime.UtcNow)
```

However, if you really need to call C# style conversion operators frequently, F#
has you covered with inline functions. Just define:

```fsharp
let inline implicitConvert (x : ^a) : ^b = ((^a or ^b) : (static member op_Implicit : ^a -> ^b)(x))
```

Now you can write:

```fsharp
let myDateTimeOffset : DateTimeOffset = implicitConvert DateTime.UtcNow
```

# Choosing between redundant options

`upcast` is the same thing as `:>` and `downcast` is the same thing as `:?>`.
So, which should you use?

My preference is to ignore the `upcast` and `downcast` keywords, and always use
the equivalent `:>` and `:?>` operators. Here are some relative strengths to consider:

In favor of `upcast` and `downcast`:

* They are self-describing, which may help readability for people unfamiliar
  with F#.

* They look nice when the type being casted to is inferred by the compiler, such
  as when returning the result of an interface method.


In favor of `:>` and `:?>`:

* The closely related `:?` type test pattern (used in `match` expressions) is
  often useful, and has no alternative syntax. The `:>` and `:?>` operators are
  visually similar to the `:?` pattern, so it's more consistent.

* Specifying the type being casted to is clearer with the `:>` and `:?>`
  operators, since it goes right next to the expression being casted.

* In places where the type being casted to can be inferred, you can use `_` as
  in `x :> _`, which while not as pretty as `upcast x`, is not bad either.

What about `box` and `unbox`?

I use these:

* To emphasize when I'm casting to and from `obj` instead of between more specific types.

* When it is handy to have a function instead of an operator, as in `Seq.map
  box` instead of `Seq.map (fun x -> x :> obj)`.

