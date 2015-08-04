**Table of Content**



# Introduction #

The present tutorial explains how to use the Lea package, through a sequence of simple examples executed in the **Leapp** console. Leapp is a small probabilistic programming language (PPL) based on Lea. It extends the syntax of Python programming language with special symbols (`?`, `$`, `!`, …). Since Leapp is basically an extension of Python syntax, any usual Python statement can also be executed ; also, Python modules can be imported and used as usual.

The goal of Leapp is to have a concise yet expressive syntax to play with probabilities. If you are not willing to learn new syntax or if you want to use Lea as an usual Python module to be imported in your application, then feel free to leave Leapp aside and jump to [Lea tutorial (Python flavor)](LeaPyTutorial.md).

_**For the impatient reader …**_ : just read the examples (light grey background)! These should be mostly self-explanatory if you are acquainted with probability distributions.

# Prerequisites #

In order to execute the statements given in the present tutorial, you have to install the Lea package. Instructions are provided in [Installation page](Installation.md). Then, to start the Leapp console, execute in a terminal (`%` is the prompt):

```
% python -m leapp
```

The Lea module version and copyright notice shall be displayed; then, the following prompt shall appear:

```
lea> 
```

From here, all the examples provided in the present tutorial can be executed. What follow the `lea>` prompt are the Leapp statements or expressions that _you_ type; then, below, the results (if any) of the execution are displayed.

Note that all demonstrated Leapp statements could also be embedded in a script file (e.g. `mytest.lea`), optionnaly mixed with standard Python statements. You can then can then execute such script by typing

```
% python -m leapp mytest.lea
```


# Defining probability distributions #

The first thing to do when working with Lea is to define a probability distribution that models a given phenomenon. There are essentially two ways to do that:
  * providing a sequence of equiprobable values
  * providing a sequence of pairs (values,probability)

## Equiprobable values ##

The simplest way to define a Lea distribution is by providing a sequence of values, each considered as equiprobable. The `?` prefix, that shall occur many times in the present tutorial, is meant to build a new probability distribution; when it is used in the `?(…)` syntax, the values passed between parentheses are taken as equiprobable. As an example, the following Leapp statement defines the result of coin flipping, assuming fair chances to get head or tail :

```
lea> flip = ?('Head','Tail')
```

If each value occurs no more than once in the sequence, then we get a uniform probability distribution: each value has the same occurrence probability, which is 1/_N_ where _N_ is the number of given values (N = 2, in the example). The distribution can be displayed simply by entering its name. This displays the different values, in sorted order, with their respective probabilities expressed as rational numbers:

```
lea> flip
Head : 1/2
Tail : 1/2
```

Note that Lea fully complies with the axioms of probability: each displayed probability is between 0 and 1, the total of probabilities displayed for a given distribution is equal to 1.

If the same value occurs several times in the definition sequence, then its probability is proportional to its number of occurrences. For instance, an unfair coin can be defined by the following statement:

```
lea> biasedFlip = ?('Head','Tail','Tail')
```

Since the value 'Tail' is provided twice, it has two more chances to occur than 'Head', which occurs only once:

```
lea> biasedFlip 
Head : 1/3
Tail : 2/3
```

We will show later in this tutorial that this feature of repeated values shall prove handy for determining frequencies of values in a given sample sequence.

In Leapp, any sequence of values (tuple, list, set, iterator, …), built with common Python syntax, can be passed after the question mark. Consider for example a fair die with six faces:

```
lea> die = ?(1,2,3,4,5,6)
lea> die
1 : 1/6
2 : 1/6
3 : 1/6
4 : 1/6
5 : 1/6
6 : 1/6
```

All the following statements are equivalent and create the same distribution:

```
lea> die = ?(range(1,7))
lea> die = ?([1,2,3,4,5,6])
lea> myList = [1,2,3,4,5,6]
lea> die = ?(myList)
lea> myTuple = (1,2,3,4,5,6)
lea> die = ?(myTuple)
```

or, using Python's list comprehension / generator expressions,

```
lea> die = ?([v+1 for v in range(6)])
lea> die = ?(v+1 for v in range(6))
lea> die = ?(6-v for v in range(6))
```

and we could continue examples with many other iterable objects, ad nauseam.

