<!-- #region -->
Day 2: Tables All the Way Down
Today, we’re going to look at two concepts that define the Lua experience: tables and coroutines. As with many prototype languages, tables define your data. Coroutines define your control flow. Both are simple but tremendously powerful, underpinning everything from Lua’s object system to your own domain-specific languages.

Let’s begin with tables.

One of the first things new programming language tutorials do is inundate you with a laundry list of data structures: arrays, tuples, vectors, lists, dictionaries, and so on. Each of these has its own API, syntax, quirks, and performance characteristics.

These collections are all useful, but when I’m first trying out a language, I’m usually wondering about much more basic things:

Where do I keep things when I need to access them by name?

Where do I store values in a particular order?

Lua answers both of these questions with one single data structure: the table.

Tables As Dictionaries
Like Python’s dictionaries or Ruby’s hashes, Lua’s tables are collections of keys (names) with associated values. You create a table with curly braces, an expression known in Lua as a table constructor:

​ 	
>​ book = {​
​ 	
>>​    title  = "Grail Diary",​
​ 	
>>​    author = "Henry Jones",​
​ 	
>>​    pages  = 100​
​ 	
>>​ }​
To get data back out of the table, you just write the table name, a dot, and the key you want to read:

​ 	
>​ =book.title​
​ 	
Grail Diary
Using the same dot notation, you can add or modify items:

​ 	
>​ book.stars  = 5                 -- new item​
​ 	
>​ book.author = "Henry Jones Sr." -- modified item​
What about keys with spaces or decimal points in them? Or keys you calculate at runtime? For these cases, you put the key in square brackets:

​ 	
>​ key = "title"​
​ 	
>​ =book[key]​
​ 	
Grail Diary
You can actually use any data type as a table key with this syntax: Booleans, functions, and even other tables. Most of the time, though, you’ll encounter string and number keys.

To remove an item from a table, just set its key to the special value nil:

​ 	
>​ book.pages = nil​
Lua doesn’t ship with a function to print the contents of a table. Fortunately, we can define a simple one that will work for these first few examples. With the REPL still running, switch to your editor and save the following code in a file called util.lua:

lua/day2/util.lua
​ 	
​function​ print_table(t)
*
   ​for​ k, v ​in​ pairs(t) ​do​
​ 	
      print(k .. ​": "​ .. v)
​ 	
   ​end​
​ 	
​end​
pairs() is a built-in Lua function. More specifically, it’s an iterator, which is a function designed to plug seamlessly into a for loop. For the gory details on how to build one, see the relevant chapter in the online Lua book.[7] The gist is that pairs() returns a new function, which the for loop calls over and over until it returns nil.

Our print_tables function won’t correctly handle nested tables or indeed much of anything beyond the basics. But it’ll do for now. You can bring it into the REPL by using the dofile() function:

​ 	
>​ dofile('util.lua')​
As an alternative, you can launch Lua with your library preloaded by using the -l option like so:

​ 	
$ ​lua -l util​
dofile() is a blunt instrument that just slurps up the file you give it. It doesn’t check to see if the code is already loaded, and it doesn’t let you customize where Lua looks for files. Later, we’ll use Lua’s module system, which does both of these and more.

Here’s the output of print_tables() on the book table we defined earlier:

​ 	
>​ print_table(book)​
​ 	
author: Henry Jones
​ 	
title: Grail Diary
​ 	
pages: 100
So far, the table seems like an ordinary dictionary type. But keys and values aren’t the only trick up its sleeve.

A Dictionary in Array’s Clothing
Sometimes, you need to store data in a specific order. Other languages give you lists or arrays for this purpose, with a separate syntax and API from dictionaries.

In Lua, there’s no need for a second abstraction. Lua views arrays as just a special case of key-value storage, where the keys are sequential numbers. You use the same syntax to create an array as you did before; just leave out the keys:

​ 	
>​ medals = {​
​ 	
>>​    "gold",​
​ 	
>>​    "silver",​
​ 	
>>​    "bronze"​
​ 	
>>​ }​
You read and write array contents using a familiar square-bracket notation:

​ 	
>​ =medals[1]​
​ 	
gold
​ 	
>​ medals[4] = "lead"​
Notice that, like mathematicians and civilians, Lua counts array indices starting at 1.

At this point, you’re probably wondering, “How can Lua arrays possibly be efficient?” In most languages, dictionaries are slower than arrays; hashing a string takes a lot longer than just incrementing a pointer.

Fortunately, the Lua runtime provides a special fast track for arrays.[8] As long as you’re adding values consecutively and using numeric keys, Lua will store and access the data efficiently.

Arrays and dictionaries aren’t mutually exclusive in Lua. You can mix both styles in the same table, and Lua will figure out how to store everything efficiently. Some programmers adopt the convention of separating the array and dictionary parts with a semicolon:

​ 	
>​ ice_cream_scoops = {​
​ 	
>>​    "vanilla",​
​ 	
>>​    "chocolate";​
​ 	
>>​​
​ 	
>>​    sprinkles = true​
​ 	
>>​ }​
​ 	
>​​
​ 	
>​ =ice_cream_scoops[1]​
​ 	
vanilla
​ 	
>​ =ice_cream_scoops.sprinkles​
​ 	
true
Storing items by name or number is a nice parlor trick, but what if you need something like custom lookup logic? For that, we turn to Lua’s metatables.

Metatables
In all the tables we’ve seen so far, you pass in a key, and Lua retrieves a value for you. This lookup logic is built into Lua.

Sometimes, this default behavior isn’t what your program needs. Say, for example, you want to supply a default value other than nil for unrecognized keys. Or perhaps you need to log all reads/writes to a particular table. You can implement both of these behaviors using a data structure known as a metatable.

The name metatable—the “table behind the table”—sounds a bit abstract, but if you’ve used JavaScript’s prototypes or Python’s special double-underscore method names, you’ll find Lua’s approach familiar.[9][10] Just think of it as “custom behavior” for now.

Every table in Lua has a corresponding metatable, containing functions for reading/writing keys, iterating contents, and overloading operators. Most tables have their metatable set to nil, which punts table operations to Lua:

​ 	
>​ greek_numbers = {​
​ 	
>>​ ena  = "one",​
​ 	
>>​ dyo  = "two",​
​ 	
>>​ tria = "three"​
​ 	
>>​ }​
​ 	
>​​
​ 	
>​ =getmetatable(greek_numbers)​
​ 	
nil
But you can easily override Lua’s default behavior. The way Lua prints tables to strings is about as useful as a bullwhip against a loaded M1917.

For instance, the way Lua prints tables to standard output is a little terse:

​ 	
>​ =greek_numbers​
​ 	
table: 0x7fec0ad002b0
It’d be nice if we could actually see the keys and values, without needing to call a separate function. Fortunately, we can.

All we have to do is create a metatable, and store a function inside it under the name of __tostring. Lua will call this function whenever someone tries to display our table. We can use a slight variation on the print_table() function we wrote earlier, where we return the contents as a string instead of printing them to the console.

Add the following code to your util.lua:

lua/day2/util.lua
​ 	
​function​ table_to_string(t)
​ 	
   ​local​ result = {}
​ 	
​ 	
   ​for​ k, v ​in​ pairs(t) ​do​
