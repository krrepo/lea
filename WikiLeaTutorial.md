
---


---


---

**_WARNING: The present page is no longer maintained!_**

**_You are invited to jump to [the new version](LeaPyTutorial.md)._**

---


---


---




# Introduction #

The present tutorial explains how to use the Lea package, through a sequence of simple examples executed in the Python interpreter.

_**For the impatient reader ...**_ : just read the examples (light grey background)! These should be mostly self-explanatory if you are acquainted with probability distributions.

# Prerequisites #

In order to execute the statements given in the present tutorial, you have to install the Lea package. Instructions are provided in [Installation page](Installation.md).

Then, start the python interpreter and type

```
>>> from lea import *
```

From now, all the examples provided in the present tutorial can be executed.

All the presented examples are executed using the Python interpreter. The Python statements follow the ">>>" prompt, then the results (if any) of the execution are displayed. The demonstrated statements could of course be embedded in Python script files, to develop an application or a custom probability toolkit.

# Defining probability distributions #

The first thing to do when working with Lea is to define a probability distribution that models a given phenomenon.

## Sequence of values ##

The simplest way to define a Lea distribution is by providing a sequence of values, using the Lea.fromVals(...) method; each value is considered as equiprobable. As an example, the following statement defines a Lea instance representing a fair die with 6 faces:

```
>>> die = Lea.fromVals(1,2,3,4,5,6)
```

If each value occurs no more than once in the sequence, then we get a uniform probability distribution: each value has the same occurrence probability, which is 1/_N_ where _N_ is the number of given values. The distribution can be displayed simply by entering its name. This displays the different values, in sorted order, with their respective probabilities expressed as rational numbers:

```
>>> die
1 : 1/6
2 : 1/6
3 : 1/6
4 : 1/6
5 : 1/6
6 : 1/6
```

If some values occur several times as arguments of Lea.fromVals, then their probability is proportional to their number of occurrences in the sequence. For instance, an unfair die can be defined by the following statement:

```
>>> dieU = Lea.fromVals(1,2,3,4,5,6,6)
```

Since the value 6 is provided twice, it has 2 more chances to give 6 than any other values.

```
>>> dieU
1 : 1/7
2 : 1/7
3 : 1/7
4 : 1/7
5 : 1/7
6 : 2/7
```

We will show later in this tutorial that this feature of repeated values shall prove handy for determining frequencies of values in a given sample sequence.

As seen above, the Lea.fromVals method has a multiple arguments signature. Any non-empty sequence of values may be used also, provided that it is prefixed by an asterisk (`*`) character (this is a standard Python feature). For instance, the following statements create the same distribution as the fair die defined above:

```
>>> die = Lea.fromVals(*(1,2,3,4,5,6))
>>> die = Lea.fromVals(*[1,2,3,4,5,6])
>>> die = Lea.fromVals(*range(1,7))
```

or, with list comprehension / generator expressions,

```
>>> die = Lea.fromVals(*[v+1 for v in range(6)])
>>> die = Lea.fromVals(*(v+1 for v in range(6)))
>>> die = Lea.fromVals(*(6-v for v in range(6)))
```


## Sequence of weighted values ##

Another way to define Lea distributions is to provide explicit pairs (value,weight). This is the purpose of the method Lea.fromValFreqs, which handles such pair tuples as arguments. For instance, the following statement is another way to define the unfair die show above.

```
>>> dieU = Lea.fromValFreqs((1,1),(2,1),(3,1),(4,1),(5,1),(6,2))
```

This way of doing is especially handy for non-uniform definitions, which could have probabilities defined as percentages. Consider for example an unfair coin, which has 65 % probability to be head when tossed:

```
>>> coinU = Lea.fromValFreqs(("HEAD",65),("TAIL",35))
>>> coinU
HEAD : 13/20
TAIL :  7/20
```

Note that the displayed probabilities are always simplified to have the lowest common denominator.

As explained for fromVals method, any sequence or expression returning a sequence can be used as argument of fromValFreqs method, by using the asterisk prefix trick.

A boolean probability distribution can be defined by using the Lea.boolProb method. For example, the following statement models a proposition that has a 1/20 probability to be true:

```
>>> b = Lea.boolProb(1,20)
>>> b
False : 19/20
 True :  1/20
```

The two arguments shall be the numerator and denominator (naturals) of the probability of the true truth value. This is basically a convenience method that is equivalent to the statement

```
>>> b = Lea.fromValFreqs((True,1),(False,19))
```

