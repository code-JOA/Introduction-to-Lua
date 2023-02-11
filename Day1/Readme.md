# Day 1: The Call to Adventure
When we first built our system, we found writing configuration files burdensome. The file formats we could use to describe test inputs and outputs just weren’t that expressive.

Take comma-separated values (CSV), for instance. Say you wanted to describe characters and vehicles for a video game. Your CSV configuration file might start like this:

​ 	
name,   treasure1, treasure2, treasure3, treasure4, treasure5
​ 	
knight, -1000,     +200,      --,        --,        --
Now, say you wanted to add a square vehicle to the in-game world, and make it so you can’t accidentally change the width without also changing the height:

​ 	
name,      width,      height
​ 	
mine cart, $cube_size, $cube_size
Here’s the problem: CSV doesn’t support collections or constraints. You’d have to work around these limitations by adding extra columns you seldom use, or by rolling (and supporting!) your own dialect.

Now, let’s see what the same configuration might look like in Lua:

​ 	
Monster{
​ 	
   name     = ​"knight"​,
​ 	
   treasure = {-1000, 200}
​ 	
}
​ 	
​ 	
​local​ cube_size = 20
​ 	
​ 	
Vehicle{
​ 	
   name     = ​"mine cart"​,
​ 	
   width    = cube_size,
​ 	
   height   = cube_size
​ 	
}
Beautiful. In one sweeping move, we’ve solved both CSV’s awkward collection syntax and its inability to handle constraints—by switching to a language that was designed precisely to implement these features.

You can even bring your monsters to life right in your level design.

​ 	
Monster{
​ 	
   name  = ​"cobra"​,
​ 	
   speed = ​function​() ​return​ 10 * damage_to_player() ​end​
​ 	
}
In this case, we’ve added some custom behavior to one particular adversary, right alongside the rest of its attributes.

The Week Ahead
On Day 1, we’re going to install Lua and find our way around. You’ll learn the basic data types and write a few simple Lua programs.

The second day, we’ll dive into the key idea that makes Lua so expressive: tables. These are a sort of array-meets-dictionary object that you can use to implement everything from smart configuration files to a homemade object system. We’ll also look at Lua’s powerful concurrency features.

To cap off our adventure, on Day 3 we’ll use Lua for its intended purpose: an expressive description language used together with fast, low-level C code. Specifically, we’re going to write a music player that takes descriptions of notes and chords, then plays them live on your computer.

First things first, though. Today, we’ll learn a little about Lua as a language. Then, we’ll write some simple Lua programs with numbers, strings, Booleans, functions, and conditionals. These constructs will likely feel familiar to you, but Lua presents them in a particularly approachable way.

Lua at a Glance
Lua is a table-based programming language, built on a single powerful abstraction that you can use to implement your own programming style—procedural, object-oriented, event-driven, and so on.

Lua’s tables lend themselves really well to the prototype style of object-oriented programming. In this style, classes and instances aren’t separate concepts. You don’t create a set of blueprints (classes) and then spin up a bunch of individual objects based on those blueprints.

Instead, in a prototype system, you create a single instance that looks like the objects you need in your program. Then, you clone this one instance a bunch of times and customize each clone. These systems are just as powerful as traditional class-based systems but have a simpler feel.

Installing Lua
By design, Lua is extremely portable. Its authors stick to a strict subset of ANSI C known to work across compilers and platforms. They were so successful in their dedication that Lua was one of only two scripting languages I could get to compile for a particularly limited embedded platform. (The other language was REXX, if you’re curious.)[2]

One of the most fun ways to install Lua is simply to compile it yourself from the source.[3] If you’re in a hurry, you can download a prebuilt binary for one of nearly a dozen platforms.[4]

Interactive Development
Like many of its fellow scripting languages, Lua supports an interactive read–eval–print loop (REPL). To start it, just type lua at the command line:

