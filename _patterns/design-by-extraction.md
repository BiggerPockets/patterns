---
categories: Our Practices
name: Design by Extraction
---

If you're presented with a problem that has already been solved by loads of duplication and you want to reduce some of
that duplication by introducing a design, make sure that whatever design you're proposing reduces some of that
duplication.

It's better to be intentional about global rewrites than to solve one problem one time with a one-off design.

# Why?

Code should be exemplary. If you add a new design but only put it in one place, how will other engineers know
to use it? The new design has not reduced overall complexity.

It should be obvious that there is now _a better way_ because alternative ways have been removed.

Consider this quote from Sandi Metz's excellent _Practical Object Oriented Design_ (2nd ed, pages 6 and 8):

> Slightly more experienced programmers encounter different design failures. These programmers are aware of OOD
> techniques but do not yet understand how to apply them. With the best of intentions, these programmers fall into the
> trap of overdesign. A little bit of knowledge is dangerous; as their knowledge increases and hope returns, they
> _design_ relentlessly. In an excess of enthusiasm, they apply principles inappropriately and see patterns where none
> exist. They construct complicated, beautiful castles of code and then are distressed to find themselves hemmed in by
> stone walls. You can recognize these programmers because they begin to greet change requests with "No, I can't add
> that feature; _it wasn't designed to do that_."

> Finally, object-oriented software fails when the act of design is separated from the act of programming. Design is a
> process of progressive discovery that relies on a feedback loop. This feedback loop should be timely and incremental;
> the iterative techniques of the [Agile software movement](http://agilemanifesto.org/) are thus perfectly suited to the
> creation of well-designed OO applications. The iterative nature of Agile development allows designs to adjust
> regularly and to evolve naturally. When design is dictated from afar, none of the necessary adjustments can occur and
> early failures of understanding get cemented into the code. Programmers who are forced to write applications that were
> designed by isolated _experts_ begin to say, "Well, I can certainly write this,
> _but it's not what you really want and you will eventually be sorry_."

> [...]

> Agile [...] requires really good design. It needs your best work. Its success relies on simple, flexible, and
> malleable code.