We have seen that the Lea.fromValFreqs method expects a sequence of weighted value pairs. As an alternative, the Lea.fromValFreqsDict method allows you to define the weighted value as a dictionary. For instance, the following statement builds the same distribution for the unfair die seen above:

```
>>> dieU = Lea.fromValFreqsDict({1:1,2:1,3:1,4:1,5:1,6:2})
```

The usage of Lea.fromValFreqs vs Lea.fromValFreqsDict is essentially a matter of convenience. Note however that Lea.fromValFreqs is able to handle multiple occurrences of the same value (by adding their weights); this is unfeasible in dictionary since the values are keys.

# Getting individual probabilities #

As seen in the previous examples, any distribution required to be displayed shows all the possible values, with their respective probabilities expressed as rational numbers.

As alternative, the asPct(_n_) method displays the probabilities as percentage values, with _n_ decimals (default = 1):

```
>>> print (die1.asPct())
1 :  16.7 %
2 :  16.7 %
3 :  16.7 %
4 :  16.7 %
5 :  16.7 %
6 :  16.7 %
>>> print (die1.asPct(10))
1 :  16.6666666667 %
2 :  16.6666666667 %
3 :  16.6666666667 %
4 :  16.6666666667 %
5 :  16.6666666667 %
6 :  16.6666666667 %
```

Lea provides also methods to get the specific probability of one given value _v_:
  * p(_v_) returns the string with probability of _v_ expressed as a rational number;
  * `_`p(_v_) returns this rational number as a tuple (numerator,denominator);
  * pf(_v_) returns this rational number as a (maybe approximated) floating-point number

Example:

```
>>> die1.p(2)
'1/6'
>>> die1._p(2)
(1, 6)
>>> die1.pf(2)
0.16666666666666666
```


# Operations on distributions #

## Distribution indicators ##

There exist several Lea methods to calculate standard indicators of probability distributions: mean, variance, standard deviation and entropy. The three first methods are demonstrated in the following examples:

```
>>> die = Lea.fromVals(1,2,3,4,5,6)
>>> dieU = Lea.fromVals(1,2,3,4,5,6,6)
>>> die.mean()
3.5
>>> dieU.mean()
3.857142857142857
>>> die.variance()
2.9166666666666665
>>> die.stdev()
1.707825127659933
```

Note that these three methods require that 1° the values can be subtracted together, 2° the differences can be multiplied by floating-point numbers and 3° the weighted differences can be added to the values. These conditions are verified, among others, for distributions having values with numeric, matrix and datetime types. These are NOT verified, among others, for strings, functions, classes, objects without overloaded operations.

The entropy provides a measure of the degree of randomness of the distribution. For a given set of values, it is maximum for a uniform probability distribution (i.e. all values have equal probabilities); in this case, it is equal to the logarithm in base 2 of the number of values. The entropy is null for distributions having a certain, unique, value (i.e. with probability = 1). The entropy method is illustrated in the following statements.

```
>>> die.entropy()
2.584962500721156
>>> dieU.entropy()
2.521640636343318
>>> Lea.fromVals(1).entropy()
0.0
>>> Lea.fromVals(True).entropy()
0.0
```

You can verify that the entropy value of the loaded die (_dieU_) is lower than the fair die's (_die_), as it is expected. The chances that the loaded die gives a 6 are twice than the chances of other numbers; so it is "less random" than the fair die.

Note that the entropy calculation does not involve the values of the distribution; only the probabilities play a role. So, contrarily, to other indicators seen above, the entropy can _always_ be calculated, whatever the type of values.


## Generation of random samples ##

From a given Lea distribution, one can easily generate random samples by using the random(...) method.

Here is how to draw a random value from the fair die defined above:

```
>>> die.random()
3
```

Note: the actual values you get here (as well as in the other examples of the present section) may be different!

To generate a sample with a given number of random values, you provide the required sample's size as argument:

```
>>> die.random(20)
(5, 6, 3, 3, 3, 4, 4, 5, 1, 2, 1, 1, 3, 4, 4, 2, 2, 2, 1, 5)
```

Let us make a random sample from the unfair die defined above.

```
>>> dieU.random(20)
(6, 1, 6, 4, 5, 1, 2, 4, 6, 6, 3, 6, 5, 4, 6, 1, 6, 6, 4, 3)
```

We see that the value 6 _seems_ to occur more often, as we could have expected it.