​ 	
$ ​lua​
​ 	
Lua 5.2.3  Copyright (C) 1994-2013 Lua.org, PUC-Rio
​ 	
>
Notice we’re using Lua 5.2.3, the latest version as of this writing. Much of this code will work fine in older versions, but we tested most extensively with 5.2.

We’ll stay in the REPL for much of this chapter. The bits after the leading > or >> are for you to type in.

Go ahead and type a value into the REPL, such as the year one of my favorite adventure movies was made:

​ 	
>​ 1989​
​ 	
stdin:1: unexpected symbol near '1989'
Interesting. Lua doesn’t print the value by default. We can deal with that easily enough. You could print() or return the value explicitly, or just add an =, like this:

​ 	
>​ print(1989)​
​ 	
1989
​ 	
>​ return 1989​
​ 	
1989
​ 	
>​ =1989​
​ 	
1989
If you wanted to get out of the REPL, you’d just type Ctrl-D. But stick around, so we can kick the tires a bit first.

First Glimpse
Lua has a friendly, approachable syntax. There’s no need to fuss over semicolons or where the whitespace goes. In fact, whitespace doesn’t matter much in Lua at all. Both of the following statements have the same output:

​ 	
>​ print "No time for love"​
​ 	
No time for love
​ 	
>​ print​
​ 	
>>​ "No time for love"​
​ 	
No time for love
You don’t even need to place line breaks between statements:

​ 	
>​ print "No time" print "for love"​
​ 	
No time
​ 	
for love
Lua’s types are similarly easy to use.

Building Blocks
Like most scripting languages, Lua is dynamically typed, meaning that while variables in a program don’t have types, runtime values do. Lua has the usual basic types you’d expect: numbers, Booleans, and strings:

​ 	
>​ =3.14​
​ 	
3.14
​ 	
>​ =true​
​ 	
true
​ 	
>​ ="The dog's name was 'Indiana!'"​
​ 	
The dog's name was 'Indiana!'
Wondering about integers? Nope, Lua doesn’t have them. In a typical Lua installation, 64-bit floating-point numbers are the only choice, just like JavaScript. (One minor exception: For embedded platforms with no floating-point numbers, you can rebuild Lua from source to use integers instead.)

Strings can be enclosed either in single or double quotes. Backslashes allow you to escape special or unprintable characters:

​ 	
>​ ='Separated\tby\t\ttabs'​
​ 	
Separated       by              tabs
You concatenate strings with the .. operator:

​ 	
>​ ='fortune' .. ' and ' .. 'glory'​
​ 	
fortune and glory
You take the length of a string with #:

​ 	
>​ =#'professor'​
​ 	
9
nil is its own type in Lua, representing “not found” or “does not exist.” (It’s also useful for deleting items from collections, as you’ll see on Day 2.)

​ 	
>​ =some_variable_that_does_not_exist​
​ 	
nil
Now that you’ve seen the fundamental building blocks of Lua data, let’s put them together into expressions.

# Expressions
Arithmetic in Lua looks like math in just about any language. As you’d expect, multiplication and division take precedence over addition and subtraction. You can group operations with parentheses.

​ 	
>​ =6 + 5 * 4 - 3 / 2​
​ 	
24.5
​ 	
>​ =6 + (5 * 4) - (3 / 2)​
​ 	
24.5
​ 	
>​ =(6 + 5) * (4 - 3) / 2​
​ 	
5.5
Lua has built-in operators for exponentiation (^) and modulo arithmetic (%):

​ 	
>​ =1899 % 100​
​ 	
99
​ 	
>​ = 2 ^ 3​
​ 	
8
Instead of Boolean operators, Lua uses the and, or, and not keywords. Conveniently, logical expressions short-circuit, meaning that Lua evaluates both halves of an expression only if it needs to.

​ 	
>​ =not ((true or false) and false)​
​ 	
true
​ 	
>​ =true or spill_antidote()​
​ 	
true
(No antidotes were spilled in the running of this code.)

You can compare any values for equality and inequality with == and ~=, respectively. The relative comparisons <, <=, >, and >= are usable only with strings and numbers:

