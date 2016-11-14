---
layout: post
title: Hello Rust!
---
I've just recently started to look at the [Rust][2] progamming language, and I thought I'd just post a few notes as I went along, comparing, for example, how something might be done in Rust as opposed to [Java][3] (a language I already know pretty well).

One very early method/function I tried to write was to calculate the factorial of a given integer. This is a simple function which would typically be implemented as a recursive function. However, since neither Rust nor Java supports [tail call][4] optimisation at the time of writing, I personally think it's better in most cases to look for alternatives to using recursion in most cases, and anyway I specifically wanted to use Java 8 streams and the equivalent in rust for this comparison.

In neither case have I made any attempt to validate the input, so if an argument `< 0` or `> 20` is supplied, then things are going to go wrong anyway, but here are the implementations.

In Java:
{% highlight java %}
// Java - only works for 0 <= n <= 20
public static long factorial(int n) {
    return LongStream.rangeClosed(2, n).reduce(1, (a, b) -> a * b);
}
// prints -> 10 factorial is 3628800 
System.out.println("10 factorial is " + factorial(10));
{% endhighlight %}

and in Rust:
{% highlight rust %}
// Rust - only works for 0 <= n <= 20
fn factorial(n: usize) -> usize {
    (2..n+1).fold(1, |a, b| a * b)
}
// prints -> 10 factorial is 3628800 
println!("10 factorial is {}", factorial(10));
{% endhighlight %}

Apart from the Rust syntax being a little more concise they are pretty much the same. Interestingly, if you try to call the function/method with a value of `21` then the Java method will simply return a wrong result(`-4249290049419214848`) due to overflowing the size of a `long`, whereas the Rust function will throw a runtime error:

`thread 'main' panicked at 'attempt to multiply with overflow'`

I personally think the Rust approach is preferable in this instance. Of course, it would be better to implement some error handling, but it's very early days for me with Rust. 


[1]: https://jekyllrb.com "Jekyll"

[2]: https://www.rust-lang.org "Rust"

[3]: http://openjdk.java.net "Java"

[4]: https://en.wikipedia.org/wiki/Tail_call "tail call"
