---
layout: post
title: Binary operators-- a basic intro with Arduino.
---

Binary operators-- a basic intro with Arduino.
----

Bitwise operators are in the same category as familliar mathematical operators like +, -, /, and \*, but provide ways to operate on data as strings of binary bits, without paying attention to what the data represents. As a (mostly) python developer, I don't usually have a lot of reasons to use them. That's a shame, because whenever I do, I find myself thinking more carefully about how data is actually represented in a computer, and appreciating the neat little tricks that representation makes possible. In this post, I'm going to use the example of a little Arduino sketch that counts in binary using LEDs. 

<iframe width="420" height="315" src="//www.youtube.com/embed/E53rhBUhtLc" frameborder="0"> </iframe>

The code looks like this:

```
// NUMBER_OF_BITS is the number of individual LEDS we can turn on and off.
const int NUMBER_OF_BITS = 6;

// We'll keep track of the count in the variable 'count'.
int count = 0;

// Whenever we need to do something in a for-loop, we'll use n to keep 
// track of where we are.
int n;

// We'll use the variable 'command' to hold the value of the command
// we want to send to an individual pin. Arduino uses the built-in variable
// HIGH to mean that the pin should be powered on, and the built-in 
// variable LOW to mean that the pin should be powered off. The value
// of 'command' should therefore always get set to either HIGH or LOW.
int command;

// The setup function does what its name implies. It gets run once when 
// the board is powered on.
void setup() {

  // Tell Arduino that we'll be writing output to each pin from
  // 1 to NUMBER_OF_BITS
  for (n = 1; n <= NUMBER_OF_BITS; n++) {

    // pinMode is a built-in function for setting the mode of pin # n
    pinMode(n, OUTPUT);
  }
}

// This function gets executed over and over, as long as the board is on.
void loop() {

  // We want to consider each LED in turn, decide whether it should be on
  // or off based on the current count, and send a command to set it correctly.
  for (n = 1; n <= NUMBER_OF_BITS; n++) {
    
    // This comparison is the important part. I'll be covering what it does in the rest
    // of the post.
    if ((count & (1 << n)) > 0) {
      command = HIGH;
    } else {
      command = LOW;
    }

    // digitalWrite is another built-in function. It sends the command 'command'
    // to pin # n
    digitalWrite(n, command);
  }

  // add one to the count.
  count++;

  // wait for 1000ms (1 second) delay is another built-in function
  delay(1000);
}
```

First, we declare the variables NUMBER_OF_BITS, count, and n, which are the only non-built-in variables we'll need for this whole program. Then in the setup function, we set the mode of pins 1 through NUMBER_OF_BITS to OUTPUT, because we'll be controlling them rather than reading data from them. Then we have a loop function, which gets run over and over. In it, we display the current count on the LEDs in binary and then add one to the count variable. Next, we're going to look at the way we decide which LEDs should be on for a given number.

Remember that an integer represented by a computer is already a binary number. In binary, numbers are represented as ones and zeros. Because I happen to have six LEDs, I'm going to use six-digit binary numbers. So the number one might look like this: 

```
000001
```

However, that way of writing a number has a problem: why all the leading zeros? No one says a candy bar costs $000001.00, because those zeros don't change the value. 000001 == 1. You could just write "1."

And yet. When we look at the LED counter, we see that we have six LEDs. Even if we only want to show the number 1, we need to remember that the first 5 LEDs should be set to off. So we're going to use a slightly different way of writing binary numbers, where we always include every light, and use an X to denote lights that are turned on. In that notation, the number 1 looks like this: 

```
| | | | | |X|
```
That way, we see the total number of bits we *could* use, even if we're not using all of them. Let's look at addition in this notation next. To add two numbers, we're going to show the numbers themselves, and the answer, and we're going to use an extra number to show the carryover. So adding 1 + 1 looks like this:

```
carryover -> | | | | | | |
        1 -> | | | | | |X|
      + 1 -> | | | | | |X|
         -----------------
   result -> | | | | | | |
```