​ 	
>​ ='cobras' < 'rats'​
​ 	
true
​ 	
>​ =#'cobras' < #'rats'​
​ 	
false
​ 	
>​ =42 < '43'​
​ 	
stdin:1: attempt to compare number with string
​ 	
...​​
​ 	
>​ =true < false​
​ 	
stdin:1: attempt to compare two boolean values
We have got a handle on data types and expressions. Let’s breathe some life into them with a few functions.

Functions
Lua function definitions look like those in any common scripting language:

​ 	
>​  function triple(num)​
​ 	
>>​     return 3 * num​
​ 	
>>​ end​
​ 	
>​​
​ 	
>​ =triple(2)​
​ 	
6
Strictly speaking, the function name isn’t necessary; you could just as easily type the following:

​ 	
>​ =(function(num) return 3 * num end)(2)​
​ 	
6
In Lua, functions are first-class values; they can be treated just like any other value in Lua. In particular, they can be assigned to variables, passed as parameters into other functions, and stored in data structures.

For example, you could easily write a function call_twice() that takes a second function f() and returns a third function ff that calls f twice:

​ 	
>​​
​ 	
>​ function call_twice(f)​
​ 	
>>​     ff = function(num)​
​ 	
>>​         return f(f(num))​
​ 	
>>​     end​
​ 	
>>​     return ff​
​ 	
>>​ end​
​ 	
>​​
​ 	
>​ function triple(n)​
​ 	
>>​    return n * 3​
​ 	
>>​ end​
​ 	
>​​
​ 	
>​ times_nine = call_twice(triple)​
​ 	
>​​
​ 	
>​ =times_nine(5)​
​ 	
45
The ability to treat code as data is crucial for Lua’s power and compactness. We’ll see more examples of these techniques later on.

Tail Calls
One other nice programmer convenience for functions in Lua is tail call optimization. This comes into play when you have a recursive function whose recursive call is the very last thing it does:

​ 	
​function​ reverse(s, t)
​ 	
    ​if​ #s < 1 ​then​ ​return​ t ​end​
​ 	
    first = string.​sub​(s, 1, 1)
​ 	
    rest  = string.​sub​(s, 2, -1)
​ 	
    ​return​ reverse(rest, first .. t)
​ 	
​end​
​ 	
​ 	
large = string.​rep​(​'h​ello ​',​ 5000)
​ 	
print(reverse(large, ​''​))
Many scripting language implementations would choke on that call; for instance, a JavaScript version fails with a stack error in the current release version of Google Chrome. Lua, however, correctly optimizes the recursive call into a simple goto and completes the calculation.

Flexible Arguments
What happens when you try to call a function with too few arguments? Some languages shut you down with an error message. Others, like Haskell, return a new function. Lua simply assigns a value of nil to all the unused parameters:

​ 	
>​ function print_characters(friend, foe)​
​ 	
>>​     print('*Friend and foe*')​
​ 	
>>​     print(friend)​
​ 	
>>​     print(foe)​
​ 	
>>​ end​
​ 	
>​ print_characters('Marcus', 'Belloq')​
​ 	
*Friend and foe*
​ 	
Marcus
​ 	
Belloq
​ 	
>​ print_characters('Marcus')​
​ 	
*Friend and foe*
​ 	
Marcus
​ 	
nil
Any extra parameters are just ignored:

​ 	
>​ print_characters('Marcus', 'Belloq', 'unused')​
​ 	
*Friend and foe*
​ 	
Marcus
​ 	
Belloq
You can also explicitly create variadic functions, that is, functions with an arbitrary number of inputs. You do so by making the last parameter in the function declaration an ellipsis (...):

