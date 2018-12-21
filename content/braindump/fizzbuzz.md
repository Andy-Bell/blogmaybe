+++
title = "12 'Days' of Fizzbuzz"
description = "Fizzbuzz in as many languages as I can write it in"
date = 2018-12-21
+++

Thought I would try a slight christmas-themed post digging into differences in FizzBuzz implementations.

For those who have not heard the term "FizzBuzz" it is a small counting challenge, where the person or program will count out numbers and on multiples of 3 will instead of saying the number will say Fizz, on multiples of 5 say Buzz and on multiples of both 3 and 5 will say FizzBuzz.
<!-- more -->

The first implementation will be in Ruby (as that was the first lang I worked in):
```ruby
```

The second will be in javascript:
```js
```

The third is one of my favourites, in Erlang:
```erl
-module(fizzbuzz).
-export([fizzbuzz/1, count/1]).

fizzbuzz({ 0, 0, _ }) -> io:format("FizzBuzz~n");
fizzbuzz({ 0, _, _}) -> io:format("Buzz~n");
fizzbuzz({ _, 0, _}) -> io:format("Fizz~n");
fizzbuzz({_, _, N}) -> io:format("~p~n", [N]);
fizzbuzz(N) when N > 0 -> fizzbuzz({ N rem 5, N rem 3, N});
fizzbuzz(N) -> {error, "N < 0"}.

count(X) ->
  for(1, X).

for(X, X) -> [fizzbuzz(X)];
for(I, X) -> [fizzbuzz(I) | for( I+1, X)].
```

I really like erlang as a language from my brief experiments in it, but have not had much opportunity to use it in large-scale projects (anything beyond small toy scripts like this really). The erlang example makes use of two really nice features of the language, pattern matching and multiple function signatures.

The count function takes the maximum value passing it into another function as the second argument, allowing us to iterate upwards. The multiple signatures allows us to essentially pattern match on the functions themselves, so that if the two arguments match, it knows that it is at the maximum value.

In the fizzbuzz function we create a series of patterns of tuples to match against, similar to the above implementations, but also allows us to use the same function name for the initial tuple generation. The function signatures are matched against in order, so the fizzbuzz(N) will always be matched against in case of errors.
