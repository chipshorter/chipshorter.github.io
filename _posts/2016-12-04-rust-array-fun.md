---
layout: post
title: Rust array fun
---
I'm still in the initial stages of trying to learn the [Rust][1] programming language, so I'm going throught the problems over at [Project Euler][2] and working my way through the problems and trying to solve them using Rust. Initially, I solved the first 30 or so using [Java][3] and then converted my solution to [Rust][1], but eventually I bit the bullet and am now trying to solve them without resorting to the comfortable pair of slippers.

Anyway, to cut a long story short, one of the problems concerned [Pandigital Numbers][4], or in this specific case, [Pandigital Numbers][4] containing exactly one each of the digits 1-9 only. So I wrote a function to check whether a number I had generated satisfied that criteria. Here it is:

{% highlight rust %}
fn is_pandigital(nbr: u64) -> bool {
    // create array of 9 elements initialiset to zero
    let mut a = [0; 9];
    let mut x = nbr;
    while x != 0 {
        let rem = x % 10;
        // we only want 1-9
        if rem == 0 {return false}
        let idx: usize = (rem - 1) as usize;
        a[idx] += 1;
        // got a number twice?
        if a[idx] > 1 {return false}
        x = x / 10
    }
    // must be true at this point, so no real need for this
    a.iter().fold(0, |a, b| a + b) == 9
}
{% endhighlight %}

I'm only learning, so there are probably some things here that I'm missing entirely, but I really struggled with indexing the array, and I didn't expect that.

For example this would work fine (as I expected):
{% highlight rust %}
let mut a = [0; 9];
a[0]=2;
println!("a[0]={}",a[0]);
{% endhighlight %}
But I couldn't get anything like this to compile:
{% highlight rust %}
let mut a = [0; 9];
let mut num: u64 = 5;
let mut idx = num % 10;
a[idx]=2;
println!("a[0]={}",a[0]);

// generates a compiler error as below
error[E0277]: the trait bound `[{integer}]: std::ops::IndexMut<u64>` is not satisfied
  --> src/problems/scratchpad.rs:28:5
   |
28 |     a[idx]=2;
   |     ^^^^^^ trait `[{integer}]: std::ops::IndexMut<u64>` not satisfied
   |
   = note: slice indices are of type `usize`
{% endhighlight %}

Hence if you look at the function I ended up with, there is a cast
{% highlight rust %}let idx: usize = (rem - 1) as usize;{% endhighlight %}

This makes it work, but somehow it feels wrong. I'll probably figure it out at some point in the future.

[1]: https://www.rust-lang.org "Rust"

[2]: https://projecteuler.net "Project Euler"

[3]: http://openjdk.java.net "Java"

[4]: https://en.wikipedia.org/wiki/Pandigital_number "Pandigital Numbers"