​ 	
>​ function print_characters(friend, ...)​
​ 	
>>​     print('*Friend*')​
​ 	
>>​     print(friend)​
​ 	
>>​​
​ 	
>>​     print('*Foes*')​
​ 	
>>​     foes = {...}​
​ 	
>>​     print(foes[1])​
​ 	
>>​     print(foes[2])​
​ 	
>>​ end​
​ 	
>​​
​ 	
>​ print_characters('Marcus', 'Belloq')​
​ 	
*Friend*
​ 	
Marcus
​ 	
*Foes*
​ 	
Belloq
​ 	
nil
​ 	
​ 	
>​ print_characters('Marcus', 'Belloq', 'Donovan')​
​ 	
*Friend*
​ 	
Marcus
​ 	
*Foes*
​ 	
Belloq
​ 	
Donovan
We assign the entire list of arguments to the foes table, which we’re treating like a simple array here. That’s one of the unique features of Lua’s tables that we’ll see on Day 2.

Multiple Return Values
By the same token, you can also return multiple values from a function and either use them or ignore them:

​ 	
>​ function weapons()​
​ 	
>>​     return 'bullwhip', 'revolver'​
​ 	
>>​ end​
​ 	
>​​
​ 	
>​ w1 = weapons()​
​ 	
>​ print(w1)​
​ 	
bullwhip
​ 	
>​​
​ 	
>​ w1, w2 = weapons()​
​ 	
>​ print(w1)​
​ 	
bullwhip
​ 	
>​ print(w2)​
​ 	
revolver
​ 	
>​​
​ 	
>​ w1, w2, w3 = weapons()​
​ 	
>​ print(w1)​
​ 	
bullwhip
​ 	
>​ print(w2)​
​ 	
revolver
​ 	
>​ print(w3)​
​ 	
nil
The rules are the same as for parameters: unused values are ignored, and unused variables are nil.

# Keyword Arguments
Lua doesn’t have a special syntax for keyword arguments like those in Python or Ruby.[5] But you can get the same effect by passing a table as a function argument:

​ 	
>​ function popcorn_prices(table)​
​ 	
>>​     print('A medium popcorn costs ' .. table.medium)​
​ 	
>>​ end​
​ 	
>​​
​ 	
>​ popcorn_prices{small=5.00,​
​ 	
>>​               medium=7.00,​
​ 	
>>​               jumbo=15.00}​
​ 	
A medium popcorn costs 7
In this example, the table is the set of size names and prices between the curly braces (with no surrounding parentheses—Lua lets us leave them out in this case). The function reads a specific value from the table with a dotted notation: table.medium.

You can build quite a lot with just functions; a whole programming language, even! But for convenience’s sake, let’s look at some control structures.

Control Flow
Lua’s built-in control flow constructs are the if statement, two flavors of for loop, and while loops.

The if statement may have an else clause and zero or more elseifs. Unlike some scripting languages, Lua’s if doesn’t return a value; you’ll need to store results in a variable or print them:

​ 	
>​ film = 'Skull'​
​ 	
>​​
​ 	
>​  if film == 'Raiders' then​
​ 	
>>​     print('Good')​
​ 	
>>​ elseif film == 'Temple' then​
​ 	
>>​     print('Meh')​
​ 	
>>​ elseif film == 'Crusade' then​
​ 	
>>​     print('Great')​
​ 	
>>​ else​
​ 	
>>​     print('Huh?')​
​ 	
>>​ end​
​ 	
Huh?
for loops work over a series of numbers (with an optional step argument):

​ 	
>​  for i = 1, 5 do​
​ 	
>>​     print(i)​
​ 	
>>​ end​
​ 	
1
​ 	
2
​ 	
3
​ 	
4
​ 	
5
​ 	
>​  for i = 1, 5, 2 do​
​ 	
>>​     print(i)​
​ 	
>>​ end​
​ 	
1
​ 	
3
​ 	
5
You can also use for to loop over items in a collection, but we won’t get to that until we talk about tables later on.

The final built-in control construct in Lua is the while loop (and its cousin, the repeat loop, which you’ll learn more about during the exercises):