In the examples above, we have seen probability distributions with Python's strings and integers being the values having some chances to occur. Actually, there are no limitation on the types of object that can be handled: floats, complexes, vectors, date/times, user-defined objects, … can be set. However, depending of the object defined in the distributions, some operations can become impossible to achieve; if you try to execute such operation then an exception shall be raised. For example, we shall see later that you shall be able to get the mean value on the die distribution; it is impossible however to get a mean on the flip distribution, since there ain't such concept among character strings!

## Values with explicit probabilities ##

Another way to define Lea distributions is to provide explicit pairs (value,probability), using `?` followed by curly braces (`{…}`). For instance, the following statement is another way to define the unfair coin shown above.

```
lea> biasedFlip = ?{'Head': 1, 'Tail': 2}
```
or
```
lea> biasedFlip = ?{'Head': 1/3, 'Tail': 2/3}
```

This way of doing is especially handy for non-uniform definitions. Note that, as the syntax suggests it, a native Python dictionary can be passed also:

```
lea> myPythonDict = {'Head': 1, 'Tail': 2}
lea> biasedFlip = ?{myPythonDict}
```

To be more expressive, the probabilities can be given as floating numbers, percentages or fractions. Consider for example an unfair coin, which has 65 % probability to be head when tossed:

```
lea> biasedFlip2 = ?{'Head': 65, 'Tail': 35}
lea> biasedFlip2
Head : 13/20
Tail :  7/20
```

The following statements are exactly equivalent:

```
lea> biasedFlip2 = ?{'Head': 65%, 'Tail': 35%}
lea> biasedFlip2 = ?{'Head': 0.65, 'Tail': 0.35}
lea> biasedFlip2 = ?{'Head': 65/100, 'Tail': 35/100}
```

Note that the displayed probabilities are always simplified to have the lowest common denominator. We shall see later how to display the probabilities as percentages. Note also that values given with probability 0, if any, are removed automatically:

```
lea> ?{'Head': 1, 'Tail': 99, 'SHAZAM!': 0}
Head :  1/100
Tail : 99/100
```

Lea provides a couple of convenience methods for a couple of standard probability distributions, viz. boolean, Bernoulli, binomial and Poisson. These are described hereafter.

## Random boolean variables ##

Based on what has been seen above, you can model random boolean variable by defining the probabilities of True and False values, such as the total is 1. For example, you could model a probability of 27 % that it shall rain tomorrow as

```
lea> rain = ?{True: 27%, False: 73%}
lea> rain
False : 73/100
 True : 27/100
```

Note that there is of course a redundancy in such display since the probability of False can be calculated from the probability of True. We shall see later a more handy way to display probabilities of random variables.

Since boolean variables are very common, Leapp provides a more concise notation to define such variable:

```
lea> rain = ?:(27%)
```

The probability can also be expressed using floating-point number or fraction:

```
lea> rain = ?:(0.27)
lea> rain = ?:(0.27e0)
lea> rain = ?:(27/100)
```

### Bernoulli distribution ###

The method `Lea.bernoulli(n,d)` defines the value 1 with probability _n_/_d_ and 0 with probability 1 - _n_/_d_

```
lea> Lea.bernoulli(3,10)
0 : 7/10
1 : 3/10
```

### Binomial distribution ###