Adding an X to an X means that we put a space in the result and carry an X, sort of like adding 9 + 1 in decimal arithmetic.

```
carryover -> | | | | |X| | 
        1 -> | | | | | |X|
      + 1 -> | | | | | |X|
         -----------------
   result -> | | | | | |_|
```

Then we see that there are no Xs in the second column, so the carried X goes into the answer, and we get:

```
carryover -> | | | | |X| |
        1 -> | | | | | |X|
      + 1 -> | | | | | |X|
         -----------------
   result -> | | | | |X| |
```

Notice how we can convert this answer back into 'traditional' binary notation by writing it as ones and zeros: 000010, and we can also see what it means for the LEDs: the second from the right should be on, and all the others should be off.

Remember that integers in a computer *always look like this*. There is no operation needed to convert an integer into binary, because that's already how it's stored. What is sometimes hard is getting the computer to *show* you the binary representation, because computers almost always display numbers in decimal notation for human convenience.

There are some interesting possibilities when we start thinking of numbers as binary strings. If we have the string ```| | | |X| | |```, we can use "shift" operators to push the string to the left and right. The binary shift operators are left-shift: ```<<``` and right-shift: ```>>```. We use them like this:

```
| | | |X| | | << 2 = | |X| | | | |

| | | |X| | | >> 2 = | | | | | |X|

```
Note that if you push an X off the string to either side, it just goes away [1]:

```
| | | |X| | | << 20 = | | | | | | |

| | | |X| | | >> 20 = | | | | | | |
```

Logical operators that act on the binary representations of numbers are called "bitwise operators" because they operate bit by bit, rather than on the numbers as a whole. There are bitwise versions of the 'and' operator: ```&```, the 'or' operator: ```|```, the 'XOR' operator: ```^```, and the 'not' operator: ```~```. We're only going to look at the and operator in this post, because it's the only one I used in the counter.

The and operator compares two binary strings and produces a third string with Xs in any position where *both* of the input strings had Xs:

```
   | |X|X| | | |
 & |X| |X| | | |
----------------
   | | |X| | | |
```

A common use of the and and or operators is "masking," and it's motivated by the need to see what's in a particular bit or sequence of bits in a number. Masking works by and-ing the number we'd like to inspect with a number that has Xs in the positions we want to inspect. For instance, when we want to know whether the third bit of a number has an x, we could and it with ```| | | |X| | |```:

```
 input ->   |X|X|X|X|X|X|      | | | | | | |
 mask  -> & | | | |X| | |    & | | | |X| | |
          ---------------   ---------------
 result ->  | | | |X| | |      | | | | | | |
```

For our counter, we have a variable called 'count' that will hold the current number we want to display. We want to turn each LED on or off based on whether the corresponding bit in 'count' is a 1 or a 0. We can make a bit mask for each bit in turn by starting with the number ```| | | | | |X|```, and then left-shifting it until the X is in the position we want to inspect. When we 'and' the count variable with this mask, we will either get zero, if the bit we were interested in was a zero, or we'll get a number bigger than zero if it wasn't. 

So for each number n between 1 and NUMBER_OF_BITS, we 'and' the count variable with ```1 << n```, and then set LED # n to HIGH if the result is bigger than 0 and LOW if not [2]:

```
for (n = 1; n <= NUMBER_OF_BITS; n++)  {
  if (count & (1 << n)) > 0) {
     command = HIGH;
  } else {
     command = LOW;
  }
  digitalWrite(n, command);
}
```

[1] This is an oversimplification. The data type (int, long, etc) often has a set width. Bits shifted beyond that width go away.  
[2] Also an oversimplification. Using Two's Complement representation <http://en.wikipedia.org/wiki/Two%27s_complement>, one bit is used to denote whether the value is positive or negative. I haven't run the counter long enough for the representation of int to wrap, so I'm not sure what will hapopen when it does. Exciting!