​ 	
>​  while math.random(100) < 50 do​
​ 	
>>​     print('Tails; flipping again')​
​ 	
>>​ end​
​ 	
Tails; flipping again
​ 	
Tails; flipping again
Lua doesn’t limit you to just the “big three” control structures of if, for, and while. If you combine them with the ability to pass functions around like data, you can build whatever control structures your program needs. In the exercises for Day 1, you’ll do just that.

# Variables
We’ve seen variables already in some of the examples today, but until now we’ve glossed over how they work. Let’s take a closer look.

One quirk of Lua is that variables are global by default:

​ 	
>​  function hypotenuse(a, b)​
​ 	
>>​     a2 = a * a​
​ 	
>>​     b2 = b * b​
​ 	
>>​     return math.sqrt(a2 + b2)​
​ 	
>>​ end​
​ 	
>​​
​ 	
>​ =hypotenuse(3, 4)​
​ 	
5
​ 	
>​ =a2​
​ 	
9 -- WHOOPS!
You’d probably prefer that our temporary a2 variable not leak outside the function. Fortunately, all we have to do is preface our local variable definitions with the local keyword:

​ 	
>​  function hypotenuse(a, b)​
​ 	
>>​     local a2 = a * a​
​ 	
>>​     local b2 = b * b​
​ 	
>>​     return math.sqrt(a2 + b2)​
​ 	
>>​ end​
I was initially surprised that local isn’t the default in Lua. But it turns out that there are good reasons for this.[6] If we really want to forbid creating globals accidentally, Lua’s tables offer a way; we’ll do something very close to this on Day 2.

Leaving Behind the REPL
So far, we’ve been typing all these expressions into the REPL. This is the best way to learn Lua, and it makes it easy to build up a program while you’re typing it.

However, in a minute I’m going to invite you to do some exercises. You may want to work on these in a text editor and then run them from the command line. To do so, just save your program with a .lua extension and then run it with the same lua command you used to launch the REPL, like so:

​ 	
lua my_program.lua
While it’s not as interactive as the REPL, saving to a file makes it easier to correct typos when you’re striking out on your own.

What We Learned in Day 1
Today, you got to know the basics of Lua syntax. You saw how easy it is to define functions, including fancy higher-order functions that take other functions as input. You now know enough Lua to write a few simple programs, and in a moment you will.

At this point, you’re probably thinking Lua is an easy-to-use scripting language, but with nothing particular to make it stand out in a crowd. That was certainly my first reaction when I encountered the language.

Then I ran into Lua’s killer feature that makes its expressiveness possible: tables. On Day 2, you’ll see what’s so special about them.

Your Turn
Find…
The Lua wiki, which supplements the built-in docs with community-maintained explanations and examples

The online version of Programming in Lua, First Edition (the newer paid editions are good too, but this one is both helpful and free)

The latest version of the Lua reference manual

The difference between a while loop and a repeat loop

Do (Easy):
Write a function called ends_in_3(num) that returns true if the final digit of num is 3, and false otherwise.

Now, write a similar function called is_prime(num) to test if a number is prime (that is, it’s divisible only by itself and 1).

Create a program to print the first n prime numbers that end in 3.

Do (Medium):
What if Lua didn’t have a for loop? Using if and while, write a function for_loop(a, b, f) that calls f() on each integer from a to b (inclusive).

Do (Hard):
Write a function reduce(max, init, f) that calls a function f() over the integers from 1 to max like so:

​ 	
​function​ add(previous, next)
​ 	
    ​return​ previous + next
​ 	
​end​
​ 	
​ 	
reduce(5, 0, add) ​-- add the numbers from 1 to 5​
​ 	
​ 	
​-- We want reduce() to call add() 5 times with each intermediate​
​ 	
​-- result, and return the final value of 15:​
​ 	
​--​
​ 	
add( 0, 1) ​--> returns 1; feed this into the next call​
​ 	
add( 1, 2) ​--> returns 3​
​ 	
add( 3, 3) ​--> returns 6​
​ 	
add( 6, 4) ​--> returns 10​
​ 	
add(10, 5) ​--> returns 15​
Implement factorial() in terms of reduce().

```python

```