Generally speaking, as the random sample size grows, the frequency of occurrence of each value is expected to match closer and closer the probability distribution. In order to measure this - and to verify the correctness of Lea's random generator -, the Lea.fromVals(...) method can be used as a frequency counter. As an example, let us generate a sample of 42,000 random values and let us count the frequency of each value.

```
>>> sample1 = die.random(42000)
>>> Lea.fromVals(*sample1)
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
>>> sample2 = dieU.random(42000)
>>> Lea.fromVals(*sample2)
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
>>> coinU = Lea.fromValFreqs(("HEAD",65),("TAIL",35))
>>> print (coinU.asPct())
HEAD :  65.0 %
TAIL :  35.0 %
>>> sample3 = coinU.random(100000)
>>> print (Lea.fromVals(*sample3).asPct())
HEAD :  64.9 %
TAIL :  35.1 %
```

You can verify that the frequency of HEAD is close to 65 %, as the defined probability.

### Draws without replacement ###

Certain stochastic processes require that a given value do not appear twice in a random sample. These processes are named "draws without replacement", which include many lottery games. The random(...) method is not suited for such cases; Lea provides a dedicated method: randomDraw(_n_).

As an example, imagine a fair lottery game with 45 balls, from which 6 balls shall be drawn.

```
>>> lottery = Lea.fromVals(*range(1,46))
>>> lottery.randomDraw(6)
(13, 24, 25, 10, 1, 26)
```

The randomDraw(_n_) method guarantees that all drawn values are different. As a special case, if the argument of this method is the number of value, we get a random permutation of all the values.

```
>>> lottery.randomDraw(45)
(44, 42, 36, 4, 10, 2, 14, 19, 22, 39, 12, 41, 18, 7, 31, 5, 38, 45, 21, 8, 13, 1, 3, 23, 32, 35, 20, 40, 11, 29, 27, 15, 30, 37, 16, 28, 43, 17, 6, 34, 25, 24, 9, 33, 26)
```

Should the argument exceeds the number of values, an exception is raised.

```
>>> lottery.randomDraw(46)
...
lea.Error: impossible to build a probability distribution with no value
```

## Calculus with probability distributions ##

### Context ###

The examples seen so far work with explicitly defined probability distributions. In many situations however, a stochastic phenomenon results from the combination of other stochastic phenomenon's, which are easier to model and which have known probability distributions. Examples are:

  * adding the results of _N_ thrown dice,
  * counting the number of successes of _N_ trials,
  * evaluating the truth of a given condition made on stochastic events,
  * evaluating joint occurrences of _N_ stochastic events,
  * applying a given function on a random variable,
  * calculating a given function with random variables as arguments.

The traditional approach requires detailed calculations, made manually or by customised programs; this is tedious and error-prone. Fortunately, Lea has several features to make these derivations of probability distribution easy and generic.

The following sections shall present the most basic techniques, assuming that

> _**all random events are independent**_.

This is an important assumption to take in mind. It allows for using simple methods to derive probabilities; in particular, it is usually well suited to model gambling situations or stateless systems.

For more complex situations, the random events at hand cannot be assumed independent. The occurrence of one random event may influence the occurrence of another random event. In general terms, the knowledge of some information can impact the probability distribution of future (or unknown) events. Lea provides some tools for that situations. These are presented in an "advanced tutorial", which covers topics like joint probability, conditional probability and Bayesian inference.

### Arithmetic operations ###

We shall see here how Lea instances can be put in arithmetic expressions to produce new Lea instances. The most basic cases involve expressions mixing one Lea instance with constant values. In such cases, the independence assumption is obviously true since a constant is something certain.

Here are some examples:

  * Subtracting 3 from the result of a die:
```
>>> die1 = Lea.fromVals(1,2,3,4,5,6)
>>> die1 - 3
-2 : 1/6
-1 : 1/6
 0 : 1/6
 1 : 1/6
 2 : 1/6
 3 : 1/6
```

  * Doubling the result of a die
```
>>> 2 * die1
 2 : 1/6
 4 : 1/6
 6 : 1/6
 8 : 1/6
10 : 1/6
12 : 1/6
```

  * Dividing the result of a die by 3, using floating-point arithmetic
```
>>> die1 / 3
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
>>> die1 // 3
0 : 2/6
1 : 3/6
2 : 1/6
```

  * Taking the residue of a die result modulo 2 (i.e. the parity bit)
```
>>> die1 % 2
0 : 1/2
1 : 1/2
```

  * Adding the result of a die with itself
