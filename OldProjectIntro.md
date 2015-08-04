
---


---


---

**_WARNING: The present page is no longer maintained!_**

**_You are invited to jump to [the new version](http://code.google.com/p/lea/)._**

---


---


---


![http://lea.googlecode.com/hg/images/Lea_logo.png](http://lea.googlecode.com/hg/images/Lea_logo.png)

# Lea - Discrete probability distributions in Python #

**NEW** : version 1.3 runs on Python 3!

**COMING SOON** : an original [probability mini-language](LeaShell.md) (DSL/PPL), to make Lea easier!

_Lea_ is a Python package aiming at working with discrete probability distributions in an intuitive way. It allows you to model a broad range of random phenomenons, like dice throwing, coin tossing, cards hands, gambling, lottery, … with fair or unfair characteristics! More generally, Lea may be  used for any finite set of discrete values having known probability: numbers, boolean variables (true/false), date/times, symbols, ... Each distribution is modeled as a plain object, which can be named, displayed, queried or processed to produce new distribution objects.

As a basic example, the statements below define and display the probability distribution of one fair die.

```
>>> die1 = Lea.fromVals(1,2,3,4,5,6)
>>> die1
1 : 1/6
2 : 1/6
3 : 1/6
4 : 1/6
5 : 1/6
6 : 1/6
```

From a given probability distribution, you are able to get the usual distribution indicators (mean, variance, etc). You may also ask to generate random samples of values with respect to that distribution, which maybe non-uniform (i.e. some value are more likely to occur than others).

Probably the most special feature of Lea is that it allows you to compute new probability distributions from existing ones, by using different transformation means: arithmetic operators, logical operators, functions, attribute retrieving and cartesian product. For example, assuming that _die1_ and _die2_ refer to distributions of the result of two thrown dice, then the expression _die1_ + _die2_ shall build the distribution of the sum of the results of these two dice. This is shown in the statements below.

```
>>> die2 = die1.clone()
>>> dice = die1 + die2
>>> dice
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

Then, comparison operators can be used to derive boolean probability distributions:

```
>>> dice < 7
False : 7/12
 True : 5/12
```

Furthermore, you may derive new Lea distributions from existing ones by using by specifying logical conditions, which shall constrain the set of possible values (conditional probability).

Here are the main Lea’s features, from a user point of view:

  * allows to define finite discrete probability distributions
  * provides standard distribution indicators
  * comprehensive set of operations to derive distributions from existing ones
  * conditional probabilities
  * easy to use
  * usable in Python interactive interpreter or in application files
  * Open-source project, LGPL license

From a technical / programming perspective, Lea has the following characteristics:

  * pure Python module, lightweight - no package dependency
  * requires Python 3.x or Python 2.x, starting from 2.5
  * OO design: one abstract class, Lea, and eight subclasses
  * probabilities stored as integers, hence no accuracy issues due to floating-point arithmetics
  * heavily relies on Python operator overloading and _lazy_ evaluation (computations done as late as possible)
  * uses a generalised convolution algorithm

Lea presents, in its current version, some limitations or liabilities:

  * no formal test suite
  * no predefined standard distributions
  * not scalable, maybe inefficient for some calculation

These limitations may be addressed in future versions.


---


To learn more, please read the [Lea tutorial](WikiLeaTutorial.md).

For installation instructions, see [Installation](Installation.md).


---


Please send your comments, critics, suggestions, bug reports,… in English or French by E-mail to: **pie.denis@skynet.be**. You are more than welcome / _bienvenus_ !

You can also post issues in the present project site.

Project creator, administrator : Pierre Denis (Louvain-la-Neuve, Belgium)