# Namespace and namespace imports

```csharp
using System;
using System.Diagnostics;
using System.IO;
using System.Text.RegularExpressions;

namespace MyNamespace
{
    // ...
```

```fsharp
namespace MyNamespace
open System
open System.Diagnostics
open System.IO
open System.Text.RegularExpressions
```

# Static classes

```csharp
public static class MyStaticClass
{
    public static string FullName(string firstName, string lastName)
    {
        return firstName + " " + lastName;
    }
}
```

```fsharp
type MyStaticClass =
    static member FullName(firstName : string, lastName : string) : string =
        firstName + " " + lastName
```

# Visibility modifiers

```csharp
internal static class MyInternalStaticClass
{
    private static void SecretMethod()
    {
        Console.WriteLine("Hello");
    }
}
```

```fsharp
// visibility modifiers always go right before the name of the class or method
type internal MyInternalStaticClass =
    static member private SecretMethod() : unit = // unit is like void
        Console.WriteLine("Hello")
```

# Instantiable classes

## Note

It is common to have constructors that assign private instance fields, either
from their arguments or from some objects they "new" up (like a list or
dictionary). F# is designed to make these very easy to write as "primary
constructors". However, you are *not* allowed to do something you can do in C#,
which is have multiple constructors that initialize the class in totally
different ways. You *can* have multiple constructors, but they **must** delegate
to the primary constructor to do initialization.

```csharp
[Serializable]
public class Circle
{
    private readonly double _radius;
    private readonly double _diameter;

    public Circle(double radius)
    {
        _radius = radius;
        _diameter = radius * 2; // compute when constructed and keep in an instance field
        Debug.WriteLine("constructed a circle");
    }
    public Circle() : this(1.0) { }
    public Circle(int radius) : this((double)radius) { }

    public double Radius => _radius;
    public double Diameter => _diameter;
    public double Area => _radius * _radius * Math.PI;
}
```

```fsharp
type Circle(radius : double) =
    // this code is the "primary constructor"

    let diameter = radius * 2.0 // compute when constructed and keep in an instance field
    do // if you want to run some code in the constructor but not assign a variable, use a "do" block
        Debug.WriteLine("constructed a circle")

    // primary constructor ends whereever you start defining methods
    // or other constructores

    // other constructor overloads can be defined for convenience,
    // but *MUST* be wrappers around the primary constructor
    new() = Circle(1.0)
    new(radius : int) = Circle(double(radius))

    member this.Radius = radius
    member this.Diameter = diameter
    member this.Area = radius * radius * Math.PI
```

# Properties and auto-properties

```csharp
public class BagOfData
{
    private double _zField;

    // auto-properties
    public int X { get; set; }
    public string Y { get; set; } = "hello";

    // explicitly implemented property
    public double Z
    {
        get { return _zField; }
        set { _zField = value; }
    }
}
```

```fsharp
// Note: it is uncommon to see setters in F# code due to the
// community's preference for immutable data.
// F#'s record types replace most "POCO" style objects.
type BagOfData() =
    let mutable zField = 0.0

    // auto-properties
    member val X = 0 with get, set
    member val Y = "hello" with get, set

    // explicitly implemented property
    member this.Z
        with get() = zField
        and set(value) = zField <- value
```

# Structs (value types)

Generally it is sufficient to just put the `[<Struct>]` attribute on a type to
make it a value type.

```csharp
[Serializable]
public struct Point
{
    public int X { get; }
    public int Y { get; }
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
}
```

```fsharp
[<Struct>]
type Point(x : int, y : int) =
    member this.X = x
    member this.Y = y
```

# Inheritance

```csharp

public class Greeter
{
    private readonly string _hello;
    public Greeter(string hello)
    {
        _hello = hello;
    }
    public Greeter() : this("Hello") { }
    public virtual void Hello(string subject)
    {
        Console.WriteLine(_hello + " " + subject);
    }
}

public class PoliteGreeter : Greeter
{
    public PoliteGreeter() : base("Salutations") { }
    public override void Hello(string subject)
    {
        base.Hello(subject);
        Console.WriteLine("How are you doing?");
    }
}

```

```fsharp

type Greeter(hello) =
    new() = Greeter("Hello")
    abstract member Hello : subject : string -> unit
    default this.Hello(subject) =
        Console.WriteLine(hello + " " + subject)

type PoliteGreeter() =
    inherit Greeter("Salutations")
    override this.Hello(subject) =
        base.Hello(subject)
        Console.WriteLine("How are you doing?")

```

# Abstract classes

```csharp

public abstract class Dismisser
{
    public abstract void Dismiss(string subject);
}

public class RudeDismisser : Dismisser
{
    public override void Dismiss(string subject)
    {
        Console.WriteLine("Get outta here, " + subject);
    }
}
```

```fsharp

[<AbstractClass>]
type Dismisser() =
    abstract member Dismiss : subject : string -> unit

type RudeDismisser() =
    inherit Dismisser()
    override this.Dismiss(subject) =
        Console.WriteLine("Get outta here, " + subject)

```

# Defining interfaces

```csharp

public interface IShape
{
    void Hello();
    double Area { get; }
    double Perimeter { get; }
    IShape Resize(double scaleX, double scaleY);
}
```