​ 	
      result[#result + 1] = k .. ​": "​ .. v
​ 	
   ​end​
​ 	
​ 	
   ​return​ table.​concat​(result, ​"\n"​)
​ 	
​end​
This new function just stores each key-value pair in a list, then concatenates them into a string at the end. With larger tables, this approach is faster than building a single string item by item.

Now, reload util.lua in the REPL:

​ 	
>​ dofile('util.lua')​
Finally, we can connect our custom output logic:

​ 	
>​  mt = {​
​ 	
>>​     __tostring = table_to_string​
​ 	
>>​ }​
​ 	
>​​
​ 	
>​ setmetatable(greek_numbers, mt)​
​ 	
>​​
​ 	
>​ =greek_numbers​
​ 	
ena: one
​ 	
tria: three
​ 	
dyo: two
We’ve changed the default behavior for printing tables to strings. Now, when we call print(), 
Lua will look for __tostring in the metatable and find this function. The end result is a much more descriptive output.

Now that we’ve dipped our toes in with an easy function, let’s look at a more complicated case.

Reading and Writing
By design, Lua’s tables are pretty forgiving. If you try to read a key that’s not in the table, nothing bad happens; you just get nil back. Say you wanted to create a stricter table, where reading a nonexistent key, or overwriting an existing key, caused a runtime error.

This task takes just a few easy steps in Lua:

Write a pair of functions to implement the custom read/write behavior you want to see.

Store these functions in a table under the names __index and __newindex.

Set this table as your data’s metatable.

Let’s look at step 3 first. Place the following code into strict.lua:

lua/day2/strict.lua
​ 	
​local​ mt = {
​ 	
   __index    = strict_read,
​ 	
   __newindex = strict_write
​ 	
}
​ 	
​ 	
treasure = {}
​ 	
setmetatable(treasure, mt)
The strict_read() function will read the underlying data from a private table that we’ll never access directly. Add the following code to the top of the file:

lua/day2/strict.lua
​ 	
​local​ _private = {}
​ 	
​ 	
​function​ strict_read(table, key)
​ 	
   ​if​ _private[key] ​then​
​ 	
      ​return​ _private[key]
​ 	
   ​else​
​ 	
      error(​"Invalid key: "​ .. key)
​ 	
   ​end​
​ 	
​end​
Lua will pass the table and key we’re reading into our lookup function; all we have to do is return the underlying data, if it exists.

The strict_write() function is similar; it needs to check the private table to see if the key’s already there. This definition goes right after the one for strict_read():

lua/day2/strict.lua
​ 	
​function​ strict_write(table, key, value)
​ 	
   ​if​ _private[key] ​then​
​ 	
      error(​"Duplicate key: "​ .. key)
​ 	
   ​else​
​ 	
      _private[key] = value
​ 	
   ​end​
​ 	
​end​
Load your strict.lua file into the REPL using dofile() or the -l option. Then, try stashing some treasure in your treasure chest:

​ 	
>​ treasure.gold = 50​
​ 	
>​​
​ 	
>​ =treasure.gold​
​ 	
50
​ 	
>​​
​ 	
>​ =treasure.silver​
​ 	
strict.lua:8: Invalid key: silver
​ 	
...​​
​ 	
>​​
​ 	
>​ treasure.gold = 100​
​ 	
strict.lua:16: Duplicate key: gold
​ 	
...
So far, we’ve used metatables for custom lookup logic, and custom output formats. You can also use them to overload arithmetic, logical, and comparison operators. The process is exactly the same: you store functions in a table with special key names like __add or __sub, then call setmetatable() to bind the custom behavior to your data.

In the next section, you’ll see just how powerful metatables can be: we’re going to build our own object-oriented system on top of Lua’s primitives.

Roll Your Own OO
Lua comes with its own syntax for object-oriented programming. But I’m going to show you how easy it is to roll your own OO scheme using the powerful abstractions built into the language. Afterward, we’ll see what a short step it is from our homegrown solution to a typical Lua one.

The kernel of an object-oriented program is the idea of autonomous objects sending one another messages. You can implement this idea in Lua with what you’ve already seen, using plain ol’ tables and functions.

Say you’re building a game and want your player to face a boss-level baddie during the final stage. The game engine responds to an attack on the villain by sending the take_hit() message.

Functions in Lua are just ordinary data values that can be stored and passed around. So, you can make take_hit() into a function and store it inside the bad guy’s table right alongside the state:

​ 	
dietrich = {
​ 	
   name     = ​"Dietrich"​,
​ 	
   health   = 100,
​ 	
​ 	
   take_hit = ​function​(self)
​ 	
      self.health = self.health - 10
​ 	
   ​end​
​ 	
}
​ 	
​ 	
dietrich.take_hit(dietrich)
​ 	
print(dietrich.health)     ​--> 90​
Presumably, the game has more than one villain. If many of them are going to be sharing an implementation of take_hit(), we need to know whose health we’re docking. That’s the purpose of the self parameter passed into the function. (We’ll see a way to hide that in a moment.)

Notice that there’s no distinct Villain class that we’re instantiating (yet)—just a table with some data inside it. If we want to make another villain, we’re going to have to initialize the fields ourselves, or copy them from another villain.

​ 	
clone = {
​ 	
   name     = dietrich.name,
​ 	
   health   = dietrich.health,
​ 	
   take_hit = dietrich.take_hit
​ 	
}
Assuming we made the clone of our opponent before damaging the original, we can see that they are indeed two distinct objects:

​ 	
print(clone.health) ​--> 100​
All that manual copying of fields is going to get tiresome. Let’s fix that next.

Prototypes
To set up the fields automatically each time we create a new villain, we’ll write a function. To keep things modular, we’ll store this and all villain-related functions in a Villain table. Here’s a quick first pass, which we’ll soon see has some problems:

​ 	
Villain = {
​ 	
   health = 100,
​ 	
​ 	
   new = ​function​(self, name)
​ 	
      ​local​ obj = {
​ 	
         name   = name,
​ 	
         health = self.health,
​ 	
      }
​ 	
​ 	
      ​return​ obj
​ 	
   ​end​,
​ 	
​ 	
   take_hit = ​function​(self)
​ 	
      self.health = self.health - 10
​ 	
   ​end​
​ 	
}
​ 	
​ 	
dietrich = Villain.new(Villain, ​"Dietrich"​)
We now have a function that will spin up villains reliably for us. We don’t really need hundreds of copies of the same take_hit() function floating around, so we’ve moved it into the common Villain table. But now, we can’t use dietrich the way we used to:

​ 	
Villain.take_hit(dietrich)  ​--> ok​
​ 	
dietrich.take_hit(dietrich) ​--> error: attempt to call field​
​ 	
                            ​--> 'take_hit' (a nil value)​
dietrich no longer has a field called take_hit(). This behavior lives in the Villain object now. It’d be nice to use a prototype-based approach like JavaScript, where the object system looks in our prototype Villain object if it can’t find what it’s looking for in dietrich.

As we saw in ​Metatables​, we can implement any lookup behavior we like using Lua’s powerful metatables. Here’s the revised body of the new function:

​ 	
new = ​function​(self, name)
​ 	
   ​local​ obj = {
​ 	
      name   = name,
​ 	
      health = self.health,
​ 	
   }
​ 	
*
   setmetatable(obj, self)
*
   self.__index = self
​ 	
​ 	
   ​return​ obj
​ 	
​end​,
Those two extra lines delegate field lookup to the Villain prototype. You’ve probably noticed one key difference from how we used metatables earlier. Our previous metatables used special functions for custom behavior. Here, we’re using a table instead. This is Lua-speak for “use this table’s fields as backup.”

Now, take_hit() works the way we expect:

​ 	
dietrich = Villain.new(Villain, ​"Dietrich"​)
​ 	
dietrich.take_hit(dietrich) ​--> ok​
So far, we’ve just made copies of a single Villain object. How do we create different kinds of villains?

Inheritance
One nice thing about prototype-based object systems is that you don’t need a special mechanism for inheritance. You just clone objects the way you’ve been doing.

If, for instance, you want to start churning out supervillains who have more effective armor, all you have to do is create a single SuperVillain prototype and start cloning it:

​ 	
SuperVillain = Villain.new(Villain)
​ 	
​ 	
​function​ SuperVillain.take_hit(self)
​ 	
   ​-- Haha, armor!​
​ 	
   self.health = self.health - 5
​ 	
​end​
​ 	
​ 	
toht = SuperVillain.new(SuperVillain, ​"Toht"​)
​ 	
toht.take_hit(toht)
​ 	
print(toht.health) ​--> 95​
This is starting to look like a fully featured object system. Passing each object around twice is getting tiresome, though.

Syntactic Sugar
The final step to take us from the ground up to Lua’s object model is a bit of syntactic sugar. When you call table:method() instead of table.method(self), you can leave out the self parameter—Lua passes it in implicitly for you.

​ 	
Villain = { health = 100 }
​ 	
​ 	
​function​ Villain:new(name)
​ 	
   ​-- ...same implementation as before...​
​ 	
​end​
​ 	
​ 	
​function​ Villain:take_hit()
​ 	
   ​-- ...same implementation as before...​
​ 	
​end​
​ 	
​ 	
SuperVillain = Villain:new()
​ 	
​ 	
​function​ SuperVillain:take_hit()
​ 	
   ​-- ...same implementation as before...​
​ 	
​end​
Now, our homegrown object system looks a lot like what we’d see in any other language:

​ 	
dietrich = Villain:new(​"Dietrich"​)
​ 	
dietrich:take_hit()
​ 	
print(dietrich.health) ​--> 90​
​ 	
​ 	
toht = SuperVillain:new(​"Toht"​)
​ 	
toht:take_hit()
​ 	
print(toht.health)     ​--> 95​
So far today, we’ve started with a simple, flexible data structure and used it to build sophisticated constructs in just a few lines of code. Next, we’re going to do the same thing for control flow.

Coroutines
Everything we’ve asked Lua to do has fit in a sequence of one-at-a-time tasks. You may be wondering about how Lua handles multithreading.

It doesn’t.

Yes, you read that right. Lua does not come with a threading API. Instead, it ships with a simpler, easier-to-understand primitive for multitasking: the coroutine.

Coroutines have been around for a few decades. Like threads, they allow your program to have multiple tasks in progress. Unlike threads, coroutines aren’t preemptive. You have to add code to your program to point out explicitly when the current task can safely be paused so that another task can run.

Why should we use them, then, if they require juggling by the programmer? Because they’re conceptually simpler, and they eliminate a whole class of concurrency problems. When you look at a piece of code, you know exactly when it can or can’t be interrupted.

Coroutines vs. Threads
If coroutines are so great, why doesn’t every language use them? Like every programming language feature, coroutines involve trade-offs. What we gain in simplicity and correctness, we lose in multicore processing.

A single Lua process with many coroutines can only use one of your system’s cores at a time. However, it’s easy to spin up multiple Lua interpreters within a single process, and run these across multiple cores.[11]

Another thing to watch out for with coroutines is that they do not play well with blocking I/O calls. If your coroutine-based program uses the network, you’ll want to use the nonblocking select() function and friends.[12]

Single Tasks: Generators
Let’s start with a simple example. We’re going to create a coroutine and assign a task to it. The coroutine starts in the paused state, so right away we’ll resume() it to get it to start working. Inside the coroutine, we’ll add code to yield() after each piece of work is done.

First, let’s define a function that has a long task to perform—an infinitely long one, in fact.

​ 	
>​ function fibonacci()​
​ 	
>>​    local m = 1​
​ 	
>>​    local n = 1​
​ 	
>>​​
​ 	
>>​    while true do​
​ 	
>>​       coroutine.yield(m)​
​ 	
>>​       m, n = n, m + n​
​ 	
>>​    end​
​ 	
>>​ end​
This function never returns; instead, each time it computes a new Fibonacci number, it yields to the caller. What’s the difference between yielding and returning? When we yield, we can come back to the same spot in our program later.

To start this coroutine, we create it with the coroutine.create() function, then call it using coroutine.resume():

​ 	
>​ generator = coroutine.create(fibonacci)​
​ 	
>​ succeeded, value = coroutine.resume(generator)​
​ 	
>​ =value​
​ 	
1
At that moment, the program jumps inside fibonacci() and runs until it hits the yield(). Then, execution jumps back to the caller, right after the call to resume(). resume() returns a status flag, plus anything that was passed into yield().

Each time we call resume(), we pick up right where we left off the last time we yield()ed:

​ 	
>​ succeeded, value = coroutine.resume(generator)​
​ 	
>​ =value​
​ 	
1
​ 	
>​ succeeded, value = coroutine.resume(generator)​
​ 	
>​ =value​
​ 	
2
This approach is handy for any task that you want to break into chunks to keep your program responsive during a long computation or network operation.

Multitasking
Coroutines are simple, but they’re powerful enough to implement thread-like behavior. Operating system schedulers are tens of thousands of lines long, but you’re going to write one in a couple dozen![13]

What we want is to be able to define a couple of top-level functions that are going to appear to run concurrently:

​ 	
​function​ punch()
​ 	
   ​for​ i = 1, 5 ​do​
​ 	
      print(​'p​unch ​' ​.. i)
*
      scheduler.wait(1.0)
​ 	
   ​end​
​ 	
​end​
​ 	
​ 	
​ 	
​function​ block()
​ 	
   ​for​ i = 1, 3 ​do​
​ 	
      print(​'b​lock ​' ​.. i)
*
      scheduler.wait(2.0)
​ 	
   ​end​
​ 	
​end​
…and then schedule them to run like so:

​ 	
scheduler.schedule(0.0, ​coroutine​.​create​(punch))
​ 	
scheduler.schedule(0.0, ​coroutine​.​create​(block))
​ 	
​ 	
scheduler.run()
None of this thread-like API exists; we’re going to have to build it. The basic idea is to keep a list of everything we need to do in the future, sorted by when we need to do it.

Check out the syntax we’re using to call our scheduling functions. This is Lua’s module system, which (like everything else) is built on top of tables. We’ll use this system to define our API.

Place the following code in scheduler.lua:

lua/day2/scheduler.lua
​ 	
​local​ pending = {}
​ 	
​ 	
​local​ ​function​ schedule(time, action)
*
   pending[#pending + 1] = {
​ 	
      time   = time,
​ 	
      action = action
​ 	
   }
​ 	
​end​
Each time we want an action to happen in the future, we throw it into the pending array, which we’ll keep ordered by timestamp (number of seconds since the program started).

The hash symbol in #pending, by the way, is the length operator. You used it on Day 1 to get the length of a string. Here, you can see that it works on arrays as well. The highlighted line is a common Lua idiom for appending to an array.

One other thing to note here: to avoid name collisions, we’re making schedule and all other functions in this file local. Later on, we’ll specifically expose just the ones we want callers to see.

Remember that our coroutines are supposed to be lightweight. They need to get in, do their work, and either return or yield quickly. So our wait() function shouldn’t actually wait. Instead, it should yield back to the scheduler:

​ 	
​local​ ​function​ wait(seconds)
​ 	
   ​coroutine​.​yield​(seconds)
​ 	
​end​
The main run() loop busy-waits until it’s time to run the next task, then resume that task. If the task calls wait(), the number of seconds will get yielded back to us, which we use to schedule a future resumption of that task:

​ 	
​local​ ​function​ run()
​ 	
   ​while​ #pending > 0 ​do​
​ 	
      sort_by_time(pending)
​ 	
      ​while​ os.​clock​() < pending[1].time ​do​ ​end​ ​-- busy-wait​
​ 	
​ 	
      ​local​ item = remove_first(pending)
​ 	
      ​local​ _, seconds = ​coroutine​.​resume​(item.action)
​ 	
​ 	
      ​if​ seconds ​then​
​ 	
         later = os.​clock​() + seconds
​ 	
         schedule(later, item.action)
​ 	
      ​end​
​ 	
   ​end​
​ 	
​end​
When the work is complete, the coroutine won’t yield anything to us. The call to resume() will return nil, and we won’t schedule that task anymore.

The sort_by_time() routine just leans on Lua’s built-in table.sort() function, which takes an optional comparison function to compare the two entries in the array. Put this code at the top of scheduler.lua:

lua/day2/scheduler.lua
​ 	
​local​ ​function​ sort_by_time(array)
​ 	
   table.​sort​(array, ​function​(e1, e2)
​ 	
                        ​return​ e1.time < e2.time
​ 	
                     ​end​)
​ 	
​end​
The only function left to implement is remove_first(), which will delete and return the first item from the array. This is all stock Lua table manipulation:

​ 	
​local​ ​function​ remove_first(array)
​ 	
   result        = array[1]
​ 	
   array[1]      = array[#array]
​ 	
   array[#array] = nil
​ 	
   ​return​ result
​ 	
​end​
Let’s wrap this API up in a nice Lua module. This is just a plain ol’ table with the functions and names we want to make available to callers. The following code goes at the end of scheduler.lua:

​ 	
​return​ {
​ 	
   schedule = schedule,
​ 	
   run = run,
​ 	
   wait = wait
​ 	
}
And now to run our program! Create a new file called punch.lua with the following line at the top of it:

lua/day2/punch.lua
​ 	
scheduler = require ​'s​cheduler​'​
We use require() instead of dofile() to load our module. These are similar functions, but require() does more for you:

Checks to see if you’ve already loaded the module

Searches multiple (configurable) library paths

Safely namespaces the code in a local variable

Next, add the punch() and block() functions we saw at the beginning of this section, plus the setup calls to schedule() and run(). When you run lua punch.lua, you should see five punches flying by, with blocks happening about half as frequently. Looks like we’re better at offense than defense.


Tomorrow, we’re going to teach Lua to interact with C++ code and generate some music. Remember that scheduler we wrote today? We’re going to need it to manage all the different voices that are contributing to our song.

Your Turn
Find…
The LuaRocks system for installing Lua modules onto your system

The open source LOOP library that implements a more sophisticated scheduler than the one we’ve written here

The list of all metatable functions that Lua recognizes (in addition to the __tostring, __index, and __newindex functions we used)

The name of the table where Lua keeps its global variables

Do (Easy):
Write a function called concatenate(a1, a2) that takes two arrays and returns a new array with all the elements of a1 followed by all the elements of a2.

Our strict table implementation in ​Reading and Writing​ doesn’t provide a way to delete items from the table. If we try the usual approach, treasure.gold = nil, we get a duplicate key error. Modify strict_write() to allow deleting keys (by setting their values to nil).

Do (Medium):
Change the global metatable you discovered in the Find section earlier so that any time you try to add two arrays using the plus sign (e.g., a1 + a2), Lua concatenates them together using your concatenate() function.

Using Lua’s built-in OO syntax, write a class called Queue that implements a first-in, first-out (FIFO) queue as follows:

q = Queue.new() returns a new object.

q:add(item) adds item past the last one currently in the queue.

q:remove() removes and returns the first item in the queue, or nil if the queue is empty.

Do (Hard):
Using coroutines, write a fault-tolerant function retry(count, body) that works as follows:

Call the body() function.

If body() yields a string with coroutine.yield(), consider this an error message and restart body() from its beginning.

Don’t retry more than count times; if you exceed count, print an error message and return.

If body() returns without yielding a string, consider this a success.

Example usage:

​ 	
retry(
​ 	
    5,
​ 	
​ 	
    ​function​()
​ 	
        ​if​ math.​random​() > 0.2 ​then​
​ 	
            ​coroutine​.​yield​(​'S​omething bad happened​')​
​ 	
        ​end​
​ 	
​ 	
        print(​'S​ucceeded​')​
​ 	
    ​end​
​ 	
)
Most of the time, the inner function will fail; retry() should keep trying until it’s achieved success or tried five times.

Hint: You may need to create more than one coroutine.
<!-- #endregion -->

```python

```