The method `Lea.binomial(x,n,d)` defines the number of successes among a number _x_ of independent experiments, each having probability _n_/_d_ of success (see [binomial distributions](http://en.wikipedia.org/wiki/Binomial_distribution)). For example, suppose a biased coin comes up heads with probability 0.3 when tossed. What is the probability of achieving 0, 1,…, 6 heads after six tosses?

```
lea> Lea.binom(6,3,10)
0 : 117649/1000000
1 : 302526/1000000
2 : 324135/1000000
3 : 185220/1000000
4 :  59535/1000000
5 :  10206/1000000
6 :    729/1000000
```

Note that `Lea.binomial(1,n,d)` is equivalent to `Lea.bernoulli(n,d)`

### Poisson distribution ###

The method `Lea.poisson(m)` defines a Poisson probability distribution having mean _m_.

```
lea> Lea.poisson(2)
 0 :  6766764161830635520/50000000000000006339
 1 : 13533528323661271040/50000000000000006339
 2 : 13533528323661271040/50000000000000006339
 3 :  9022352215774180352/50000000000000006339
 4 :  4511176107887090176/50000000000000006339
 5 :  1804470443154836224/50000000000000006339
 6 :   601490147718278656/50000000000000006339
 7 :   171854327919508192/50000000000000006339
 8 :    42963581979877048/50000000000000006339
 9 :     9547462662194900/50000000000000006339
10 :     1909492532438980/50000000000000006339
11 :      347180460443451/50000000000000006339
12 :       57863410073908/50000000000000006339
13 :        8902063088294/50000000000000006339
14 :        1271723298328/50000000000000006339
15 :         169563106444/50000000000000006339
16 :          21195388305/50000000000000006339
17 :           2493575095/50000000000000006339
18 :            277063899/50000000000000006339
19 :             29164621/50000000000000006339
20 :              2916462/50000000000000006339
21 :               277758/50000000000000006339
22 :                25251/50000000000000006339
23 :                 2196/50000000000000006339
24 :                  183/50000000000000006339
25 :                   15/50000000000000006339
26 :                    1/50000000000000006339
```

Because the Poisson distribution has, in theory, a infinite number of possible values, Lea provides an approximation where values having a probability below 1e-20 are dropped. To keep a probability total equal to 1, the remaining probabilities are all slightly overestimated. The threshold can be changed by issuing the needed precision as a second argument, e.g. `Lea.poisson(2,1e-30)`.

# Displaying probabilities #

As seen in the previous examples, a probability distribution displays itself as rows giving the possible values with their respective probabilities expressed as fractions. This has the advantage to give exact probability values, without rounding errors (as found in many probability packages) and with a probability total guaranteed to be 1. However, in several cases, such value may be difficult to interpret ; also, many examples found in literature use floating-point notation, which are painful to compare with fractions.

Lea provides two display alternatives for probability values : decimals and percentages. These are accessible by starting Leapp expression with a colon (`:`), roughly equivalent to Python's `print`, followed by a character specifying the display mode:

```
lea> :. biasedFlip
Head : 0.333333
Tail : 0.666667
lea> :% biasedFlip
Head :  33.3 %
Tail :  66.7 %
```

A standard way to display a probability distribution is to draw its histogram. Lea does not have real plotting features. However, a modest text-oriented histogram is provided through the `:-` syntax (available as of 2.0.0 beta 5):

```
lea> :- die
1 :  -----------------
2 :  -----------------
3 :  -----------------
4 :  -----------------
5 :  -----------------
6 :  -----------------
lea> :- biasedFlip
Head :  ---------------------------------
Tail :  -------------------------------------------------------------------
```

The histogram can be augmented with numerical probability values, by putting a symbol `/`, `.` or `%` before the `-` :

```
lea> :/- biasedFlip
Head : 1/3 ---------------------------------
Tail : 2/3 -------------------------------------------------------------------
lea> :.- biasedFlip
Head : 0.333333 ---------------------------------
Tail : 0.666667 -------------------------------------------------------------------
```


# Getting individual probabilities #

Given a probability distribution, the probability of a given value can be extracted by using the `@` operator

```
lea> biasedFlip@'Head'
1/3
lea> die@5
1/6
```

The returned value is a `ProbFraction` instance, an object inheriting Python's `Fraction` class and which displays itself as a fraction. To display this probability in other formats, use the "colon" facilties seen before:

```
lea> :. die@5
0.16666666666666666
lea> :% die@5
16.666667 %
```

Note that, if the required value results from an expression instead of a simple litteral, parentheses shall be used:

```
lea> biasedFlip@('H'+'ead')
1/3
lea> die@(2*3)
1/6
```

To get a genuine floating-point number (instead of just display it), you can use Python's `float` function or Lea's `pmf` method:

```
lea> p5 = float(biasedFlip@'Head')
 #or p5 = biasedFlip.pmf('Head')
lea> p5
0.3333333333333333
```

For boolean random variables, which are ubiquitous, `@True` can be abbreviated as `@`:

```
lea> rain@
27/100
lea> :. rain@
0.27
lea> :% rain@
27.000000 %
```

As alternative, useful for a longer boolean expression, the at `@` can be put as suffix of `:`:

```
lea> :@ rain
27/100
lea> :.@ rain
0.27
lea> :%@ rain
27.000000 %
```

# Getting probability distribution data #

Consider the following height distribution of a sample of smurfs:

```
lea> heights = ?{0.5: 5%, 1.0: 10%, 1.5: 20%, 2.0: 25%, 2.5: 25%, 3.0: 10%, 3.5: 5%} 
lea> :%- heights
0.5 :   5.000000 % -----
1.0 :  10.000000 % ----------
1.5 :  20.000000 % --------------------
2.0 :  25.000000 % -------------------------
2.5 :  25.000000 % -------------------------
3.0 :  10.000000 % ----------
3.5 :   5.000000 % -----
```

Lea provides several methods to get the distribution values and probabilities at once. The most general method is `vps`, which returns (value,probability weight) pairs in a tuple:

```
lea> heights.vps()
((0.5, 1), (1.0, 2), (1.5, 4), (2.0, 5), (2.5, 5), (3.0, 2), (3.5, 1))
```

Methods are available also to get the the values of a given distribution, its p.m.f. (probability mass function) and its c.d.f (cumulative distribution function):

  * `vals` (or `support`): getting values only:
```
lea> heights.vals()
(0.5, 1.0, 1.5, 2.0, 2.5, 3.0, 3.5)
lea> heights.support()
(0.5, 1.0, 1.5, 2.0, 2.5, 3.0, 3.5)
```

  * `pmf` without argument: getting probabilities only, as floating-point values
```
lea> heights.pmf()
(0.05, 0.1, 0.2, 0.25, 0.25, 0.1, 0.05)
```

  * `cdf` without argument: getting cumulative distributive function
```
lea> heights.cdf()
(0.05, 0.15, 0.35, 0.6, 0.85, 0.95, 1.0)
```
Note that the last value returned by `cdf` is always 1.0.

You can use these methods, for example, to plot histograms using your prefered Python chart package.

**Important**: the methods `vps`, `pmf` and `cdf` without argument returns tuples with an order consistent with the order of values returned by `vals`. For instance, the first  values of `pmf` corrsponds to the first value of `vals`, etc. What is this order? This depends on the values:
  * if the values are sortable, then the values are sorted in increasing order (e.g. integers, floats, strings, …)
  * if the values are not sortable, then an arbitrary order, consistent from call to call, is used (e.g. complexes, matrices, …)
Note that the order you use to pass your sequence of values to build a Lea distribution is irrelevant. If these values can't be sorted, then they will most probably be mixed up, similarily to what happens in Python's dictionaries and sets.

`pmf` and `cdf` can be called with a value argument: then, the method returns the probability of pmf or cdf for the given value.

```
lea> heights.pmf(2.0)
0.25
lea> heights.cdf(2.0)
0.6
```

Note that these values can be found also by using comparison operators:
```
lea> :.@ heights == 2.0
0.25
lea> :.@ heights <= 2.0
0.6
```


# Probability distribution indicators #

There exist several Lea attributes to get standard indicators of probability distributions: mean, variance, standard deviation, mode and entropy. The three first attributes are demonstrated in the following examples:

```
lea> die = ?(1,2,3,4,5,6)
lea> die.mean
3.5
lea> die.var
2.9166666666666665
lea> die.std
1.707825127659933
```

Note that these three indicators require that 1° the values can be subtracted together, 2° the differences can be multiplied by floating-point numbers and 3° the weighted differences can be added to the values. These conditions are verified, among others, for distributions having values with numeric, matrix and datetime types. These are NOT verified, among others, for strings, functions, classes, objects without overloaded operations.

The mode attribute returns a tuple giving the value(s) having the highest probability:

```
lea> die.mode
(1, 2, 3, 4, 5, 6)
lea> heights = ?{0.5: 5%, 1.0: 10%, 1.5: 20%, 2.0: 25%, 2.5: 25%, 3.0: 10%, 3.5: 5%} 
lea> heights.mode
(2.0, 2.5)
```

This indicator requires of course that the values can be compared together.

The entropy provides a measure of the degree of randomness of the distribution. For a given set of values, it is maximum for a uniform probability distribution (i.e. all values have equal probabilities); in this case, it is equal to the logarithm in base 2 of the number of values, the result is expressed in bits.  The entropy method is illustrated in the following statements.

```
lea> ?:(50%).entropy
1.0
lea> die.entropy
2.584962500721156
lea> dieU = ?(1,2,3,4,5,6,6)
lea> dieU.entropy
2.521640636343318
```

You can verify that the entropy value of the loaded die (`dieU`) is lower than the fair die's (`die`) : the chances that the loaded die gives a 6 are twice than the chances of other numbers; so it is "less random" than the fair die.

The entropy is null for distributions having a certain, unique, value (i.e. with probability = 1).

```
lea> ?:(100%).entropy
0.0
lea> ?:(0%).entropy
0.0
lea> ?(1,).entropy
0.0
```

Note that the entropy calculation does not involve the values of the distribution; only the probabilities play a role. So, contrarily, to other indicators seen above, the entropy can _always_ be calculated, whatever the type of values.


# Generation of random samples #

Since Lea copes with randomness, you can legitimately expect means to generate random samples! Leapp provides the `$` operator for this purpose.

Here is how to draw a random value from the fair die defined above:

```
lea> die $
3
```

You could interpret this as: "Append a `$` after a probability distribution and you get the associated _random variable_!"

Note: since we cope with randomness, the actual values you get here (as well as in the other examples of the present section) may be different! To generate a sample with a given number of random values, you provide the required sample's size between parentheses:

```
lea> die $(20)
(5, 6, 3, 3, 3, 4, 4, 5, 1, 2, 1, 1, 3, 4, 4, 2, 2, 2, 1, 5)
```

The distribution of random values shall of course respect the given probability distribution. Let us make a random sample from the unfair die defined above.

```
lea> dieU $(20)
(6, 1, 6, 4, 5, 1, 2, 4, 6, 6, 3, 6, 5, 4, 6, 1, 6, 6, 4, 3)
```

We see that the value 6 _seems_ to occur more often, as we could have expected it. Generally speaking, as the random sample size grows, the frequency of occurrence of each value is expected to match closer and closer the probability distribution. In order to measure this trend - and to verify the correctness of Lea's random generator -, the Leapp `?` constructor can be used as a frequency counter. To illustrate this, let us generate a sample of 42,000 random values with the fair die and let us count the frequency of each value.

```
lea> sample1 = die $(42000)
lea> ?(sample1)
1 : 6933/42000
2 : 6942/42000
3 : 7059/42000
4 : 6956/42000
5 : 7039/42000
6 : 7071/42000
```

We notice that the frequencies are all close to 7000/42000, which is 1/6. So far, so good!

Now, let us proceed similarly with the unfair die.

```
lea> sample2 = dieU $(42000)
lea> ?(sample2)
1 :  5869/42000
2 :  5958/42000
3 :  6050/42000
4 :  6100/42000
5 :  6112/42000
6 : 11911/42000
```

You can verify here that the frequencies of 1, 2, 3, 4 and 5 are close to 1/7, while the frequency of 6 is close to 2/7.

The random sampling works with any distribution, whatever the type of values. Here is an example with tossing of an unfair coin.

```
lea> biasedFlip = ?{'Head': 1/3, 'Tail': 2/3}
lea> :. biasedFlip
Head : 0.333333
Tail : 0.666667
lea> biasedFlip $(10)
('Tail', 'Tail', 'Tail', 'Tail', 'Tail', 'Head', 'Tail', 'Head', 'Tail', 'Tail')
lea> :. ?(biasedFlip$(100000))
Head : 0.333200
Tail : 0.666800
```

You can verify that the frequency of Head in the 100,0000 tosses sample is close to the defined probability.

Certain stochastic processes require that a given value does not appear twice in a random sample. These are named "draws without replacement", which include many lottery games. The `$_` syntax is dedicated to such processes. As an example, imagine a fair lottery game with 45 balls, from which 6 balls shall be drawn.

```
lea> lottery = ?(range(1,46))
lea> lottery $_(6)
(13, 24, 25, 10, 1, 26)
```

This method guarantees that all drawn values are different. As a special case, if the argument of this method is the number of values, we get a random permutation of all the values. Note that this can be done also by omitting the argument:

```
lea> lottery $_(45)
(44, 42, 36, 4, 10, 2, 14, 19, 22, 39, 12, 41, 18, 7, 31, 5, 38, 45, 21, 8, 13, 1, 3, 23, 32, 35, 20, 40, 11, 29, 27, 15, 30, 37, 16, 28, 43, 17, 6, 34, 25, 24, 9, 33, 26)
lea> lottery $_()
(40, 41, 36, 27, 15, 26, 31, 24, 9, 7, 21, 16, 13, 2, 10, 34, 35, 30, 29, 33, 3, 23, 42, 32, 37, 45, 22, 20, 5, 25, 17, 43, 28, 4, 18, 39, 14, 1, 11, 8, 19, 12, 6, 38, 44)
```

Should the argument exceed the number of values, then an exception is raised.


# Calculus with probability distributions #

## Context ##

The examples seen so far work with explicitly defined probability distributions. In many situations however, a stochastic phenomenon results from the combination of other stochastic phenomenon's, which are easier to model and which have known probability distributions. Examples are:

  * adding the results of _N_ thrown dice,
  * counting the number of successes of _N_ trials,
  * evaluating the truth of a given condition made on stochastic events,
  * evaluating joint occurrences of _N_ stochastic events,
  * applying a given function on a random variable,
  * calculating a given function with random variables as arguments.

The traditional approach requires detailed calculations, made manually or by customised programs; this is tedious and error-prone. Fortunately, Lea has several features to make these derivations of probability distribution easy and generic.

The following sections shall present the most basic techniques, assuming that

> _all random events are independent_.

This is an important assumption to take in mind. It allows for using simple methods to derive probabilities; in particular, it is usually well suited to model gambling situations or stateless systems.

For more complex situations, the random events at hand cannot be assumed independent. The occurrence of one random event may influence the occurrence of another random event. In general terms, the knowledge of some information can impact the probability distribution of future (or unknown) events. Lea provides some tools for that situations. These are presented in an "advanced tutorial", which covers topics like joint probability, conditional probability and Bayesian inference.

## Arithmetic operations ##

We shall see here how Lea instances can be put in arithmetic expressions to produce new Lea instances. The most basic cases involve expressions mixing one Lea instance with constant values. In such cases, the independence assumption is obviously true since a constant is something certain.

Here are some examples:

  * Subtracting 3 from the result of a die:
```
lea> die1 = ?(1,2,3,4,5,6)
lea> die1 - 3
-2 : 1/6
-1 : 1/6
 0 : 1/6
 1 : 1/6
 2 : 1/6
 3 : 1/6
```

  * Doubling the result of a die
```
lea> 2 * die1
 2 : 1/6
 4 : 1/6
 6 : 1/6
 8 : 1/6
10 : 1/6
12 : 1/6
```

  * Dividing the result of a die by 3, using floating-point arithmetic
```
lea> die1 / 3
0.333333333333 : 1/6
0.666666666667 : 1/6
           1.0 : 1/6
 1.33333333333 : 1/6
 1.66666666667 : 1/6
           2.0 : 1/6
```

For Python 2.x users, note that Lea (from ver 1.3) uses the "true division", as Python 3. For integer division, use double slash as in the example below.

  * Dividing the result of a die by 3, truncated to integers
```
lea> die1 // 3
0 : 2/6
1 : 3/6
2 : 1/6
```

  * Taking the residue of a die result modulo 2 (i.e. the parity bit)
```
lea> die1 % 2
0 : 1/2
1 : 1/2
```

  * Adding the result of a die with itself
```
lea> die1 + die1
 2 : 1/6
 4 : 1/6
 6 : 1/6
 8 : 1/6
10 : 1/6
12 : 1/6
```

Note that the last expression returns the same distribution as `2 * die1`.

Now, to get the distribution resulting from the sum of two dice, you have to create a new die instance, by issuing a new `?(1,2,3,4,5,6)` expression or, simpler, by "cloning" the first die with a `?` prefix:

```
lea> die2 = ?die1
```

From now, `die2` represents a second die having the same probability distribution as `die1` but its throw is assumed to be an event independent of the throw of the first die. Then, the following statement produces the distribution of the sum of the two dice.

```
lea> die1 + die2
 2 : 1/36
 3 : 2/36
 4 : 3/36
 5 : 4/36
 6 : 5/36
 7 : 6/36
 8 : 5/36
 9 : 4/36
10 : 3/36
11 : 2/36
12 : 1/36
```
which can be displayed as the well-known triangular histogram :
```
lea> :/- die1 + die2
 2 : 1/36 ---
 3 : 2/36 ------
 4 : 3/36 --------
 5 : 4/36 -----------
 6 : 5/36 --------------
 7 : 6/36 -----------------
 8 : 5/36 --------------
 9 : 4/36 -----------
10 : 3/36 --------
11 : 2/36 ------
12 : 1/36 ---
```

Actually, as you could expect it, the expression has been used to browse all the combination pairs from two dice values (an operation known as the _cartesian product_); these pairs have been reduced by addition and the occurrences of each sum value has been counted.

Other similar expressions may be used to calculate the distribution of the difference of two dice, their product, etc. A more sophisticated example below calculates the square of the difference of the two thrown dice.

```
lea> (die1-die2)**2
 0 : 3/18
 1 : 5/18
 4 : 4/18
 9 : 3/18
16 : 2/18
25 : 1/18
```

Note that the same Lea instance may appear at several places in the same expression. The values are processed consistently, as shown in the following:

```
lea> die1 + 2*die2 - die1 - (die2 + die2)
0 : 1
```

## Repeating operations ##

In some situations, you will want to repeat an operation a given number of times, like throwing _N_ dice, and adding the results.

For instance, what is the probability of getting a total of 13 when throwing 4 dice?

From the techniques seen so far, you could write:

```
lea> (die1 + ?die1 + ?die1 + ?die1)@13
35/324
```

This is of course tedious to type. Here is an alternative, shown essentially to demonstrate that Lea integrates smoothly with Python's building blocks:

```
lea> diexN = sum(?die1 for i in range(4))
lea> diexN@13
35/324
```

Now, to alleviate the job, Leapp provides a dedicated syntax:

```
lea> ?[4]die1
 4 :   1/1296
 5 :   4/1296
 6 :  10/1296
 7 :  20/1296
 8 :  35/1296
 9 :  56/1296
10 :  80/1296
11 : 104/1296
12 : 125/1296
13 : 140/1296
14 : 146/1296
15 : 140/1296
16 : 125/1296
17 : 104/1296
18 :  80/1296
19 :  56/1296
20 :  35/1296
21 :  20/1296
22 :  10/1296
23 :   4/1296
24 :   1/1296
lea> ?[4]die1@13
35/324
```

This emulates the distribution of the sum of 4 thrown dice; then it returns the same result as the previous expression. Note that this is not only shorter to write but also _faster to execute_ because the method uses a dynamic programming technique, that is a dichotomic algorithm with far less calculations than the former naive expression. To illustrate that point, you may notice that Lea has no particular problem to calculate the distribution resulting of 50 thrown dice:

```
lea> ?[50]die1
 50 :                                      1/808281277464764060643139600456536293376
 51 :                                     50/808281277464764060643139600456536293376
 52 :                                   1275/808281277464764060643139600456536293376
 53 :                                  22100/808281277464764060643139600456536293376
 54 :                                 292825/808281277464764060643139600456536293376
…
```

Note that the method for binomial distribution (see above) is no more than a shortcut to avoid to use `?[n]` method explicitly. So,

```
lea> Lea.binom(6,3,10)
```

is equivalent to

```
lea> bc = ?{0: 0.7, 1: 0.3}
lea> ?[6]bc
```

By default, the given syntax performs additions. Any other two-ary operator or two arguments function can be specified as a second argument. For instance, here is how you can get the probability that the _product_ of 4 thrown dice is 5:

```
lea> from operator import mul
lea> ?[4,mul]die1@5
1/324
```

_Note : This probability of 1/324 can be checked by noticing that 5 is a prime number : this means that it shall be reached only by one of 4 permutations of the form 1x1x1x5; hence the probability is 4/6`*``*`4 = 1/324._


## Comparison operations ##

Usual comparison operators may be used on any Lea instances, resulting in Boolean distributions giving the probabilities of each truth value.

```
lea> die1 == 4
False : 5/6
 True : 1/6
lea> die1 <= 4
 False : 1/3
  True : 2/3
```

To get only the probability value that the expression is true, you may use the display facilities seen earlier:

```
lea> :@ die1 == 4
1/6
lea> :@ die1 <= 4
2/3
lea> :.@ die1 <= 4
0.6666666666666666
```

Comparison operators may be used in expressions, as shown in the following examples involving the two independent dice.

```
lea> :@ die1 < die2
5/12
lea> :@ die1 <= die2
7/12
lea> :@ die1 == die2
1/6
lea> :@ die1 != die2
5/6
```

The first example (`die1 < die2`) shows that, after throwing two dice, there are 5/12 chances that one _given_ die has a value strictly less than the other's. This assumes that you are able to distinguish between the two dice, for example using colors (e.g. green die + blue die). If you make no difference between dice, then the probability that one of them is strictly less than the other is 5/6, as shown in the last example.

As more advanced examples, the following expressions involve multiple occurrences of `die1` and `die2`.

```
lea> :@ (die1-die2)**2 == die1**2 - die2**2
1/6
lea> :@ (die1-die2)**2 == die1**2 + die2**2
0
lea> :@ (die1-die2)**2 == die1**2 + die2**2 - 2*die1*die2 
1
```

Let us remark that the last expression is evaluated as certainly true, as we can expect from the well-known algebra rule.

For evaluating the probability of a range of values, you can put two inequalities connected with a logical _and_ (see next section).

For testing the equality of distribution against a set of given values, the `isAnyOf(…)` shall be used. As an example, here is how to calculate the probability of a "craps":

```
lea> :@ (die1+die2).isAnyOf(2,3,12)
1/9
```

The method `isNoneOf(…)` calculates the inverse distribution, that is the probability of having none of the values in a given set:
```
lea> :@ (die1+die2).isNoneOf(2,3,12)
8/9
```

## Logical operations ##

Boolean probability distributions (as obtained for example by using comparison operator, as seen just above) can be combined by usual logical operators.

These are detailed in the following table, where _a_ and _b_ are standing for expressions returning Lea boolean distributions or simple Python booleans:

| **Lea expression** | **meaning** |
|:-------------------|:------------|
| ~_a_               | not _a_     |
| _a_ & _b_          | _a_ and _b_ |
| _a_ | _b_          | _a_ or _b_  |
| _a_  ^ _b_         | _a_ xor _b_ |

Examples:
```
lea> :@ ~(die1 == 3)
5/6
lea> :@ ~(die1 == 3) | (die1 == 3)
1
lea> :@ (die1 == 4) & (die2 == 2)
1/36
lea> :@ (die1 <= 3) & (die2 > 3)
1/4
```

WARNINGS:

  * Due to Python precedence rules, it is extremely important to embed expressions connected by logical operators with parentheses. If parentheses are missing, an exception shall be raised.

  * The Python's augmented comparison between Lea instances (e.g. _a_ < _b_ < _c_) DOES NOT WORK! Actually, for several technical reasons related to Python design, such expressions do not (and cannot) return any sensible result. These can be replaced by using a logical AND (e.g. (_a_ < _b_) & (_b_ < _c_) ).


## Applying functions ##

If you need special processing beyond arithmetic or logical operator, you can apply a function on the values of a probability distribution; this builds a new probability distribution on the function results. This is done very easily by prefixing the function to call with a `?`. For instance, this is how to transform the integer values of a die into floating point numbers by invoking the standard Python `float` function:

```
lea> ?float(die1)
1.0 : 1/6
2.0 : 1/6
3.0 : 1/6
4.0 : 1/6
5.0 : 1/6
6.0 : 1/6
```

You could of cause use your own functions:

```
lea> def parity(x):
 ...     return "odd" if x%2==1 else "even"
lea> ?parity(die1)
even : 1/2
 odd : 1/2
```

and embed calls

```
lea> ?len(?parity(die1))
3 : 1/2
4 : 1/2
```

The last examples show that the number of values of the resulting distributions can be lower than the one in the distribution passed to the function. This is normal since several values can be mapped to the same resulting value.

If you prefer a more functional style, note that you could also use the `map` method on any Lea instance.

```
lea> die1.map(parity)
even : 1/2
 odd : 1/2
lea> die1.map(parity).map(len)
3 : 1/2
4 : 1/2
```

To conclude, here is a more interesting example, showing multiple arguments functions : the goal is to determine the probability distribution that results from keeping the die having the smallest value among two thrown dice.

```
lea> ?min(die1,die2)
1 : 11/36
2 :  9/36
3 :  7/36
4 :  5/36
5 :  3/36
6 :  1/36
```

and here is what happens when keeping the die having the greatest value:

```
lea> ?max(die1,die2)
1 :  1/36
2 :  3/36
3 :  5/36
4 :  7/36
5 :  9/36
6 : 11/36
```


# What's next? #

Thank you for reading the present tutorial!

You should be able now to define your own probability distributions and combine them to model many phenomenon's with uncertainties, assuming independence of the random events. You can find more examples in the [Examples](LeappExamples.md) page of the Wiki.

If you are looking for special topics, like draws without replacement, joint distributions, marginalisation, conditional probabilities, then jump to [Lea advanced tutorial: Part 1](LeappTutorial1.md).

Please send your comments, critics, suggestions, bug reports,… by E-mail to pie.denis@skynet.be . Thanks!