```fsharp

type IShape =
    abstract Hello : unit -> unit
    abstract Area : double
    abstract Perimeter : double
    abstract Resize : scaleX : double * scaleY : double -> IShape

```

# Explicitly implementing interfaces

```csharp

[Serializable]
public class Rectangle : IShape
{
    private readonly double _width;
    private readonly double _height;

    public Rectangle(double width, double height)
    {
        _width = width;
        _height = height;
    }

    public double Width => _width;
    public double Height => _height;

    void IShape.Hello()
    {
        Console.WriteLine("I am a rectangle!");
    }
    double IShape.Area => _width * _height;
    double IShape.Perimeter => _width * 2 + _height * 2;
    IShape IShape.Resize(double scaleX, double scaleY)
        => new Rectangle(_width * scaleX, _height * scaleY);
}

```

```fsharp

type Rectangle(width, height) =
    member this.Width = width
    member this.Height = height
    interface IShape with
        member this.Hello() = Console.WriteLine("I am a rectangle!")
        member this.Area = width * height
        member this.Perimeter = width * 2.0 + height * 2.0
        member this.Resize(scaleX, scaleY) =
            // must explicitly upcast to IShape with :> operator
            // this is checked by the compiler to be a safe cast (it cannot fail at runtime)
            Rectangle(width * scaleX, height * scaleY) :> IShape

```

# Implicitly implementing interfaces

```csharp

[Serializable]
public class RightTriangle : IShape
{
    private readonly double _width;
    private readonly double _height;

    public RightTriangle(double width, double height)
    {
        _width = width;
        _height = height;
    }

    void Hello()
    {
        Console.WriteLine("I am a right triangle!");
    }
    public double Area => _width * _height * 0.5;
    public double Perimeter => _width + _height + Math.Sqrt(_width * _width + _height * _height);
    public IShape Resize(double scaleX, double scaleY)
        => new RightTriangle(_width * scaleX, _height * scaleY);
}

```

```fsharp

type RightTriangle(width, height) =
    member this.Hello() = Console.WriteLine("I am a right triangle!")
    member this.Area = width * height * 0.5
    member this.Perimeter = width * height + sqrt(width * width + height * height)
    member this.Resize(scaleX, scaleY) =
        RightTriangle(width * scaleX, height * scaleY)
    interface IShape with
        // F# does not support implicit interface implementation, so you have to explicitly
        // delegate to the methods on the class
        member this.Hello() = this.Hello()
        member this.Area = this.Area
        member this.Perimeter = this.Perimeter
        member this.Resize(scaleX, scaleY) = this.Resize(scaleX, scaleY) :> IShape

```

# Enums

```csharp

public enum Suit
{
    Hearts = 1,
    Diamonds,
    Spades,
    Clubs,
}

[Flags]
public enum PermissionFlags : ushort
{
    Read = 0,
    Write = 1,
    Execute = 2,
    Move = 4,
    Delete = 8,
}

```

```fsharp

// Note: it is idiomatic in F# to use a discriminated union instead. Enums are only
// used rarely, for performance, bit flags, or compatibility with C# and VB.NET.
type Suit =
    | Hearts = 1
    | Diamonds = 2
    | Spades = 3
    | Clubs = 4

[<Flags>]
type PermissionFlags =
    | Read = 0us
    | Write = 1us
    | Execute = 2us
    | Move = 4us
    | Delete = 8us

```

# Mutable fields

```csharp
[Serializable]
public class Point
{
    public int X;
    public int Y;
}

// example usage
var p = new Point { X = 1, Y = 2 };
p.X = 3;
```

```fsharp
type Point =
    val mutable public X : int
    val mutable public Y : int

// example usage
let p = Point(X = 1, Y = 2)
p.X <- 3
```

# Extension methods

```csharp

public static class MyExtensions
{
    public static string[] Lines(this string str)
        => Regex.Split(str, @"(?:\r\n)|[\r\n]");
}

```

```fsharp

open System.Runtime.CompilerServices

// Note: there is also an F#-specific extension method syntax.
// This is the C# equivalent way to do it though.

[<Extension>]
type MyExtensions =
    [<Extension>]
    static member Lines(str) =
        Regex.Split(str, @"(?:\r\n)|[\r\n]")

```

# Generic classes

```csharp

public class Wrapper<T>
{
    private readonly T _it;
    public Wrapper(T it)
    {
        _it = it;
    }
    public T It => _it;
}

```

```fsharp

type Wrapper<'a>(it : 'a) =
    member this.It = it

```

# Generic methods

```csharp

public class ServiceProvider
{
    private readonly IServiceProvider _underlyingProvider;
    public ServiceProvider(IServiceProvider underlying)
    {
        _underlyingProvider = underlying;
    }
    public T GetService<T>() => (T)(_underlyingProvider.GetService(typeof(T)));
}

```

```fsharp
type ServiceProvider(underlyingProvider : IServiceProvider) =
    // :?> is the unsafe cast operator
    member this.GetService<'a>() = unbox(underlyingProvider.GetService(typeof<'a>)) : 'a
```

# Generic constraints

```csharp

public class SimpleFactory<T> where T : new()
{
    public T Create() => new T();
}

```

```fsharp

type SimpleFactory<'a when 'a : (new : unit -> 'a)>() =
    member this.Create() = new 'a()

```