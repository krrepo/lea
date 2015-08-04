**Table of Content**



# Introduction #

The present page is the advanced tutorial for the Lea package, using the Leapp language. It assumes that you have installed the latest version of Lea 2 and that you are familiar with the techniques presented in the [Lea tutorial](LeappTutorial.md).

<a href='Hidden comment: 
The goal of Leapp is to have a concise yet expressive syntax to play with probabilities. If you are not willing to learn new syntax or if you want to use Lea as an usual Python module to be imported in your application, then leave Leapp aside and jump to [LeaPyTutorial1 Lea advanced tutorial (Python flavor)].
'></a>

The present tutorial uses the **Leapp** language. If you prefer a more standard yet longer syntax, then you can jump to [the present tutorial translated in Python](LeaPyTutorial1.md).

# References #

In some sections below, we shall use some examples found in the excellent ["Artificial Intelligence: A Modern Approach" book](http://aima.cs.berkeley.edu/) of Stuart Russell and Peter Norvig (second edition). The reader is invited to refer to this book (simply named the "AIMA book" in the following) for more details on presented methods and results.

_The examples below are reproduced with the kind permissions of Stuart Russell and Peter Norvig_.


# A small trick : the Leapp DEBUG mode #

The current implementation of Leapp is quite poor to make good diagnosis of possible syntax errors. One of the reason is that the Leapp parser does not try to understand Python fragments interleaved with Leapp symbols. If you are stuck at understanding why your Leapp expression is erroneous, you could switch the Leapp console into DEBUG mode by issuing the following statement :

```
lea> _leapp.debug = True
```

This has two effects on next executed statements or expressions:
  1. the Python translation of Leapp statement / expression is displayed after a `py#` prefix  (assuming no Leapp syntax error),
  1. when an exception is raised, the full stack trace is displayed.

Example:
```
lea> ?('Head','Tail')
 py# Lea.fromVals(*('Head','Tail'))
Head : 1/2
Tail : 1/2
```

This trick may be useful also for the curious ones, who want to understand how Leapp works.


# Cartesian product #

Given a set of Lea probability distributions, the `?*(…)` syntax allows you to build a new distribution that represents the cartesian product of the values of the given distributions.

```
lea> die1 = ?(1,2,3,4,5,6)
lea> die2 = ?die1
lea> ?*(die1,die2)
(1, 1) : 1/36
(1, 2) : 1/36
(1, 3) : 1/36
(1, 4) : 1/36
(1, 5) : 1/36
(1, 6) : 1/36
(2, 1) : 1/36
(2, 2) : 1/36
(2, 3) : 1/36
…
(6, 3) : 1/36
(6, 4) : 1/36
(6, 5) : 1/36
(6, 6) : 1/36
```

This syntax has unlimited multiple arguments. Note that, as in any Lea expression, if the same variable occurs several times, its values are bound consistently.

```
lea> ?*(die1,die1)
(1, 1) : 1/6
(2, 2) : 1/6
(3, 3) : 1/6
(4, 4) : 1/6
(5, 5) : 1/6
(6, 6) : 1/6
lea> ?*(die1,7-die1)
(1, 6) : 1/6
(2, 5) : 1/6
(3, 4) : 1/6
(4, 3) : 1/6
(5, 2) : 1/6
(6, 1) : 1/6
```

The cartesian product method may seem uninteresting for most of real use cases but, actually, it is used behind the scene for most of Lea's calculations. Also, it could be helpful, for a user point of view, in order to understand or verify how Lea calculates a probability distribution. by adding atomic probabilities present in the cartesian product distribution. For instance, here is how to get the 36 combinations of 2 thrown dice, along with their sum:

```
lea> ?*(die1,die2,die1+die2)
 (1, 1, 2) : 1/36
 (1, 2, 3) : 1/36
 (1, 3, 4) : 1/36
 (1, 4, 5) : 1/36
 (1, 5, 6) : 1/36
 (1, 6, 7) : 1/36
 (2, 1, 3) : 1/36
 (2, 2, 4) : 1/36
 (2, 3, 5) : 1/36
…
 (5, 6, 11) : 1/36
 (6, 1, 7) : 1/36
 (6, 2, 8) : 1/36
 (6, 3, 9) : 1/36
(6, 4, 10) : 1/36
(6, 5, 11) : 1/36
(6, 6, 12) : 1/36
```

Also, you will see later in the present tutorial how it can be used as a "bootstrap" for building joint probability distributions.


# Drawing without replacement #

In many situations (lottery, card games,…), the outcome of a random process results from several draws, where the objects already drawn cannot be drawn again.

For example, imagine a bag with 3 coloured balls:

```
lea> colours = ?('Red','Green','Blue')
lea> colours
 Blue : 1/3
Green : 1/3
  Red : 1/3
```

The `draw(n)` method builds a new Lea distribution resulting from drawing _n_ objects without replacement from a given Lea distribution.

```
lea> colours.draw(2)
('Blue', 'Green') : 1/6
  ('Blue', 'Red') : 1/6
('Green', 'Blue') : 1/6
 ('Green', 'Red') : 1/6
  ('Red', 'Blue') : 1/6
 ('Red', 'Green') : 1/6
lea> colours.draw(3)
('Blue', 'Green', 'Red') : 1/6
('Blue', 'Red', 'Green') : 1/6
('Green', 'Blue', 'Red') : 1/6
('Green', 'Red', 'Blue') : 1/6
('Red', 'Blue', 'Green') : 1/6
('Red', 'Green', 'Blue') : 1/6
```

As you can expect it, should you put a number greater than the number of elements, an exception shall be raised.


# Calculating conditional probabilities #

Lea provides several mechanisms to cope with conditional probabilities. We shall cover here the simplest one, that is _calculating_ condition probabilities involving dependent random events. The problem of _setting_ explicit conditional probabilities and calculating new probabilities thereof (Bayes inference) is more involved; it is covered in a [dedicated page](LeappTutorial2.md) on the wiki.

So, we study here the calculation of conditional probabilities, that is revising a prior probability distribution based on a given evidence. The general syntax corresponding to the mathematical formula

> P( A1 ) = P( A0 | B )

is as follows:

> `a1 = a0 ! b`

where `a1` is the revised Lea instance calculated from prior `a0` Lea instance revised by the known, certain, fact expressed in `b`, a boolean Lea instance.

As a first example, imagine that your friend Nancy calls you by phone; she tells you
> "_Hi, you! I just threw two dice. Can you guess the result?_"
She tells you also
> "_Here is a tip : the result is not higher than 6!_".

OK, calm down… Let us first calculate the prior probabilities for the two dice. Nothing new here:

```
lea> die1 = ?(1,2,3,4,5,6)
lea> die2 = ?die1
lea> dice = die1 + die2
```

Now, you shall revise the distribution of `dice` to include the fact given by Nancy that the result is less or equal to 6.

```
lea> revisedDice = dice ! dice <= 6
lea> revisedDice
2 : 1/15
3 : 2/15
4 : 3/15
5 : 4/15
6 : 5/15
```

The new distribution shows you the chances you have to guess the correct result, based on all the information you have. The best choice undoubtedly is to bet on the 6. Note that this calculation does not take into account the strategy used by Nancy to tell you the upper bound. If the actual result is 4, she could have tell you other upper bounds, from 4 to 12. We suppose here that the strategy of Nancy, if any, is unknown; so, the sole evidence condition we have is that the upper bound is 6.

It is important to note that there is not limitation on the prior distributions; in particular, it could be itself a condition. For example, here is how to calculate the probability that the dice result is greater than 4, provided that we know that it is less or equal to 6:

```
lea> (dice >= 4) ! dice <= 6
False : 1/5
 True : 4/5
```
or, equivalently,
```
lea> revisedDice >= 4
False : 1/5
 True : 4/5
```

Note how this differs from the prior probability that the results is in the range [4,6], assuming nothing else is known:

```
lea> (dice >= 4) & (dice <= 6)
False : 2/3
 True : 1/3
```

The technique seen above can be extended to cope with dependant random variables. We can evaluate the probability of dice sum, assuming we got -by any means- some certain information on each individual die.

Examples:

  * assume we know that the first die is 3:
```
lea> dice ! die1 == 3
4 : 1/6
5 : 1/6
6 : 1/6
7 : 1/6
8 : 1/6
9 : 1/6
```

  * assume we know that at least one of the die is 3:
```
lea> dice ! (die1 == 3) | (die2 == 3)
4 : 2/11
5 : 2/11
6 : 1/11
7 : 2/11
8 : 2/11
9 : 2/11
```

WARNING: It is essential to enclose in parentheses the two subexpressions connected by any logical operator.
_(Thanks, Gilles, for revealing the error in the present tutorial!)_

  * assume we know that the first die is strictly inferior to the second die:
```
lea> dice ! die1 < die2
 3 : 1/15
 4 : 1/15
 5 : 2/15
 6 : 2/15
 7 : 3/15
 8 : 2/15
 9 : 2/15
10 : 1/15
11 : 1/15
```

  * assume we know that the first die is even and the second one is odd:
```
lea> dice ! (die1%2 == 0) & (die2%2 == 1)
 3 : 1/9
 5 : 2/9
 7 : 3/9
 9 : 2/9
11 : 1/9
```

> or, equivalently,
```
lea> dice ! (die1%2 == 0) ! die2%2 == 1
 3 : 1/9
 5 : 2/9
 7 : 3/9
 9 : 2/9
11 : 1/9
```

Note how, in the previous examples, the `dice` variable "remembers" that it was built up from `die1` and `die2`. In the vocabulary of probability theory, we could say that the random variable modelled by `dice` is dependent of the random variables modelled by `die1` and `die2` (these last two being independent from each other). Actually, the Lea engine relies on a "lazy evaluation" technique, in order to take into account the latest given information.

The fact that two random variables are dependent is a symmetric relationship. For example, it is possible to calculate the probability of one die, given information on the sum of dice. Assuming that we know that the sum is 4, without further information on the values of individual die, we get the following probability distribution on the first die:

```
lea> die1 ! dice == 4
1 : 1/3
2 : 1/3
3 : 1/3
```

This results is sensible since any value exceeding 3 for one die shall make the sum of two dice exceed 4.
To get the conditional probabilities for the combinations of the two dice, the cartesian product method shall be used:

```
lea> ?*(die1,die2) ! dice == 4
(1, 3) : 1/3
(2, 2) : 1/3
(3, 1) : 1/3
```

How this magic works? Generally speaking, the evaluation of
```
priorProbabilityDistribution ! evidenceCondition
```

shall
  1. browse all possible values of variables occurring in _evidenceCondition_ or dependent of such variables,
  1. evaluate _priorProbabilityDistribution_ for each combination of values where _evidenceCondition_ has a non-null probability to be true,
  1. count the results of _priorProbabilityDistribution_, by multiplying the probabilities of occurrences of combinations of values in _priorProbabilityDistribution_ and _evidenceCondition_.


# Revised Distributions #

The conditional probability distributions that have just be seen are fine when the new information expressed in the condition is 100% sure. In some situations, this information is uncertain but can be  associated to a precise probability value _P_. The goal is then to calculate a _revised_ probability distribution so that the given condition has a probability _P_ to occur.

The method `withProb(cond,pn,pd)` serves this purpose, where `cond` is a condition expressed as a boolean distribution, `pn` and `pd` are naturals expressing the probability that `cond` is true as the rational number `pn` / `pd`.

As an example, suppose that from a die assumed to be fair initially, we note that it gives eventually the values 5 or 6, with a probability of 1/2. The revised distribution is calculated as follows:

**WARNING**: the method `withProb` is bugged in Lea 2.1.0 and 2.1.1. It is fixed as of Lea 2.1.2.

```
lea> dieU = die1.withProb(die1>=5,1,2)
lea> dieU
1 : 1/8
2 : 1/8
3 : 1/8
4 : 1/8
5 : 2/8
6 : 2/8
```

The probability that has been set can be verified easily by evaluating the given condition on the new distribution:

```
lea> dieU >= 5
False : 1/2
 True : 1/2
```

Note that the present method is a generalisation of the conditional probability method since, for any Lea instance `d` and any condition `cond`,

> `d ! cond`     is equivalent to      `d.withProb(cond,1,1)`

and

> `d ! ~cond`    is equivalent to      `d.withProb(cond,0,1)`


# Min / max functions #

In the ["Applying functions" section](https://code.google.com/p/lea/wiki/LeappTutorial#Applying_functions), we have seen how to build a distribution taking the maximum (or minimum) values from other distributions :

```
lea> ?max(die1,die2)
1 :  1/36
2 :  3/36
3 :  5/36
4 :  7/36
5 :  9/36
6 : 11/36
```

These min/max functions from Python work fine in the present case but have two shortcomings if applied to different number of arguments:

  1. The syntax `?max(*aSequence)` is not accepted. So, there is no way to use tuples, lists, … of Lea instances.
  1. The performance dramatically degrades when adding more arguments (exponential complexity).

Consider for example the following expression that takes the maximum value among 8 dice :
```
lea> ?max(die1,?die1,?die1,?die1,?die1,?die1,?die1,?die1)
1 :       1/1679616
2 :     255/1679616
3 :    6305/1679616
4 :   58975/1679616
5 :  325089/1679616
6 : 1288991/1679616
```

This takes about 10 seconds to calculate (on a "decent" CPU bought in 2013).

These two shortcomings are addressed by new special methods, `Lea.fastMin(…)` and `Lea.fastMax(…)` (requires Lea 2.0.0 beta 4, at least). These methods use an efficient algorithm (linear complexity), which is due to Nicky van Foreest (see [Nicky's "Stochastic Makespans" page](http://nicky.vanforeest.com/scheduling/cpm/stochasticMakespan.html)).

Here is the previous example revamped using this special method :
```
lea> dice = [?die1 for i in range(8)]
lea> Lea.fastMax(*dice)
1 :       1/1679616
2 :     255/1679616
3 :    6305/1679616
4 :   58975/1679616
5 :  325089/1679616
6 : 1288991/1679616
```

The result is the same... but it is produced in the blink of an eye!

However, there ain't no such thing as a free lunch... `Lea.fastMin(…)` and `Lea.fastMax(…)` methods present a small drawback. Unlike most of Lea methods, the distributions returned by these two methods lose any dependency with given arguments; this could be important when using expressions with cartesian product or conditional probabilities.

For instance, the following results are WRONG :

```
lea> Lea.fastMax(die1,die2) ! die1 > 4
1 :  1/36
2 :  3/36
3 :  5/36
4 :  7/36
5 :  9/36
6 : 11/36
lea> (Lea.fastMax(die1,die2) == 6) & (die1 > 4)
False : 97/108
 True : 11/108
```

since each `die1` occurrence is considered as an independent event, e.g. in the first expression, the given information `die1 > 4` is ignored.

To circumvent this, there are special methods `Lea.min(…)` and `Lea.max(…)`, which accept sequences and are "waterproof" :

```
lea> Lea.max(die1,die2) ! die1 > 4
5 : 5/12
6 : 7/12
lea> (Lea.max(die1,die2) == 6) & (die1 > 4)
False : 29/36
 True :  7/36
```

The price to pay is that `Lea.min(…)` and `Lea.max(…)` have dramatic performance degradation when adding more arguments (exponential complexity).

As a conclusion, use `Lea.fastMin(…)` and `Lea.fastMax(…)` for best performance but take care to use them in simple expressions, without other occurrences of inner arguments.


# Indexing and slicing #

When working with Python's strings, tuples or lists, it is common to extract subparts of these objects by using indexing or slicing operations (i.e. the brackets syntax). When a such operation is operated on a Lea instance, it is propagated on the inner values, passing the given (slice) index. The result is a new probability distribution containing the indexed or sliced values.

To give an example, consider the following Lord of the Rings' characters micro-database :

```
lea> character = ?(('Gimli','dwarf'),('Bilbo','hobbit'),('Frodo','hobbit'),('Meriadoc','hobbit'),('Pippin','hobbit'),('Gandalf','wizard'),('Boromir','man'),('Faramir','man'))
lea> character
   ('Bilbo', 'hobbit') : 1/8
    ('Boromir', 'man') : 1/8
    ('Faramir', 'man') : 1/8
   ('Frodo', 'hobbit') : 1/8
 ('Gandalf', 'wizard') : 1/8
    ('Gimli', 'dwarf') : 1/8
('Meriadoc', 'hobbit') : 1/8
  ('Pippin', 'hobbit') : 1/8
```

We can extract the distribution of names and races as follows (remind that first item has index 0 in Python) :

```
lea> name = character[0]
lea> name
   Bilbo : 1/8
 Boromir : 1/8
 Faramir : 1/8
   Frodo : 1/8
 Gandalf : 1/8
   Gimli : 1/8
Meriadoc : 1/8
  Pippin : 1/8
lea> race = character[1]
lea> race
 dwarf : 1/8
hobbit : 4/8
   man : 2/8
wizard : 1/8
```

Other examples (aka "_SQL, my precious!_ ") :

  * What is the distribution of the last letter of names?
```
lea> name[-1]
c : 1/8
f : 1/8
i : 1/8
n : 1/8
o : 2/8
r : 2/8
```

  * Who are the hobbits? (note how `name` and `race` are interdependent variables!)
```
lea> name ! race == 'hobbit'
   Bilbo : 1/4
   Frodo : 1/4
Meriadoc : 1/4
  Pippin : 1/4
```

  * What is the race of those having a name ending with "ir"?
```
lea> race ! name[-2:] == 'ir'
man : 1
```

  * Who is who?
```
lea> name + ' is a ' + race + '.'
   Bilbo is a hobbit. : 1/8
    Boromir is a man. : 1/8
    Faramir is a man. : 1/8
   Frodo is a hobbit. : 1/8
 Gandalf is a wizard. : 1/8
    Gimli is a dwarf. : 1/8
Meriadoc is a hobbit. : 1/8
  Pippin is a hobbit. : 1/8
```

  * Who is the Chosen One?
```
lea> name$ + ' is the Chosen One.'
'Frodo is the Chosen One.'
```


# Object attributes and methods #

The micro-DB example above can be redone using user-defined objects instead of tuples.

```
class C(object):
    
    def __init__(self,name,race):
        self.name = name
        self.race = race
        
    def __str__(self):
        return "%s (%s)"%(self.name,self.race)
        
    def healthPoints(self):
        return 1 + hash(self.name+self.race) % 10

character = ?(C('Gimli','dwarf'),C('Bilbo','hobbit'),C('Frodo','hobbit'),C('Meriadoc','hobbit'),C('Pippin','hobbit'),C('Gandalf','wizard'),C('Boromir','man'),C('Faramir','man'))
```

```
lea> character
   Bilbo (hobbit) : 1/8
  Pippin (hobbit) : 1/8
 Gandalf (wizard) : 1/8
   Frodo (hobbit) : 1/8
    Gimli (dwarf) : 1/8
Meriadoc (hobbit) : 1/8
    Boromir (man) : 1/8
    Faramir (man) : 1/8
```

Then, attributes can be extracted using the dot notation directly on the probability distribution.

```
lea> character.name
   Bilbo : 1/8
 Boromir : 1/8
 Faramir : 1/8
   Frodo : 1/8
 Gandalf : 1/8
   Gimli : 1/8
Meriadoc : 1/8
  Pippin : 1/8
lea> character.race
 dwarf : 1/8
hobbit : 4/8
   man : 2/8
wizard : 1/8
```

Also, method invocation is feasible; this returns a new probability distribution with the returned results.

```
lea> character.healthPoints()
1 : 1/8
3 : 1/8
7 : 2/8
8 : 3/8
9 : 1/8
lea> character.name + ', with hp = ' +  ?str(character.healthPoints())
   Bilbo, with hp = 8 : 1/8
 Boromir, with hp = 3 : 1/8
 Faramir, with hp = 9 : 1/8
   Frodo, with hp = 8 : 1/8
 Gandalf, with hp = 7 : 1/8
   Gimli, with hp = 8 : 1/8
Meriadoc, with hp = 7 : 1/8
  Pippin, with hp = 1 : 1/8
```

Now, the cleverest readers may have noticed an issue with this simple syntax. How Lea can distinguish between his own attributes/methods (like `mean`, `stdev`, `given`, etc) and those to be relayed to the inner objects (like `name`, `race`, etc)? Well, the rule is simple: if the attribute/method name is unknown in Lea API, then it is propagated in the inner objects. Now, it may cause worries in the case of homonyms, i.e. the inner objects have, by mischance, a name defined in Lea API ; such attribute is not accessible with the simple technique since it is "hidden" by the Lea API. Should such situation occurs, there exists a trick using the `getattr` function. Assume for example, that the class C above has an attribute called `mean`. Here is how to get the distribution of this attribute.

```
lea> ?getattr(character,'mean')
```

# Aggregating sequences #

TO BE WRITTEN (`Lea.reduce` method)


# Joint probabilities #

So far, we have assumed a independency of random variables modelled by Lea instances. For modelling dependant random variables, Lea provides several techniques. The most basic technique consist in merging the dependent variables into one distribution that defines the joint probability of each combination of values of these variables.

For instance, if you want to model 3 dependent boolean variables, you shall define the 2 x 2 x 2 = 8 probabilities of each True/False combinations. The values are given by 3-tuples, for instance `(True,False,True)`. The method `asJoint(…)` allows to define a name for each of joint variables.

In the following, we shall use the example of 3 boolean variables Toothache / Cavity / Catch, given in the [AIMA book](http://aima.cs.berkeley.edu/) referred in the introduction (section 13.4 in the second edition).

The probability of occurrence of each of the 8 truth combinations of Toothache / Cavity / Catch is given by a joint distribution table, which can be modelled as a Lea instance having 3-tuples with respective truth values of these variables; then, the method asJoint(…) allows you to define the 3 respective variable names:

```
lea> (T,F) = (True,False)
lea> tdp = ?{ (T,T,T): .108, (T,F,T): .012, (F,T,T): .072, (F,F,T): .008,
 ...          (T,T,F): .016, (T,F,F): .064, (F,T,F): .144, (F,F,F): .576 }
lea> td = tdp.asJoint('toothache','catch','cavity')
lea> :. td
<toothache=False, catch=False, cavity=False> : 0.576000
<toothache=False, catch=False, cavity= True> : 0.008000
<toothache=False, catch= True, cavity=False> : 0.144000
<toothache=False, catch= True, cavity= True> : 0.072000
<toothache= True, catch=False, cavity=False> : 0.064000
<toothache= True, catch=False, cavity= True> : 0.012000
<toothache= True, catch= True, cavity=False> : 0.016000
<toothache= True, catch= True, cavity= True> : 0.108000
```

Then, the joint distribution may be queried to get probabilities of given variables or logical combination of variables.

```
lea> td.cavity
False : 4/5
 True : 1/5
lea> :.@ td.cavity
0.2
lea> :.@ td.cavity | td.toothache
0.28
lea> :.@ td.cavity & ~td.toothache
0.08
```

The resulting distributions are calculated by summing the atomic probabilities given in the joint distribution. In the literature, this kind of calculation is referred to as a _marginalisation_.

You can see in the example that the variables defined in the joint distribution are dependant, as P(catch & cavity) is different from P(catch) x P(cavity):

```
lea> :.@ td.catch
0.34
lea> :.@ td.cavity
0.2
lea> :.@ td.catch & td.cavity
0.18
```

Since variables are dependent, the calculation of conditional probabilities provides useful results like
  * the probability of cavity for a patient having toothache,
  * the probability of cavity for a patient having toothache and a probe being caught.

```
td.cavity ! td.toothache
False : 2/5
 True : 3/5
lea> :.@ td.cavity ! td.toothache
0.6
lea> :.@ td.cavity ! td.toothache & td.catch
0.8709677419354839
```

The reader can verify the correctness of these values in the [AIMA book](http://aima.cs.berkeley.edu/).

The conditional probability method can be used also to produce "reduced" joint distributions from the initial joint distribution:

```
lea> td ! td.cavity
<toothache=False, catch=False, cavity= True> :  2/50
<toothache=False, catch= True, cavity= True> : 18/50
<toothache= True, catch=False, cavity= True> :  3/50
<toothache= True, catch= True, cavity= True> : 27/50
lea> td ! td.cavity & td.toothache
<toothache= True, catch=False, cavity= True> : 1/10
<toothache= True, catch= True, cavity= True> : 9/10
```

Note that, beyond boolean random variables, joint distributions can also be used for numerical values or other Python objects, for which you can evaluate expressions containing usual comparison operators.

The technique of joint distribution is very flexible, allowing defining any possible dependencies between variables. However, it requires defining many probability values and it scales not well as the number of variables grows. In [Lea advanced tutorial: Part 2](LeappTutorial2.md), we shall see an alternative available in Lea, namely the Bayesian networks.


# Information Theory #

Lea allows calculating some of the indicators defined by the information theory. These could provide new insights on your probability distributions.

## Information ##

The most basic concept is the _information_ (in bits) carried by a given event. As a basic example, consider two coins, a fair one and an unfair one :

```
lea> flip = ?('head','tail')
lea> flipU = ?{'head': 1/4, 'tail': 3/4}
```

Here is how to get the information carried by each possible result, after flipping the **fair** coin :
```
lea> flip.informationOf('head')
1.0
lea> flip.informationOf('tail')
1.0
```

Each result carries an information of 1 bit, as expected by the theory. Now, here are the information values for the **unfair** coin :

```
lea> flipU.informationOf('head')
2.0
lea> flipU.informationOf('tail')
0.4150374992788437
```

We see that the information carried by a 'head' result is higher (2 bits) than the the information carried by 'tail'. This is sensible since a 'head' is less likely to occur (prob. 1/4) than a 'tail' (prob. 3/4), hence it carries more information.

For boolean probability distributions, note that `.informationOf(True)` can be replaced by `.information`, which is a convenient shortcut. Example :

```
lea> rain = ?:(1/8)
lea> rain.information
3.0
lea> (flipU=='head').information
2.0
```

## Entropy ##

If you take the average information carried by all events modelled by a given probability distribution, using probabilities of these events as weights, you get the _entropy_ of this distribution. The entropy is one of the most important concept of information theory. It captures the "degree of randomness" of a given probability distribution.

Here is how to get the entropy of each defined coin :

```
lea> flip.entropy
1.0
lea> flipU.entropy
0.8112781244591328
```

The entropy of the fair coin is 1 bit, which is not surprising since we have seen that each possible outcome brings 1 bit of information. The entropy of the unfair coin is about 0.81 bits; this is less than 1 bit, which is consistent with the fact that the unfair coin is less random than the fair one. Note that the theory states that 1 bit is the maximum entropy for a distribution with two values ; it happens when the two values are equally likely.

## Entropy and conditional probabilities ##

The information and entropy can be calculated on any Lea instance, whether defined explicitly as above or resulting of calculations, as explained throughout the tutorials. In particular, these indicators can be calculated on conditional probability distributions.

Let us take an example. There are 64 balls in an urn. 62 of these balls are blue and marked with a "x"; 2 of these balls are red : one is marked with an "x", the other with "y". Here is a short model of the distribution of these balls:

```
lea> ball = ?{'Bx': 62, 'Rx': 1, 'Ry': 1}
lea> ball
Bx : 62/64
Rx :  1/64
Ry :  1/64  
```

Note that it's easy to extract the color code or the mark, using the expression `ball[0]` or `ball[1]` respectively. Let us create two new variables to ease the work :

```
lea> color = ball[0]
lea> color
B : 31/32
R :  1/32
lea> mark = ball[1]
lea> mark
x : 63/64
y :  1/64
```

Let us calculate the the entropy of `ball` :

```
lea> ball.entropy
0.23187232431271465
```

It is quite low since it's almost certain that a drawn ball is Blue, with a "x" mark.

Suppose now that someone draws a ball and tell us some information on its color or mark. Let us calculate the new entropy in the the following cases :

  * What is the entropy if the ball's mark is "x"?
```
lea> (ball ! mark=='x').entropy
0.11759466565886476
```
  * What is the entropy if the ball's mark is "y"?
```
lea> (ball ! mark=='y').entropy
0.0
```
  * What is the entropy if the ball's color is blue?
```
lea> (ball ! color=='B').entropy
0.0
```
  * What is the entropy if the ball's color is red?
```
lea> (ball ! color=='R').entropy
1.0
```

We see in the three first cases that the given information lower the entropy, considering the initial value of 0.23 bits. In two cases, the entropy becomes null : this happens because the new information dissolve any uncertainty on the mark and color. In the last case however, the information that the ball is red increases the entropy to 1 bit. This value is consistent with the uncertainty remaining on the mark of the ball, which is equally likely "x" or "y'. It may sound odd that new information makes the entropy increase. Actually, there is nothing wrong here; it is simply unusual. The initial unbalanced distribution is such that the reception of unlikely information ("the ball is red") can reduce the prior beliefs we had.

## Mutual information ##

When dealing with two dependent random variables, it's interesting to know the amount of information they share. This is the purpose of the _mutual information_, aka _transinformation_. The examples below use the `ball`, `color` and `mark` distributions of our last case, which are clearly interdependent.

```
lea> Lea.mutualInformation(color,mark)
0.08486507530476972
lea> Lea.mutualInformation(color,ball)
0.20062232431271465
lea> Lea.mutualInformation(mark,ball)
0.11611507530476972
```

Note that the mutual information is expressed in bits.

As a stupid example, we can verify that the our ball lottery and coin flipping do not share anything :
```
lea> Lea.mutualInformation(ball,flip)
0.0
```

Note that the mutual information is symmetric, so the method call is unaffected by the order of its to arguments.


# What's next? #

Thank you for reading the present advanced tutorial!

We hope that you enjoyed it and that you are now willing to experiment these techniques to solve your own probability problems. You can find more examples on the [Leapp snippets page](LeappSnippets.md) or, using Python syntax, on the [Examples page](LeappExamples.md).

For a tutorial on conditional probabilities and Bayes inference, you are invited to follow [Lea advanced tutorial: Part 2](LeappTutorial2.md).

Please send your comments, critics, suggestions, bug reports,… by E-mail to pie.denis@skynet.be .