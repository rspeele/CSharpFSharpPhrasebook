# C# to F# Phrasebook

This is a list of code samples showing the translation from C# to F# code.

The idea is that if you can easily do the same stuff you already know from C#,
that gives you some solid ground to stand on for learning the features that
_don't_ exist in C#.

Emphasis has been placed on making the translations as literal as possible. This
means that I value making these samples 1-to-1 equivalent with the C# code they
replace more than I value making them idiomatic, functional-style F# code. I
figure that you can learn idioms _after_ getting over the frustration of not
being able to express something you're accustomed to writing in C#.

The samples are currently split into three sections:

* [The code section](Code.md) shows how to translate the code you
  write inside methods: variable assignments, loops, conditionals, etc.

* [The types section](Types.md) shows how to translate the code you
  write to define classes, interfaces, enums, generics, etc.

* [The casting and conversion section](Casting.md) shows how to
  translate C# code that casts, converts, boxes, and unboxes objects. F#
  includes a slew of casting and conversion operators, and it is important to
  see how they differ (and sometimes, how they are identical).

