---
layout: post
title: Coffescript gotchas
---

CoffeeScript Gotchas
---

Learning CoffeeScript is really learning two and a half languages. You have to 
learn CoffeeScript itself. You have to be familliar enough with JavaScript to
be comfortable digging through compiled CoffeeScript to debug. Most 
interestingly, though, you have to have a feel for a particular style and set
of assumptions to resolve ambiguity correctly. Today I ran into a case where 
I didn't have the right intuition and it took me a bit to figure out what 
was going on.

The issue was CS's list comprehensions. Coming from Python. I was comfortable
using that shorthand for assigning a list. In Python, I would use:

    x = [n * 2 for n in range(3)]
    print x
    >>> [0, 2, 4]

CoffeeScript is a bit different. At first I thought this would work:

    x = n * 2 for n in [0, 1, 2]

However, CoffeeScript's compiler doesn't assume you want a list when you use a
list comprehension. The result of that comprehension is `x = 4`. It compiles to:

    _ref = [0, 1, 2];
    for (_i = 0, _len = _ref.length; _i < _len; _i++) {
      y = _ref[_i];
      x = y * 2;
    }

Instead of making a list and then assigning it to `x`, the comprehension assigns
each value to x in turn, and x ends up as the last value. Ok, I can see how 
that might be a reaonable thing to assume. I really want a list though, maybe
square braces give me what I want?

    x = [y * 2  for y in [0, 1, 2]]

Sorry, no! This does succeed in assigning to a list, but it wraps the entire
expression in square braces, so I get a list containing the list I want as its 
zeroth element:
    
    x = [
      (function() {
        var _j, _len1, _ref1, _results;
        _ref1 = [0, 1, 2];
        _results = [];
        for (_j = 0, _len1 = _ref1.length; _j < _len1; _j++) {
          y = _ref1[_j];
          _results.push(y * 2);
        }
        return _results;
      })()
    ];

So how do I tell the compiler that I think of the right side of that original
equality, taken together, as one group, and it's that group I want assigned to 
the left side. Hmmm... grouping... I know, what If I try parentheses?

    x = (y * 2  for y in [0, 1, 2])

Bingo! this compiles to 

    x = (function() {
      var _j, _len1, _ref1, _results;
      _ref1 = [0, 1, 2];
      _results = [];
      for (_j = 0, _len1 = _ref1.length; _j < _len1; _j++) {
        y = _ref1[_j];
        _results.push(y * 2);
      }
      return _results;
    })();

which is exactly what I wanted. 

Each of these cases makes sense now that I've paid attention, but it took me 
a while to get there. Specifically, I notice a difference between the second and
third examples: in the second, the square brackets passed through the compilation
and were inserted into the resultant JavaScript. In the third, however, the 
enclosing parentheses were *interpreted* by the compiler and did not make it
into the compiled javascript. When it comes to symbols that are meaningful 
in both CoffeeScript and JavaScript, you just have to know which are going to
be interpreted at which level and in what situations.
