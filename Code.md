# Code

This file contains translations for C# statements and expressions, i.e. code
that you might find within the body of a method.

For code _declaring_ classes and methods, see [Types.md](Types).

## Variables

```csharp

int x = 1; // type explicitly written on declaration
var y = 1 + x; // type inferred

```

```fsharp

let x : int = 1 // type explicitly written on declaration
let y = 1 + x // type inferred

```

## Re-assigning variables

```csharp
var z = "hello"
z = z + "world"
```

```fsharp
let mutable z = "hello"
z <- z + "world"
```

## Instantiating objects (calling constructors)

```csharp
var june24 = new DateTime(2017, 6, 24);
```

```fsharp
let june24 = new DateTime(2017, 6, 24)
// or...
let june24 = DateTime(2017, 6, 24)
```

In F#, the `new` keyword is optional unless the type being instantiated
implements `IDisposable`, in which case you will get a warning if you don't use
`new`. I suggest not using `new` in general. This way, the warning will
occasionally serve to inform you that a type is `IDisposable` where you may
otherwise not have realized it.


```fsharp
let exampleDisposableWarning = MemoryStream()
// compiler warning:
// It is recommended that objects supporting the IDisposable
// interface are created using the syntax 'new Type(args)',
// rather than 'Type(args)'
```

## Conditionals

```csharp

if (1 < 2)
{
    Console.WriteLine("This line executes if the condition is true.");
    Console.WriteLine("Reality seems to be working normally.");
} else
{
    Console.WriteLine("This code executes if the condition is false.");
    Console.WriteLine("The sky is falling.");
}
Console.WriteLine("This line executes either way.");

```

```fsharp

if 1 < 2 then
    Console.WriteLine("This line executes if the condition is true.")
    Console.WriteLine("Reality seems to be working normally.")
else
    Console.WriteLine("This code executes if the condition is false.")
    Console.WriteLine("The sky is falling.")
Console.WriteLine("This line executes either way.")
```

## Conditional expressions

```csharp

// In C# you use the ternary operator (?) to put a conditional in an expression
Console.WriteLine(1 + 1 == 2 ? "Sane" : "Donkey brains");

```

```fsharp
// In F#, if/then/else works as an expression, like most statements.
// Also note single = for comparison, since <- is used for reassignment.
Console.WriteLine(if 1 + 1 = 2 then "Sane" else "Donkey brains")
```

## Switches

```csharp
var x = int.Parse(Console.ReadLine());
switch (x)
{
    case 0:
        Console.WriteLine("foo");
        break;
    case 2:
        Console.WriteLine("bar");
        break;
    case 3:
    case 4:
    case 5:
        Console.WriteLine("baz");
        break;
    default:
        Console.WriteLine("qux");
        break;
}
```

```fsharp
let x = int.Parse(Console.ReadLine())
match x with
| 0 ->
    Console.WriteLine("foo")
| 2 ->
    Console.WriteLine("bar")
| 3
| 4
| 5 ->
    Console.WriteLine("baz")
| _ ->
    Console.WriteLine("qux")
```

Be aware that the `match` expression in F# can also do much more than this. For
switching on type (as is possible in C# >= 7.0), see the section on [casting and
conversion](Casting.md).

## While loops

```csharp

var reading = true;
while (reading)
{
    var line = Console.ReadLine();
    if (line == null)
    {
        reading = false;
    }
    else
    {
        Console.WriteLine(line);
    }
}

```

```fsharp
let mutable reading = true
while reading do
    let line = Console.ReadLine()
    if line = null then
        reading <- false
    else
        Console.WriteLine(line)
```

## Iterating over an integer range

```csharp

for (int i = 0; i <= 10; i++)
{
    Console.WriteLine(i);
}

```

```fsharp
for i = 0 to 10 do
    Console.WriteLine(i)
```

## Iterating over a collection

```csharp

var myArray = new[] { "abc", "123" };
foreach (var element in myArray)
{
    Console.WriteLine("It's as easy as " + element);
}

```

```fsharp
let myArray = [| "abc"; "123" |]
for element in myArray do
    Console.WriteLine("It's as easy as " + element)
```

## Lambda functions

```csharp

var countingByTwo = new[] { 1, 2, 3 }.Select(x => x * 2);

```

```fsharp
let countingByTwo = [| 1; 2; 3 |].Select(fun x -> x * 2)
```

## Local functions

```csharp

public static class FizzBuzzer
{
    public static void FizzBuzz(string fizz, string buzz)
    {
        string representation(int i)
        {
            string str = null;
            if (i % 3 == 0) str += fizz;
            if (i % 5 == 0) str += buzz;
            if (str == null) str = i.ToString();
        }
        for (var i = 0; i <= 100; i++)
        {
            Console.WriteLine(representation(i));
        }
    }
}

```

```fsharp
type FizzBuzzer =
    static member FizzBuzz(fizz, buzz) =
        let representation i =
            match i % 3, i % 5 with
            | 0, 0 -> fizz + buzz
            | 0, _ -> fizz
            | _, 0 -> buzz
            | _, _ -> i.ToString()
        for i = 0 to 100 do
            Console.WriteLine(representation(i))
```

## default(T) (generic default value)

```csharp
var x = default(T);
```

In F#, this is under the `Unchecked` module, because it is a backdoor way to get
a null value of an F# reference type that is not supposed to contain nulls.
Therefore, use it with caution!

```fsharp
let x = Unchecked.defaultof<'a>
```

## typeof (getting a System.Type object from a compile-time type name)

```csharp
var specificDictionaryType = typeof(Dictionary<string, int>);
var genericDictionaryType = typeof(Dictionary<,>);
```

```fsharp
let specificDictionaryType = typeof<Dictionary<string, int>>

// note: typeDEFof<>, not typeof<>
let genericDictionaryType = typedefof<Dictionary<_, _>>
```

## Calling methods with ref parameters

```csharp
var x = 1;
var y = Interlocked.Increment(ref x);
```

```fsharp
let mutable x = 1
let y = Interlocked.Increment(&x)
```

## Calling methods with out parameters

```csharp
// long way
DateTime myDateTime1;
var success1 = DateTime.TryParse("2017-06-24", out myDateTime1);

// or short way
var success2 = DateTime.TryParse("2017-06-24", out var myDateTime2);
```

```fsharp
// long way
let mutable myDateTime1 = Unchecked.defaultof<DateTime>
let success1 = DateTime.TryParse("2017-06-24", &myDateTime1)

// or short way
let success2, myDateTime2 = DateTime.TryParse("2017-06-24")
```

