# Introduction-to-Lua


```python

```

#### An Interview with Roberto Ierusalimschy, creator of Lua
Interviewer:
###Why did you write Lua?

Roberto:

`Back in 1993, I was working as a consultant for Tecgraf, a partnership between my university (PUC-Rio) 
and Petrobras (the Brazilian Oil Company). There were two programs with somewhat similar problems for 
end-user configuration. People there had developed some little languages for their specific needs, but 
soon they realized that they needed more power from their languages, such as full arithmetic expressions, 
variables, conditionals, and even some abstractions (functions). However, they did not want to entangle 
their entire program with this configuration language. At that time, the only language with that profile 
was Tcl, but its syntax was too confusing for our users, who were mainly non-professional programmers 
(such as geologists and mechanical engineers). So, we started Lua for the very specific goal of providing
a good language for programs that needed a good configuration language.`

Interviewer:

What do you like the most about Lua?

Roberto:

Lua is a language with a very clear and small set of goals. It does not try to be everything for everybody. 
I am the first to recommend other languages when that is the case. Language design involves many trade-offs, 
and different languages solve those trade-offs in different ways, which are good or bad for different uses 
and users. A good programmer should know how to choose the best tool for each problem.

Interviewer:

What kinds of problems does it solve the best?

Roberto:

I think Lua is best suited for what it was created, real scripting. Nowadays, most people use "scripting language"
as almost a synonym for dynamic language. But scripting has a more specific meaning, of a language to 
"orchestrate," or to "glue," software that is frequently written in a different language. 
(Thinking about the script of a game like the script of a movie gives a good idea of what I mean by "scripting.")
Lua has always been developed with this kind of use in mind.

Interviewer:

What is a feature that you would like to change, if you could start over?

Roberto:
That is a tricky question to answer ; ) The new vararg mechanism, which was implemented only in version 5.1 of
the language (in 2006), is something that I do not like much. Frequently I think the old mechanism was better, 
but I do not think we can roll back to it. The pattern matching functions are another area where I would like to
use something based on PEGs, but I cannot see a roadmap from the current system for a new one.

Interviewer:

What’s the most surprising place you’ve ever seen Lua used in production?

Roberto:
Maybe games. Now games are the main niche of the language, but it was not like that in the beginning. We hardly 
thought about games in the first years of the language. When Grim Fandango came out, that was a big surprise for 
us. It is also a little surprising to see Lua embedded in so many devices, such as keyboards, printers, routers,
cameras, and the like.

```python

```

# Resources


http://www.lua.org/start.html


https://www.youtube.com/watch?v=iMacxZQMPXs

```python

```

```python

```