```
>>> die1 + die1
 2 : 1/6
 4 : 1/6
 6 : 1/6
 8 : 1/6
10 : 1/6
12 : 1/6
```

> Note that this is exactly equivalent to the expression 2 `*` _die1_.

Now, to get the distribution resulting from the sum of two dice, you have to create a new die instance, by issuing a new Lea.fromVals(...) statement or, simpler, by using the clone() method:

```
>>> die2 = die1.clone()
```

From now, _die2_ represents a second die having the same probability distribution as _die1_ but its throw is assumed to be an event independent of the throw of the first die. Then, the following statement produces the distribution of the sum of the two dice.

```
>>> die1 + die2
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

Actually, as you could expect it, the expression has been used to browse all the combination pairs from two dice values (an operation known as the _cartesian product_); these pairs have been reduced by addition and the occurrences of each sum value has been counted.

Other similar expressions may be used to calculate the distribution of the difference of two dice, their product, etc. A more sophisticated example below calculates the square of the difference of the two thrown dice.

```
>>> (die1-die2)**2
 0 : 3/18
 1 : 5/18
 4 : 4/18
 9 : 3/18
16 : 2/18
25 : 1/18
```

Note that the same Lea instance may appear at several places in the same expression. The values are processed consistently, as shown in the following:

```
>>> die1 + 2*die2 - die1 - (die2 + die2)
0 : 1
```

### Repeating operations ###

In some situations, you will want to repeat an operation a given number of times, like throwing _N_ dice, and adding the results.

For instance, what is the probability of getting a total of 13 when throwing 4 dice?

From the techniques seen so far, you could write:

```
>>> (die1 + die1.clone() + die1.clone() + die1.clone()).p(13)
'35/324'
```

This is of course tedious to type. Here are alternatives (shown essentially to demonstrate that Lea integrates smoothly with Python's building blocks !):

```
>>> diexN = 0
>>> for i in range(4):
...     diexN += die1.clone()
... 
>>> diexN.p(13)
'35/324'
```

or, better,

```
>>> diexN = sum(die1.clone() for i in range(4))
>>> diexN.p(13)
'35/324'
```

Now, to alleviate the job, Lea provides a dedicated method called "times":

```
>>> (die1.times(4)).p(13)
'35/324'
```

This emulates the distribution of the sum of 4 thrown dice; then it returns the same result as the previous expression. Note that this is not only shorter to write but also _faster to execute_ because the times(_n_) method uses a dichotomic algorithm, with far less calculations than the former naive expression. To illustrate that point, you may notice that Lea has no particular problem to calculate the distribution resulting of 50 thrown dice:

```
>>> die1.times(50)
 50 :                                      1/808281277464764060643139600456536293376
 51 :                                     50/808281277464764060643139600456536293376
 52 :                                   1275/808281277464764060643139600456536293376
 53 :                                  22100/808281277464764060643139600456536293376
 54 :                                 292825/808281277464764060643139600456536293376
...
```

By default, times(_n_) method performs additions. Any other two-ary operator or two arguments function can be specified as a second argument. For instance, here is how you can get the probability that the product of 4 thrown dice is 5:

```
>>> from operator import mul
>>> (die1.times(4,mul)).p(5)
'1/324'
```

_Note : This probability of 1/324 can be checked by noticing that 5 is a prime number : this means that it shall be reached only by one of 4 permutations of the form 1x1x1x5; hence the probability is 4/6`*``*`4 = 1/324._

#### Binomial distributions ####

The times(_n_) method that has been introduced is very useful to build [binomial distributions](http://en.wikipedia.org/wiki/Binomial_distribution).

Suppose that a given experiment has a success probability _pn_ /_pd_, independent of other performed experiments. Then, the probability distribution of the number of successes after _n_ experiments is given by

> Lea.fromValFreqs((0,_pd_-_pn_),(1,_pn_)).times(_n_)

For example (see Wikipedia link above), suppose a biased coin comes up heads with probability 0.3 when tossed. What is the probability of achieving 0, 1,..., 6 heads after six tosses?

```
>>> bc = Lea.fromValFreqs((0,7),(1,3))
>>> bc.times(6)
0 : 117649/1000000
1 : 302526/1000000
2 : 324135/1000000
3 : 185220/1000000
4 :  59535/1000000
5 :  10206/1000000
6 :    729/1000000
>>> print (bc.times(6).asPct())
0 :  11.8 %
1 :  30.3 %
2 :  32.4 %
3 :  18.5 %
4 :   6.0 %
5 :   1.0 %
6 :   0.1 %
```

### Comparison operations ###

Usual comparison operators may be used on any Lea instances, resulting in Boolean distributions giving the probabilities of each truth value.

```
>>> die1 == 4
False : 5/6
 True : 1/6
