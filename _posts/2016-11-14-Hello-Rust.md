---
layout: post
title: Hello Rust
---
I recently created a new blog on blogger.com where I wanted to show code samples, but found that getting markdown code onto the blog entry was pretty painful, so I've decided to give [Jekyll][1] a shot to see if I fare any better.

As way of creating a "Hello, world!" blog entry, and because I've just started to look at the [Rust][2] progamming language, I thought I'd just post a few of my notes on things as I went along, comparing, for example, how something might be done in [Rust][2] as opposed to [Java][3] (a language I already know pretty well).

One very early method/function I tried to write was to calculate the factorial of a given integer. This is a simple function which would typically be implemented as a recursive function. However, since neither [Rust][2] nor [Java][3] support [tail call][4] optimisation at the time of writing, I wanted to do this using [Java][3] 8 streams and the equivalent in rust.

In neither case have I made any attempt to validate the input, so if an argument `< 0` or `> 20` is supplied, then things are going to go wrong anyway, but here are the implementations.

{% highlight java %}
// Java - only works for 0 <= n <= 20
public static long factorial(int n) {
    return LongStream.rangeClosed(2, n).reduce(1, (sum, x) -> sum * x);
}
// prints -> 10 factorial is 3628800 
System.out.println("10 factorial is " + factorial(10));
{% endhighlight %}

{% highlight rust %}
// Rust - only works for 0 <= n <= 20
fn factorial(n: usize) -> usize {
    (1..n+1).fold(1, |sum, x| sum * x)
}
// prints -> 10 factorial is 3628800 
println!("10 factorial is {}", factorial(10));
{% endhighlight %}

The [Rust][2] syntax is a little more concise, but both are pretty much the same in this case. Interestingly, if you try to call the function/method with a value of `21` then the [Java][3] method will just return a wrong result(`-4249290049419214848`) due to overflowing the size of a `long`, whereas the [Rust][2] function will throw a runtime error `thread 'main' panicked at 'attempt to multiply with overflow'`, which I prefer as it's safer. Of course, it would be better to implement some error handling, but it's early days for me with [Rust][2]. 


[1]: https://jekyllrb.com "Jekyll"

[2]: https://www.rust-lang.org "Rust"

[3]: http://openjdk.java.net "Java"

[4]: https://en.wikipedia.org/wiki/Tail_call "tail call"
