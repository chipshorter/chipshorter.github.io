---
layout: post
title: Hello Rust!
---
I've just recently started to look at the [Rust][2] progamming language, and I thought I'd just post a few notes as I went along, comparing, for example, how something might be done in Rust as opposed to [Java][3] (a language I already know pretty well).

One very early function I wrote was to calculate the factorial of a given integer, and I wanted to compare how I would do that in Rust against a similar implemention in Java using Java 8 streams. I know that using recursion would typically be the way to implement a factorial function, however, since neither Rust nor Java support [tail call][4] optimisation at the time of writing, I think it's better to look for alternatives to recursion, and in this case it was only a trivial function as part of my self-learning experience.

In neither case did I make any attempt to validate the input, so if an argument `< 0` or `> 20` is supplied, then things are going to go wrong anyway, but here are the implementations I ended up with.

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

Apart from the Rust syntax being a little more concise they are pretty much the same. Interestingly, if you try to call the function/method with a value of `21` then the Java method will simply return a "wrong" result (`-4249290049419214848`) due to overflowing the size of a `long`, whereas the Rust function will throw a runtime error:

`thread 'main' panicked at 'attempt to multiply with overflow'`

I personally think the Rust approach is preferable in this instance. Of course, it would be better to implement some error handling, but it's very early days for me with Rust. 


[1]: https://jekyllrb.com "Jekyll"

[2]: https://www.rust-lang.org "Rust"

[3]: http://openjdk.java.net "Java"

[4]: https://en.wikipedia.org/wiki/Tail_call "tail call"
