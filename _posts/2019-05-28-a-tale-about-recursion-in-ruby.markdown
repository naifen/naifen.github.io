---
toc: true
toc_label: "Table of Contents"
toc_icon: "file-alt"
layout: single
title:  "A Tale About Recursion In Ruby"
date:   2018-05-28 21:05:13 -0700
categories: ruby recursion tips tricks
---
There are many ways to do "recursion" in ruby, in this post I'm going to
talk about some of the tips & tricks I've learned about it.

## Intro

Recursion was a pain for me when I first touched it in school. I was using
Java at that time, it was just hard to think problems through recursively.
I took me time to get used to it, and once I was comfortable with it,
it's a surprisingly easier way to solve some programming problems, for
example the classic fibonacci problem.

## Fibonacci example

### Traditional solotion

The traditional way to write Fibonacci funtion in a dynamic language
like Ruby is pretty straight forward:
{% highlight ruby %}
def fib(n)
  if n <= 1
    n
  else
    fib(n - 1) + fib(n - 2)
  end
end

# or write it as one-liner
def fib(n)
  n <= 1 ? n : fib(n - 1) + fib(n - 2)
end
{% endhighlight %}
They work well untill you do something like ``fib 50``. I tried this on
my macbook pro and it got stuck. What we do?

### Memoization is here to rescue

Ruby has an operator ``||=``. I used it quite often as soon as I first
started programming in Ruby and Rails, eg ``@current_user ||=
find_user``. We can create a memoization recursion using the same
pattern.
{% highlight ruby %}
def memo_fib(n, memo = {})
  return n if n <= 1
  memo[n] ||= memo_fib(n - 1, memo) + memo_fib(n - 2, memo)
end
{% endhighlight %}
To explain it using my own words, calculations are memoized using the
``||=`` operator to avoid repeated calculations. Now fire up your
favoriate Ruby REPL and try ``memo_fib 50``, voilà it worked. I draw a
diagram to try to help myself understand it:
{% highlight ruby %}
# fib(5, {})                    VI|- NOTE memo[3] is already set so return 2 without perform any calculation
#   |-memo[5] ||= fib(4, memo) + fib(3, memo)
#                 |I
#                 |-memo[4] = fib(3, memo) + fib(2, memo) => 2 + 1 => 3 NOTE: memo is now { 2 => 1, 3 => 2, 4 => 3 }
#                             |II eval to 2   |V
#                             |               |- fib(1, memo) + fib(0, memo) => 1 + 0 => 1
#                             |
#                             |- NOTE: memo is { 2 => 1, 3 => 2} AFTER IV
#                             |                             IV|- n == 1 return 1
#                             |-memo[3] = fib(2, memo)   +  fib(1, memo) => NOTE: memo is now { 3 => 2 }
#                                      III|-memo[2] = 1 + 0 => NOTE: memo is now { 2 => 1 }
#
# NOTE: to inspect memoization add "puts memo" right above memo[n] ||= line
# this is the print result for fib(10)
# {}
# {}
# {}
# {}
# {}
# {}
# {}
# {}
# {}
# {2=>1, 3=>2}
# {2=>1, 3=>2, 4=>3}
# {2=>1, 3=>2, 4=>3, 5=>5}
# {2=>1, 3=>2, 4=>3, 5=>5, 6=>8}
# {2=>1, 3=>2, 4=>3, 5=>5, 6=>8, 7=>13}
# {2=>1, 3=>2, 4=>3, 5=>5, 6=>8, 7=>13, 8=>21}
# {2=>1, 3=>2, 4=>3, 5=>5, 6=>8, 7=>13, 8=>21, 9=>34}
{% endhighlight %}

### Ruby Hash has the same magic of memoization

Because of the nature of Hash in Ruby, memoization is enabled by
default. Since Ruby 2.4, Hash received a pretty significant performance
improvement. We can convert our fibonacci function to Hash versions as
well:
{% highlight ruby %}
# consider |fib, n| as |key, value| where each key provided will get its value
hash_fib = Hash.new do |fib, n|
  fib[n] = n < 2 ? n : fib[n - 1] + fib[n - 2]
end

# or a merge! version, replace ternary "n < 2 ? n" part with default values
hash_fib = Hash.new do |fib, n|
  fib[n] = fib[n - 1] + fib[n - 2] }
end.merge!(0 => 0, 1 => 1)
{% endhighlight %}
These Hash verions have incredible speed, try call ``hash_fib[1000]`` in a
REPL and it's like instant.

### Tail recursion is more memory friendly

Although Ruby Hash has great performance, but try ``hash_fib[10000]``
directly in a REPL, this is what I received ``SystemStackError: stack level too deep``, noting is perfect right? Let me introduce tail recursion.
{% highlight ruby %}
def tail_fib(n, a = 0, b = 1)
  return a if n == 0
  return b if n == 1

  tail_fib(n - 1, b, a + b)
end
{% endhighlight %}
Notice the biggest difference of tail recursion version fibonacci is it
returns a function call at the bottom, rather than multiple function
calls and operations like all the other versions. This result in less
stack, save memory.
{% highlight ruby %}
# to use tail recursion in this file, open a REPL and do the followings:
require "./tail_recur_config.rb"
# => true
require "./tail_recursion.rb"
# => true
# and call tail recursion methods, and tail_recur_config.rb file looks
# like this
RubyVM::InstructionSequence.compile_option = {tailcall_optimization: true, trace_instruction: false}
{% endhighlight %}
Now I can even do ``tail_fib 100000`` like a easy clap 🥳.

## Tail call

Here are some good explainations I found online from GeeksforGeeks and
Stackoverflow:
> The tail recursive functions considered better than non tail recursive functions as tail-recursion can be optimized by compiler. The idea used by compilers to optimize tail-recursive functions is simple, since the recursive call is the last statement, there is nothing left to do in the current function, so saving the current function’s stack frame is of no use.

> Tail-call optimization is where you are able to avoid allocating a new stack frame for a function because the calling function will simply return the value that it gets from the called function. The most common use is tail-recursion, where a recursive function written to take advantage of tail-call optimization can use constant stack space.

So in my simple pharase, tail call version of recursions only return a
function call at the end without doing anything else, which can reduce
stacks.
Some more tail recursions examples implemented in Ruby:
{% highlight ruby %}
# Factorial
def tail_factorial(n, prod = 1)
  return prod if n == 1

  tail_factorial(n - 1, n * prod) # just return a function call
end

# Sum
def tail_sum(n, total = 0)
  return total if n == 0

  tail_sum(n - 1, total + n)
end
{% endhighlight %}
Please take a look at my github [repo][tail-call-repo] for more explanations and benchmark about tail recursion.

Also Here is a intersting and creative [video][tail-recursion-vid] about
Tail call, I highly recommend you to take a look at it.

[tail-recursion-vid]: https://www.youtube.com/watch?v=-PX0BV9hGZY
[tail-call-repo]:     https://github.com/naifen/ruby-stuff/tree/master/tips_tricks