>>> die1 <= 4
False : 1/3
 True : 2/3
>>> die1 > 4
False : 2/3
 True : 1/3
```

If you want to get the probability value that the expression is true as a floating-point number, you may use pf(True) method.

```
>>> (die1 <= 4).pf(True)
0.6666666666666666
```

Comparison operators may be used in expressions, involving two independent dice, as shown in the following examples.

```
>>> die1 < die2
False : 7/12
 True : 5/12
>>> die1 <= die2
False : 5/12
 True : 7/12
>>> die1 == die2
False : 5/6
 True : 1/6
```

The first example (die1 < die2) shows that, after throwing two dice, there are 5/12 chances that one _given_ die has a value strictly less than the other's. This assumes that you are able to distinguish between the two dice, for example using colors (e.g. green die + blue die). If you make no difference between dice, then the probability that one of them is strictly less than the other is 5/6, as shown in the last example (die1 == die2 is false).

As more advanced examples, the following expressions involve multiple occurrences of _die1_ and _die2_.

```
>>> (die1-die2)**2 == die1**2 - die2**2
False : 5/6
 True : 1/6
>>> (die1-die2)**2 == die1**2 + die2**2
False : 1
>>> (die1-die2)**2 == die1**2 + die2**2 - 2*die1*die2 
True : 1
```

Let us remark that the last expression is evaluated as certainly true, as we can expect from the well-known algebra rule.

For evaluating the probability of a range of values, you can put two inequalities connected with a logical _and_ (see next section).

For testing the equality of distribution against a set of given values, the isAnyOf(...) shall be used. As an example, here is how to calculate the probability of a "craps":

```
>>> (die1+die2).isAnyOf(2,3,12)
False : 8/9
 True : 1/9
```

The method isNoneOf(...) calculates the inverse distribution, that is the probability of having none of the values in a given set:
```
>>> (die1+die2).isNoneOf(2,3,12)
False : 1/9
 True : 8/9
```

### Logical operations ###

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
>>> ~(die1 == 3)
False : 1/6
 True : 5/6
>>> ~(die1 == 3) | (die1 == 3)
True : 1
>>> (die1 == 4) & (die2 == 2)
False : 35/36
 True :  1/36
>>> (die1 <= 3) & (die2 > 3)
False : 3/4
 True : 1/4
```

WARNINGS:

  * Due to Python precedence rules, it is extremely important to embed expressions connected by logical operators with parentheses. If parentheses are missing, an exception shall be raised.

  * The Python's augmented comparison between Lea instances (e.g. _a_ < _b_ < _c_) DOES NOT WORK! Actually, for several technical reasons related to Python design, such expressions do not (and cannot) return any sensible result. These can be replaced by using a logical AND (e.g. (_a_ < _b_) & (_b_ < _c_) ).

_Note: in Lea versions anterior to 1.2, missing parentheses produce wrong results and_not_raised exception._

### Applying functions ###

The map(_f_) method builds a new Lea distribution by applying the given function _f_ on all the values of a given distribution. For instance, this is how to transform the integer values of a die into floating point numbers:

```
>>> die1.map(float)
1.0 : 1/6
2.0 : 1/6
3.0 : 1/6
4.0 : 1/6
5.0 : 1/6
6.0 : 1/6
```

You could of cause use your own (lambda-) functions:

```
>>> def parity(x):
...     return "odd" if x%2==1 else "even"
... 
>>> die1.map(parity)
even : 1/2
 odd : 1/2
```

Since the map method produces a new Lea instance from an existing Lea instance, you could easily chain several functions, in a pipeline style:

```
>>> die1.map(parity).map(len)
3 : 1/2
4 : 1/2
```


# What's next? #

Thank you for reading the present tutorial!

You should be able now to model your own probability distributions and manipulate them to model a large range of stochastic phenomenon's, assuming independence of the random events.

You can find more examples in the [Examples](Examples.md) page of the Wiki.

If you are looking for more advanced techniques, including draws without replacement, joint distributions, conditional probabilities and Bayesian reasoning, then you are invited to jump to [Lea advanced tutorial](WikiLeaTutorialAdv.md).

Please send your comments, critics, suggestions, bug reports,… by E-mail to pie.denis@skynet.be .