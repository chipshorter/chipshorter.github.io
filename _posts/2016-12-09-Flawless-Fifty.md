---
layout: post
title: Flawless 50 with Rust
---
Continuing my journey with [Rust][1], I've now found solutions for the first fifty problems over at [Project Euler][2], so I thought it was about time to write up a few of the more interesting Rust stuff I'd done, before I forgot and also as a reminder to myself just how bad my Rust code is at this point in time.

I'm not going to be giving away any of the answers to the problems, so no need to cover your eyes.

One of the questions we are asked to analyse a text file of dictionary words, all of which were upper-case A-Z, and we needed arrive at a sum for the word based on the characters position in the alphabet ('A'=1, 'B'=2, ... 'Z'=26). As part of this solution I also decided to write my own [finite-state machine][3] CSV parser as an exercise, which is another story, but for now, I did find a rather neat way of getting the 'word sum'.

{% highlight rust %}
// subtract 9 for the integer digits to get 'A'=1
let word_sum: u32 = word.chars().map(|ch| ch.to_digit(36).unwrap()-9).sum();
{% endhighlight %}

It's interesting to see how differently I've done things as I've gone on. Here is an example of string concatenation:

{% highlight rust %}
// first I did it like this
// concatenating an integer to an existing string
digits = format!("{}{}", digits, i.to_string());

// later I did it like this
// concatenating a character to an existing string
buffer.push(ch);
{% endhighlight %}

Here are two differnt ways I found of checking if a number was a palindrome (i.e. reads same left-to-right as right-to-left:

{% highlight rust %}
// version 1
fn is_palindrome(nbr: u64) -> bool {
    let nbrstr = nbr.to_string();
    let half = nbrstr.len() / 2;
    nbrstr.chars().zip(nbrstr.chars().rev()).take(half).filter(|&(x,y)| x == y).count() == n
}

// version 2
fn is_palindrome(nbr: u64) -> bool {
    let nbrstr = nbr.to_string();
    let half = nbrstr.len() / 2;  
    nbrstr.bytes().take(half).eq(nbrstr.bytes().rev().take(half))
}
{% endhighlight %}

I was intending this to be a longer post, but I've run of time for now, hopefully I'll find some time to complete it later.


[1]: https://www.rust-lang.org "Rust"

[2]: https://projecteuler.net "Project Euler"

[3]: https://en.wikipedia.org/wiki/Finite-state_machine "FSM"
