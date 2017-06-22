# Variables

```csharp

int x = 1;
var y = 1 + x; // type inferred to be int

```

```fsharp

let x = 1
let y = 1 + x

```

# Re-assigning variables

```csharp
var z = "hello"
z = z + "world"
```

```fsharp
let mutable z = "hello"
z <- z + "world"
```

# Conditionals

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

# Conditional expressions

```csharp

// In C# you use the ternary operator (?) to put a conditional in an expression
Console.WriteLine(1 + 1 == 2 ? "Sane" : "Donkey brains");

```

```fsharp
// In F#, if/then/else works as an expression, like most statements.
// Also note single = for comparison, since <- is used for reassignment.
Console.WriteLine(if 1 + 1 = 2 then "Sane" else "Donkey brains")
```

# While loops

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

# Iterating over an integer range

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

# Iterating over a collection

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

# Lambda functions

```csharp

var countingByTwo = new[] { 1, 2, 3 }.Select(x => x * 2);

```

```fsharp
let countingByTwo = [| 1; 2; 3 |].Select(fun x -> x * 2)
```

# Local functions

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
        let representation(i) =
            match i % 3, i % 5 with
            | 0, 0 -> fizz + buzz
            | 0, _ -> fizz
            | _, 0 -> buzz
            | _, _ -> i.ToString()
        for i = 0 to 100 do
            Console.WriteLine(representation(i))
```