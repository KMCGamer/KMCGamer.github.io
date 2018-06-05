---
layout: single
title: Remainder vs. Modulus
categories:
  - programming
tags: 
  - java
  - arithmetic
  - hackerrank
---

I've been doing HackerRank exercises lately, and came across [this question](https://www.hackerrank.com/challenges/circular-array-rotation/problem). Please note that this blog post uses **Java 8**, so YMMV when using other languages. For those who don't want to read a giant wall of text, below is a quick summary of the question:

You are given an array of integers, and this array is able to undergo a certain amount of "right circular rotations". To do a rotation, you pop the last element off the array and append it to the front. So, given an array of integers, `a`, and an integer of the number of times the array has undergone a rotation, `k`, you are tasked with outputting the elements that are now at a particular set of indices (array of integers `m`).

For instance, if we have an array `a = [1, 2, 3]` and a number of rotations, `k = 2`, our new array will look like `[2, 3, 1]` since 3 got moved the front, and then 2 got moved the front. Now if we are given an array `m = [0, 1, 2]` containing the indices we want, we would output `[2, 3, 1]` since index 0 = 2, 1 = 3, and 2 = 1.

## The Idea

I immediately noticed the circular pattern to this array. If you were to do one more rotation to this array (so k would be equal to 3), you would be right back where you started with `[1, 2, 3]`. My mind went straight to the modulus operator.

I mapped out the pattern like so:

```
k = 0: [A, B, (C), D] -> [A, B, (C), D] // 2 -> 2
k = 1: [A, B, (C), D] -> [D, A, (B), C] // 2 -> 1
k = 2: [A, B, (C), D] -> [C, D, (A), B] // 2 -> 0
k = 3: [A, B, (C), D] -> [B, C, (D), A] // 2 -> 3
k = 4: [A, B, (C), D] -> [A, B, (C), D] // 2 -> 2
```

So it seems that for every time you rotate, the desired index gets shifted once to the left. So C becomes B, B becomes A, A becomes D, D becomes C, rinse and repeat.

I came up with the following equation: `newIdx = oldIdx - k % length` where `newIdx` represents the new element index that is in the place of the old (original) index after `k` rotations, and `length` is the size of the array. Boom, all done.

## The Difference

So what took me so long? For starters, I did not know exactly how the modulus operator would react to a negative number. So, I went online and tried out some modulus calculators.

Lets try [this calculator](https://www.miniwebtool.com/modulo-calculator/?number1=-3&number2=4) from miniwebtool.com. When we input -3 % 4, we get 1. Interesting... Now lets try [this other calculator](https://www.dcode.fr/modulo-n-calculator) from dcode.fr. If we put -3 % 4, we get -3 this time. **WHAT GIVES?!**

Here is the ["Modulo Operation"](https://en.wikipedia.org/wiki/Modulo_operation) as defined by Wikipedia:
> Given two positive numbers, `a` (the dividend) and `n` (the divisor), `a modulo n` (abbreviated as `a mod n`) is the remainder of the [Euclidean division](https://en.wikipedia.org/wiki/Euclidean_division) of `a` by `n`. For example, the expression `5 mod 2` would evaluate to 1 because 5 divided by 2 leaves a quotient of 2 and a remainder of 1, while `9 mod 3` would evaluate to 0 because the division of 9 by 3 has a quotient of 3 and leaves a remainder of 0; there is nothing to subtract from 9 after multiplying 3 times 3.

Heres is where it gets tricky. Further down it states this:
> When either `a` or `n` is negative, the naive definition breaks down and programming languages differ in how these values are defined.

![mind blown](https://media.giphy.com/media/xT0xeJpnrWC4XWblEk/giphy.gif)

**Mind. Blown.**

This explains why Java is giving me the undesired -3 result instead of the 1 result! Java calulates the remainder as -3 because 4 goes into -3 zero times, with a remainder of -3 (`4 * 0 + -3 = -3`). This is exactly how remainders work in mathematics, but I don't want the remainder, **I want the modulus!**

In the table on the same wikipedia page it states that in Java, the modulo result will always have the sign of the Dividend if we use the `%` operator. However, if we `Math.floorMod` we can make the modulo result share the sign of the Divisor. So instead of `-3 % 4` equaling -3, we can do `Math.floorMod(-3, 4)` and get 1, which will effectively help us cycle through the array backwards with no problems.

Please note that `Math.floorMod` is only available in Java 8. I provide an alternative solution for pre-Java 8 in my full answer below.

Here are some articles you can read to further grasp this concept:
- [Mod In Java Produces Negative Numbers](https://stackoverflow.com/questions/5385024/mod-in-java-produces-negative-numbers)
- [Whats the difference Between Mod and Remainder](https://stackoverflow.com/questions/13683563/whats-the-difference-between-mod-and-remainder)
- [A Neat Trick to Compute Moduluo of Negative Numbers](https://dev.to/maurobringolf/a-neat-trick-to-compute-modulo-of-negative-numbers-111e)

## Wrapping It Up

Whats makes this so interesting to me is the fact that I have been using the modulus operator for so long and have never knew that a problem like this existed. Really goes to show that there is always something new to learn.

Also here is my solution to the HackerRank problem:

```java
static int[] circularArrayRotation(int[] a, int[] m, int k) {
    int[] values = new int[m.length];
    for (int i = 0; i < m.length; i++) {
        int idx = m[i];       
        values[i] = a[Math.floorMod(idx - k, a.length)]; // Java 8
        // Alternative solution:
        // values[i] = a[(((idx - k % a.length) + a.length) % a.length)];
    }
    return values;
}
```
