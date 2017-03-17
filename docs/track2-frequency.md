# Clojure maps and frequency analysis 

## Frequency analysis for breaking ciphers

English language, as most languages, has an even distribution of letters. The most frequent letter in English is `e`, which, on average, constitutes close to 13% of English texts, followed by `t` (about 9%) and `a` (a bit over 8%). The least frequent letter is `z` which occurs less than 0.1%.

A complete distribution of English letters is given here: [Relative frequencies of letters in the English language (wikipedia)](https://en.wikipedia.org/wiki/Letter_frequency#Relative_frequencies_of_letters_in_the_English_language) You might want to open this link in a new tab since you would need it. 

The non-even distribution of letters and the fact that every letter is always encrypted by the same letter (no matter where it appears in the text) allow breaking a Caesar cipher encryption (i.e. finding out the key) much faster than trying all possible keys and looking if the encryption contains any words. This approach is known as *frequency analysis*. 

Note that we need a somewhat long English text in order to do frequency analysis: a single word is not enough. 

Our example will be this text encrypted with Caesar cipher with an unknown key:

```clojure
"radyjgtxhpsncpbxrvtctgpaejgedhtegdvgpbbxcvapcvjpvtrdbqxcxcv
iwtpeegdprwpqxaxinpcsxcitgprixktstktadebtciduphrgxeixcvapcvjp
vtlxiwpctuuxrxtcipcsgdqjhixcugphigjrijgtudgbjaixiwgtpstsegdvg
pbbxcvo"
```

## Clojure hashmaps

*Relevant functions:* [assoc](https://clojuredocs.org/clojure.core/assoc), [zipmap](https://clojuredocs.org/clojure.core/zipmap)

A convenient way of storing of frequencies is a Clojure collection type known as a *hashmap*, also referred to as just a *map* - just don't confuse it with a function `map`. 

A map is a sequence of pairs of a key and a value. It is written in curly braces. For instance, `{"apple" 2, "banana" 3}` is a map with a key `"apple"` bound to a value `2`, and a key `"banana"` bound to a value `3`. 

Keys and values can be anything: numbers, strings, vectors, etc. It is common to use a special Clojure type known as a *keyword* for keys, as it may make lookups in larger maps slightly faster, but this is by no means required. Keywords in Clojure are written like this: `:apple`. 

Since we are computing frequencies of characters, our keys will be characters, and values will be counts or percentages.

Here is some of functionality of maps: 

1. `{}` is an empty map. 
2. You can add elements to a map using a function `assoc` by providing a key and a value. If a key given to `assoc` is already present, the value gets replaced by the new one.  
```clojure
(assoc {} "a" 3) ; results in {"a" 3}
(assoc {"a" 3} "x" 2) ; results in {"a" 3, "x" 2}
(assoc {"a" 3, "x" 2} "a" 5) ; results in {"a" 5, "x" 2}
```
3. The main use of maps is to be able to look up a value for a key. There is no special function that does it: the map itself serves as a function that, when passed a key, returns its value. If the key isn't found, a special value `nil` is returned:
```clojure
({"a" 3, "x" 2} "a") ; results in 3
({"a" 3, "x" 2} \a) ; results in nil (\a is different from "a')
({"a" 3, "x" 2} "b") ; results in nil
({} "any") ; results in nil
```
4. Another way of creating a map is to give it a vector of keys and a vector of corresponding values. The function that does this is called `zipmap`:
```clojure
(zipmap [\a \b] [2 5]) ; results in {\a 2, \b 5}
```

**Exercise:** Practice with maps. For instance, create a small map of animals and their sounds (e.g. "duck"/"quack"), and look up animal sounds by the animal. 

## Counting the number of occurrences of a character 
*Relevant functions:* [count](https://clojuredocs.org/clojure.core/count), [first](https://clojuredocs.org/clojure.core/first), [second](https://clojuredocs.org/clojure.core/second), [=](https://clojuredocs.org/clojure.core/=).

In order to compute frequencies of letters, we need to find out how many times each letter appears in the encrypted string. We also need to work with vectors that are pairs of a key and a value. 

The functions that we need for work with vectors (not necessarily all in this section) are:

1. `count` that returns the  umber of elements in a vector: `(count ["apple" "banana"])` returns `2`.
2. `first` returns the first element of a vector: `(first ["apple" "banana"])` returns `"apple"`. 
3. `second` returns the second element of a vector: `(second ["apple" "banana"])` returns `"banana"`, so does `(second ["apple" "banana" "kiwi"])`

Note that these functions also work with sequences (we will see those later). 

Now we are back to writing Clojure functions. We need to write a function `count-letters` that, given a character and a string, would return the number of occurrences of this character in the string. For instance,
```clojure
(count-letters \a "aadvark") ; 3
(count-letters \x "aadvark") ; 0
```

One (not the most efficient, but straightforward) way is filter out all the other characters, and then return the count of the result. In Clojure `=` is a function that can be used on any two objects (be it numbers, strings, vectors, etc.). For instance, `(= (/ 6 2) 3)` returns `true` (note that `(/ 6 2)` is the result of dividing 6 by 2), and `(= "97" 97)` returns `false` since a string is not equal to a number. 

Here we need to use `filterv` with an anonymous function that checks if its parameter is equal to the given character, and then we need to return the `count` of the result.  

**Exercise:** Look at your previous function that uses `filterv`, and then write a call to `filterv` to filter out all characters other than `\a` in the word `"aadvark"`. Once you have accomplished this task, make it into a function `count-letters` that works as in the example above. Try it on a different word and with a different character.

## Clojure lazy sequences, ranges
*Relevant functions:* [range](https://clojuredocs.org/clojure.core/range), [take](https://clojuredocs.org/clojure.core/take), [drop](https://clojuredocs.org/clojure.core/drop)

Data is often organized in a sequential way: the first element, then the second, etc. Most of Clojure predefined functions work with data in a sequential way, processing elements one after another. 
So far the only sequential collections of data that we have seen were vectors. However, a lot of sequential data in Clojure is given by sequences. The interesting feature of Clojure is that sequences are *lazy*, i.e. they can be potentially infinite, but only the portion of the sequences that's being used is *realized*, i.e. evaluated. 

One example of such lazy sequences is a range. A range is a sequence of integers, and it is generated by a function `range`. For instance, `(range 2 5)` gives you a sequence `(2 3 4)` (the upper bound is not inclusive). `(range 10)` gives you a sequence of integers from 0 to 9 (not including 10).  

You can try the above examples in REPL. What you shouldn't try in REPL, however, is a call to ` range` with no parameters: `(range)`. This gives you a sequence of integers from 0 to infinity! However, you can store this sequence in a variable, since this will not evaluate it:
```clojure
(def inf (range))
```
Now you can take as many elements of it as you need, using the function `take`:
```clojure
(take 20 inf) ; integers from 0 to 19
(take 100 inf) ; integers from 0 to 99
```
`take` has a complementary function `drop` that drops the first `n` elements. Using `drop` on an infinite sequence would produce an infinite sequences. 

Now going back to descriptions of `map` and `filter`, you would notice that they both return lazy sequences. For instance, you can define an infinite sequence of even numbers by filtering an infinite range with the `even?` function:
```clojure
(def evens (filter even? (range)))
(0 2 4 6 8 10 12 14 16 18) ; even integers from 0 to 18
```
Also note that `mapv` and `filterv` aren't lazy (such functions are called *eager*), so replacing `filter` by `filterv` above would lead to an infinite evaluation. 

**To stop Nightcode infinite evaluation**, use a **Stop** button in the REPL panel (on the right). You may need to click **Run with REPL** afterwards to get the REPL prompt back. 

Feel free to explore laziness more by studying [sequences functions in clojuredocs](https://clojuredocs.org/quickref). 

We will be using a finite range to compute frequencies of letters, but we will come back to potentially infinite lazy sequences in the next section on Vigenere cipher. 

However, for now we need to create a sequence of 26 English letters. Rather than typing it in, we will create a range of 26 first integers and then apply the `to-char` function that you wrote previously.  
```clojure
(def alphabet (map to-char (range 26)))
```  
You can use `map` or `mapv` here, it doesn't matter since we will be using the entire finite sequence anyway. 

Check that `alpahbet` does indeed have all characters of English alphabet in order. 

## Creating a hashmap of letter counts
Now we are all set to put together a function that takes a string and returns a hashmap of the number of times each letter appears in the string. 

Let's revisit the encrypted text we are trying to break, and save it in a variable:
```clojure
(def encr1 "radyjgtxhpsncpbxrvtctgpaejgedhtegdvgpbbxcvapcvjpvtrdbqxcxcv
iwtpeegdprwpqxaxinpcsxcitgprixktstktadebtciduphrgxeixcvapcvjp
vtlxiwpctuuxrxtcipcsgdqjhixcugphigjrijgtudgbjaixiwgtpstsegdvg
pbbxcvo")
```
We assume that you have the function `count-letters` and the variable `alphabet` defined as we discussed above.

You might want to revisit `mapv` and `zipmap` examples before working on this. 

**Exercise:** Construct a hashmap of the number of occurrences of each letter in the string `encr1`. Here is a sequence of steps that accomplishes the task:

1. Use `map` (or `mapv`) and `count-letters` to find the number of occurrences of each letter of the `alphabet` in the string `encr1`. If this sounds complicated, we are actually showing a solution for this below. 
2. Use `zipmap` on `alphabet` and the result of step 1 to "zip" the counts with the alphabet into a hash map.  

If it all works, you should get the following map:
```clojure
{\a 7, \b 8, \c 16, \d 10, \e 8, \f 0, \g 16, \h 5, \i 13, \j 8, \k 2, \l 1, \m 0, \n 2, \o 1, \p 19, \q 3, \r 8, \s 6, \t 17, \u 5, \v 11, \w 4, \x 17, \y 1, \z 0}
```

**Solution for step 1 of the previous exercise:**
We need to map over the alphabet, computing the counts for each letter. Thus we will be passing each letter to the anonymous function, on eby one, as the `%` parameter:
```clojure
(map #(count-letters % encr1) alphabet)
```

**Exercise:** Make your solution of the previous exercise into a function that takes a string and returns . Pass `enrc1` to it to make sure that it returns the right result. 

## Clojure sorting
*Relevant functions:* [sort-by](https://clojuredocs.org/clojure.core/sort-by), [last](https://clojuredocs.org/clojure.core/last), [reverse](https://clojuredocs.org/clojure.core/reverse)

**To-do** explain that the sequence of a hashmap consists of pairs. 

**To-do** sort the map on counts, `last`, `drop`

Alternatively, you can reverse the resulting list after you have sorted it, reverse it using the function `reverse`, and take the first item or the first few items to find likely key values. 

## Computing the cipher key

**To-do:** explain, but not show, the formula for the key. 

**Exercise:** Now try this approach on the string "ahixblmaxmabgzpbmayxtmaxklmatmixkvaxlbgmaxlhnetgwlbgzlmaxmngxpbmahnmmaxphkwltgwgxoxklmhiltmtee".

**Exercise:** Also, try uploading your own longer examples of encryption (but not the key) to slack and try figuring out the keys for other examples uploaded there. Make sure they are long enough for frequency analysis. Use only English texts. 

**Next:** [Vigenere cipher](track2-vigenere.md)
<br />
**Previous:** [Caesar cipher](track2-caesar.md)