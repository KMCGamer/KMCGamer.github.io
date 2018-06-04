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

I've been doing HackerRank exercises lately, and came across [this question](https://www.hackerrank.com/challenges/circular-array-rotation/problem). For those who don't want to read a giant wall of text, below is a quick summary of the question:

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

In Java, while the modulus operator is called "modulus", what it really means is remainder. So when you do `-3 % 4` you will get `-3` because 4 goes into -3 zero times with a remainder of -3.

However, some other languages like Python handle modulus differently. If you were to do the same math in Python, you would get `1`. Why? Because 4 times 1 equals 4 and then minus 3 is 1.

You don't have to worry about this when dealing with positive numbers. Only when your dividend/divisor is negative do the two methods differ.

![mind blown](https://media.giphy.com/media/xT0xeJpnrWC4XWblEk/giphy.gif)

You can read more about this [here](https://stackoverflow.com/questions/5385024/mod-in-java-produces-negative-numbers) and [here](https://stackoverflow.com/questions/13683563/whats-the-difference-between-mod-and-remainder) on StackOverflow.

## Wrapping It Up

Whats makes this so interesting to me is the fact that I have been using the modulus operator for so long and have never had to work with negative numbers. Really goes to show that there is always something new to learn.

Also here is my solution to the HackerRank problem. I converted the remainder to a modulus.

```java
static int[] circularArrayRotation(int[] a, int[] m, int k) {
    int[] values = new int[m.length];
    for (int i = 0; i < m.length; i++) {
        int idx = m[i];       
        values[i] = a[(((idx - k % a.length) + a.length) % a.length)];
    }
    
    return values;
}
```
