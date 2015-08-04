
---


---


---

**_WARNING: The present page is no longer maintained!_**

**_You are invited to jump to [the new version](LeaPyTutorial1.md)._**

---


---


---




# Introduction #

The present page is the advanced tutorial for the Lea package. It assumes that you have installed the latest version of Lea (1.3.1) and that you are familiar with the techniques presented in the [tutorial](WikiLeaTutorial.md).

We shall here cover several methods that allows, among others, to deal with dependent random variables and conditional probabilities.

# References #

In some sections below, we shall use some examples found in the excellent  ["Artificial Intelligence: A Modern Approach" book](http://aima.cs.berkeley.edu/)  of Stuart Russell and Peter Norvig (second edition). The reader is invited to refer to this book (simply named the "AIMA book" in the following) for more details on presented methods and results.

_The examples below are reproduced with the kind permissions of Stuart Russell and Peter Norvig_.

# Cartesian product #

Given a set of Lea probability distributions, the method cprod(...) allows you to build a new distribution that is represents the Cartesian product of the values of the given distributions.

```
>>> die1 = Lea.fromVals(1,2,3,4,5,6)
>>> die2 = die1.clone()
>>> die1.cprod(die2)
(1, 1) : 1/36
(1, 2) : 1/36
(1, 3) : 1/36
(1, 4) : 1/36
(1, 5) : 1/36
(1, 6) : 1/36
(2, 1) : 1/36
(2, 2) : 1/36
(2, 3) : 1/36
(2, 4) : 1/36
(2, 5) : 1/36
(2, 6) : 1/36
(3, 1) : 1/36
(3, 2) : 1/36
(3, 3) : 1/36
(3, 4) : 1/36
(3, 5) : 1/36
(3, 6) : 1/36
(4, 1) : 1/36
(4, 2) : 1/36
(4, 3) : 1/36
(4, 4) : 1/36
(4, 5) : 1/36
(4, 6) : 1/36
(5, 1) : 1/36
(5, 2) : 1/36
(5, 3) : 1/36
(5, 4) : 1/36
(5, 5) : 1/36
(5, 6) : 1/36
(6, 1) : 1/36
(6, 2) : 1/36
(6, 3) : 1/36
(6, 4) : 1/36
(6, 5) : 1/36
(6, 6) : 1/36
```

This method has unlimited multiple arguments and could be called also in the form Lea.cprod(...), which may be handier in some cases. For example,

```
>>> Lea.cprod(die1,die2)
```

is equivalent to the previous expression.

Note that, as in any Lea expression, if the same variable occurs several times, its values are bound consistently.

```
>>> die1.cprod(die1)
(1, 1) : 1/6
(2, 2) : 1/6
(3, 3) : 1/6
(4, 4) : 1/6
(5, 5) : 1/6
(6, 6) : 1/6
>>> die1.cprod(7-die1)
(1, 6) : 1/6
(2, 5) : 1/6
(3, 4) : 1/6
(4, 3) : 1/6
(5, 2) : 1/6
(6, 1) : 1/6
>>> Lea.cprod(die1,die2,die1+die2)
 (1, 1, 2) : 1/36
 (1, 2, 3) : 1/36
 (1, 3, 4) : 1/36
 (1, 4, 5) : 1/36
 (1, 5, 6) : 1/36
 (1, 6, 7) : 1/36
 (2, 1, 3) : 1/36
 (2, 2, 4) : 1/36
 (2, 3, 5) : 1/36
 (2, 4, 6) : 1/36
 (2, 5, 7) : 1/36
 (2, 6, 8) : 1/36
 (3, 1, 4) : 1/36
 (3, 2, 5) : 1/36
 (3, 3, 6) : 1/36
 (3, 4, 7) : 1/36
 (3, 5, 8) : 1/36
 (3, 6, 9) : 1/36
 (4, 1, 5) : 1/36
 (4, 2, 6) : 1/36
 (4, 3, 7) : 1/36
 (4, 4, 8) : 1/36
 (4, 5, 9) : 1/36
(4, 6, 10) : 1/36
 (5, 1, 6) : 1/36
 (5, 2, 7) : 1/36
 (5, 3, 8) : 1/36
 (5, 4, 9) : 1/36
(5, 5, 10) : 1/36
(5, 6, 11) : 1/36
 (6, 1, 7) : 1/36
 (6, 2, 8) : 1/36
 (6, 3, 9) : 1/36
(6, 4, 10) : 1/36
(6, 5, 11) : 1/36
(6, 6, 12) : 1/36
```

The cartesian product method may seem uninteresting for most of real use cases but it is actually used behind the scene for most of Lea's calculations. Also, it could be helpful, for a user point of view, in order to understand or verify how Lea calculates a probability distribution by adding atomic probabilities present in the cartesian product distribution. Also, you will see later in the present tutorial how it can be used as a "bootstrap" for building joint probability distributions.

# Applying _n_-ary functions #

In the basic tutorial, you have seen how to apply a one-argument function thanks to the "map" method. This can be used on a cartesian product distribution, provided that the passed function knows how to handle a tuple given as argument. For instance, to get the distribution of the minimum value between two thrown dice, we can use the standard "min" function of Python, since it can work with a tuple:

```
>>> Lea.cprod(die1,die2).map(min)
1 : 11/36
2 :  9/36
3 :  7/36
4 :  5/36
5 :  3/36
6 :  1/36
```

This technique fails if you try to pass a two-arguments function. For example, trying to emulate the addition with the following expression shall raise an exception:

```
>>> Lea.cprod(die1,die2).map(lambda x,y:x+y)
...
TypeError: <lambda>() missing 1 required positional argument: 'y'
```

and replacing the lambda expression by operator.add will fail very similarly. What you could do instead is unpacking the tuple:

```
>>> Lea.cprod(die1,die2).map(lambda x:x[0]+x[1])
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

A better approach however is to use the Flea.build method:

```
>>> Flea.build(lambda x,y:x+y,(die1,die2))
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

This technique is easier: the first argument is a _n_-ary function and the second argument is a tuple with _n_ Lea distributions, which are combined with an implicit cartesian product.

As a final, rather contrived example, here is how to calculate the distribution of the length of the hypothenuse of a triangle if the two other sides are sized by dice:

```
>>> from math import hypot
>>> Flea.build(hypot,(die1,die2))
1.4142135623730951 : 1/36
  2.23606797749979 : 2/36
2.8284271247461903 : 1/36
3.1622776601683795 : 2/36
 3.605551275463989 : 2/36
 4.123105625617661 : 2/36
 4.242640687119285 : 1/36
  4.47213595499958 : 2/36
               5.0 : 2/36
5.0990195135927845 : 2/36
 5.385164807134504 : 2/36
 5.656854249492381 : 1/36
 5.830951894845301 : 2/36
 6.082762530298219 : 2/36
 6.324555320336759 : 2/36
6.4031242374328485 : 2/36
 6.708203932499369 : 2/36
7.0710678118654755 : 1/36
 7.211102550927978 : 2/36
 7.810249675906654 : 2/36
  8.48528137423857 : 1/36
```

# Drawing without replacement #

In many situations (lottery, card games,...), the outcome of a random process results from several draws, where the objects already drawn can not be drawn again.

For example, imagine a bag with 3 coloured balls:

```
>>> colours = Lea.fromVals('Red','Green','Blue')
>>> colours
 Blue : 1/3
Green : 1/3
  Red : 1/3
```

The draw(_n_) method builds a new Lea distribution resulting from drawing _n_ objects without replacement from a given Lea distribution.

```
>>> colours.draw(2)
('Blue', 'Green') : 1/6
  ('Blue', 'Red') : 1/6
('Green', 'Blue') : 1/6
 ('Green', 'Red') : 1/6
  ('Red', 'Blue') : 1/6
 ('Red', 'Green') : 1/6
>>> colours.draw(3)
('Blue', 'Green', 'Red') : 1/6
('Blue', 'Red', 'Green') : 1/6
('Green', 'Blue', 'Red') : 1/6
('Green', 'Red', 'Blue') : 1/6
('Red', 'Blue', 'Green') : 1/6
('Red', 'Green', 'Blue') : 1/6
```

# Conditional probabilities #

## Calculating conditional probabilities ##

Lea is able to calculate conditional probabilities, that is revising a prior probability distribution based on a given evidence. For that purpose, a new Lea method, called _given_, is introduced. The general pattern is as follows:

```
newProbabilityDistribution = priorProbabilityDistribution.given(evidenceCondition)
```

where _newProbabilityDistribution_ is a Lea instance calculated from initial _priorProbabilityDistribution_ revised by the known, certain, fact expressed in _evidenceCondition_.

As a first example, imagine that your friend Nancy calls you by phone; she tells you that she just threw two dice and she asks you to guess what is the result; she tells you also that the actual results is not higher than 6.

OK. Let us first calculate the prior probabilities for the two dice. Nothing new here:

```
>>> die1 = Lea.fromVals(1,2,3,4,5,6)
>>> die2 = die1.clone()
>>> dice = die1 + die2
```

Now, you shall revise the distribution of _dice_ to include the fact given by Nancy that the result is less or equal to 6.

```
>>> revisedDice = dice.given(dice <= 6)
>>> revisedDice
2 : 1/15
3 : 2/15
4 : 3/15
5 : 4/15
6 : 5/15
```

The new distribution shows you the chances you have to guess the correct result, based on all the information you have. The best choice undoubtedly is to bet on the 6. Note that this calculation does not take into account the strategy used by Nancy to tell you the upper bound. If the actual result is 4, she could have tell you other upper bounds, from 4 to 12. We suppose here that the strategy of Nancy, if any, is unknown; so, the sole evidence condition we have is that the upper bound is 6.

It is important to note that there is not limitation on the prior distributions; in particular, it could be itself a condition. For example, here is how to calculate the probability that the dice result is greater than 4, provided that we know that it is less or equal to 6:

```
>>> (dice >= 4).given(dice <= 6)
False : 1/5
 True : 4/5
```
or, equivalently,
```
>>> revisedDice >= 4
False : 1/5
 True : 4/5
```

Note how this differs from the prior probability that the results is in the range [4,6], without any other information:

```
>>> (dice >= 4) & (dice <= 6)
False : 2/3
 True : 1/3
```

The technique seen above can be extended to cope with dependant random variables. We can evaluate the probability of dice sum, assuming we got -by any means- some certain information on each individual die.

Examples:

  * assume we know that the first die is 3:
```
>>> dice.given(die1 == 3)
4 : 1/6
5 : 1/6
6 : 1/6
7 : 1/6
8 : 1/6
9 : 1/6
```

  * assume we know that at least one of the die is 3:
```
>>> dice.given((die1 == 3) | (die2 == 3))
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
>>> dice.given(die1 < die2)
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
>>> dice.given((die1%2 == 0) & (die2%2 == 1))
 3 : 1/9
 5 : 2/9
 7 : 3/9
 9 : 2/9
11 : 1/9
```

> or, equivalently,
```
>>> dice.given(die1%2==0).given(die2%2==1) 
 3 : 1/9
 5 : 2/9
 7 : 3/9
 9 : 2/9
11 : 1/9
```

Note how, in the previous examples, the _dice_ variable "remembers" that it was built up from _die1_ and _die2_. In the vocabulary of the probability theory, we could say that the random variable modelled by _dice_ is dependent of the random variables modelled by _die1_ and _die2_ (these last two being independent from each other). Actually, the Lea engine relies on a "lazy evaluation" technique, in order to take into account the latest given information.

The fact that two random variables are dependent is a symmetric relationship. For example, it is possible to calculate the probability of one die, given information on the sum of dice. Assuming that we know that the sum is 4, without further information on the values of individual die, we get the following probability distribution on the first die:

```
>>> die1.given(dice == 4)
1 : 1/3
2 : 1/3
3 : 1/3
```

This results is sensible since any value exceeding 3 for one die shall make the sum of two dice exceed 4.
To get the conditional probabilities for the combinations of the two dice, the cartesian product method shall be used:

```
>>> (die1.cprod(die2)).given(dice == 4)
(1, 3) : 1/3
(2, 2) : 1/3
(3, 1) : 1/3
```

How this magic works? Generally speaking, the evaluation of
```
priorProbabilityDistribution.given(evidenceCondition)
```

shall
  1. browse all possible values of variables occurring in _evidenceCondition_ or dependent of such variables,
  1. evaluate _priorProbabilityDistribution_ for each combination of values where _evidenceCondition_ has a non-null probability to be true,
  1. count the results of _priorProbabilityDistribution_, by multiplying the probabilities of occurrences of combinations of values in _priorProbabilityDistribution_ and _evidenceCondition_.

## Revised Distributions ##

The conditional probability distributions that have just be seen are fine when the new information expressed in the condition is 100% sure. In some situations, this information is uncertain but can be  associated to a precise probability value _P_. The goal is then to calculate a _revised_ probability distribution so that the given condition has a probability _P_ to occur.

The method withProb(_cond_,_pn_,_pd_) serves this purpose, where _cond_ is a condition expressed as a boolean distribution, _pn_ and _pd_ are naturals expressing the probability that _cond_ is true as the rational number _pn_ / _pd_.

As an example, suppose that from a die assumed to be fair initially, we note that it gives eventually the values 5 or 6, with a probability of 1/2. The revised distribution is calculated as follows:

```
>>> dieU = die1.withProb(die1>=5,1,2)
>>> dieU
1 : 1/8
2 : 1/8
3 : 1/8
4 : 1/8
5 : 2/8
6 : 2/8
```

The probability that has been set can be verified easily by evaluating the given condition on the new distribution:

```
>>> dieU >= 5
False : 1/2
 True : 1/2
```


Note that the present method is a generalisation of the conditional probability method since, for any Lea instance _d_ and any condition _cond_,

> _d_.given(_cond_)     is equivalent to      _d_.withProb(_cond_,1,1)

and

> _d_.given(~_cond_)    is equivalent to      _d_.withProb(_cond_,0,1)


# Joint probabilities #

So far, we have assumed a independency of random variables modelled by Lea instances. For modelling dependant random variables, Lea provides several techniques. The most basic technique consist in merging the dependent variables into one distribution that defines the joint probability of each combination of values of these variables.

For instance, if you want to model 3 dependent boolean variables, you shall define the 2 x 2 x 2 = 8 probabilities of each True/False combinations. The values are given by 3-tuples. The method asJoint(...) allows to define a name for each of joint variables.

In the following, we shall use the example of 3 boolean variables Toothache / Cavity / Catch, given in the [AIMA book](http://aima.cs.berkeley.edu/) referred in the introduction (section 13.4 in the second edition).

The probability of occurrence of each of the 8 truth combinations of Toothache / Cavity / Catch is given by a joint distribution table, which can be modelled as a Lea instance having 3-tuples with respective truth values of these variables; then, the method asJoint(...) allows you to define the 3 respective variable names:

```
>>> (T,F) = (True,False)
>>> td = Lea.fromValFreqs( ((T,T,T), 108), ((T,F,T),  12), ((F,T,T),  72), ((F,F,T),   8),
                           ((T,T,F),  16), ((T,F,F),  64), ((F,T,F), 144), ((F,F,F), 576) ).asJoint('toothache','catch','cavity')
>>> td
<toothache=False, catch=False, cavity=False> : 144/250
<toothache=False, catch=False, cavity= True> :   2/250
<toothache=False, catch= True, cavity=False> :  36/250
<toothache=False, catch= True, cavity= True> :  18/250
<toothache= True, catch=False, cavity=False> :  16/250
<toothache= True, catch=False, cavity= True> :   3/250
<toothache= True, catch= True, cavity=False> :   4/250
<toothache= True, catch= True, cavity= True> :  27/250
```

Then, the joint distribution may be queried to get probabilities of given variables or logical combination of variables.

```
>>> td.cavity
False : 4/5
 True : 1/5
>>> td.cavity.pf(True)
0.2
>>> td.cavity | td.toothache
False : 18/25
 True :  7/25
>>> (td.cavity | td.toothache).pf(True)
0.28
>>> (td.cavity & ~td.toothache).pf(True)
0.08
```

The resulting distributions are calculated by summing the atomic probabilities given in the joint distribution. You can see in the example that the variables defined in the joint distribution are dependant, as P(catch & cavity) is different from P(catch) x P(cavity):

```
>>> td.catch.pf(True)
0.34
>>> td.cavity.pf(True)
0.2
>>> (td.catch & td.cavity).pf(True)
0.18
```

Since variables are dependent, the calculation of conditional probabilities provides useful results like
  * the probability of cavity for a patient having toothache,
  * the probability of cavity for a patient having toothache and a probe being caught.

```
td.cavity.given(td.toothache)
False : 2/5
 True : 3/5
>>> td.cavity.given(td.toothache).pf(True)
0.6
>>> td.cavity.given(td.toothache&td.catch).pf(True)
0.8709677419354839
```

The reader can verify the correctness of these values in the [AIMA book](http://aima.cs.berkeley.edu/).

The conditional probability method can be used also to produce "reduced" joint distributions from the initial joint distribution:

<a href='Hidden comment: 
_values with Lea 0.3 - WRONG !!!_
```
>>> td.given(td.cavity)
<toothache=False, catch=False, cavity= True> : 26/150
<toothache=False, catch= True, cavity= True> : 34/150
<toothache= True, catch=False, cavity= True> : 39/150
<toothache= True, catch= True, cavity= True> : 51/150
>>> td.given(td.cavity & td.toothache)
<toothache= True, catch=False, cavity= True> : 13/30
<toothache= True, catch= True, cavity= True> : 17/30
```

_values with Lea 1.0 - CORRECT_
'></a>

```
>>> td.given(td.cavity)
<toothache=False, catch=False, cavity= True> :  2/50
<toothache=False, catch= True, cavity= True> : 18/50
<toothache= True, catch=False, cavity= True> :  3/50
<toothache= True, catch= True, cavity= True> : 27/50
>>> td.given(td.cavity & td.toothache)
<toothache= True, catch=False, cavity= True> : 1/10
<toothache= True, catch= True, cavity= True> : 9/10
```

Note that, beyond boolean random variables, joint distributions can also be used for numerical values or other Python objects, for which you can evaluate expressions containing usual comparison operators.

# Setting conditional probabilities #

We have seen how to define dependant random variables through joint distributions. This method allows to model very accurately the dependences but it requires the knowledge of the probability of each atomic combination, which could be difficult to know. Also, it has the drawback that combinatoric explosion entails the need to enter a lot of probability values.

If we accept to reduce the degree of freedom of the variable dependencies, then a simpler approach consists in setting known conditional probability involving the dependent variables. This approach is made up of two steps:

  1. defining an initial "neutral" joint distribution between the variables, assuming they are independent,
  1. derive, from this distribution, a new joint distribution that obeys a given conditional probability.

We shall take here again an example of the [AIMA book](http://aima.cs.berkeley.edu/), referred in the introduction. It establishes a causal dependency between the disease meningitis and the stiff neck. The assumptions are the following:

  * the probability that a patient has meningitis is 1/50,000 (prior, or unconditional, probability)
  * the probability that a patient has stiff neck is 1/20 (prior, or unconditional, probability)
  * the meningitis causes stiff neck 50 % of the times (conditional probability)

Here are the statements required to model these data.

```
>>> meningitis = Lea.boolProb(1,50000)
>>> stiffneck = Lea.boolProb(1,20)
>>> ms0 = Lea.cprod(meningitis,stiffneck).asJoint('meningitis','stiffneck')
>>> ms1 = ms0.withCondProb(ms0.stiffneck,ms0.meningitis,1,2)
```

The distribution _ms0_ is a "temporary" joint distribution that assumes independence between meningitis and stiff neck : it only takes into account the given prior probabilities and not the conditional probability. It does not bring any useful data, apart from the prior probabilities. For instance, in ms0, knowing the presence of meningitis does not affect the probability of stiff neck and conversely.

```
>>> ms0
<meningitis=False, stiffneck=False> : 949981/1000000
<meningitis=False, stiffneck= True> :  49999/1000000
<meningitis= True, stiffneck=False> :     19/1000000
<meningitis= True, stiffneck= True> :      1/1000000
>>> ms0.stiffneck.given(ms0.meningitis)
False : 19/20
 True :  1/20
>>> ms0.meningitis.given(ms0.stiffneck)
False : 49999/50000
 True :     1/50000
```

Now, the true final result is _ms1_, which is the joint distribution that takes into account all given probabilities, whether prior or conditional.

```
>>> ms1
<meningitis=False, stiffneck=False> : 94999/100000
<meningitis=False, stiffneck= True> :  4999/100000
<meningitis= True, stiffneck=False> :     1/100000
<meningitis= True, stiffneck= True> :     1/100000
>>> ms1.meningitis
False : 49999/50000
 True :     1/50000
>>> ms1.stiffneck
False : 19/20
 True :  1/20
>>> ms1.stiffneck.given(ms1.meningitis)
False : 1/2
 True : 1/2
```

The added value of the _ms1_ distribution is that it can calculate new conditional probability values:

```
>>> ms1.meningitis.given(ms1.stiffneck)
False : 4999/5000
 True :    1/5000
>>> ms1.meningitis.given(ms1.stiffneck).pf(True)
0.0002
>>> ms1.meningitis.given(~ms1.stiffneck)
False : 94999/95000
 True :     1/95000
```

The consistency of Lea calculations can be demonstrated by checking the Bayes rule in the present specific case:

> P(meningitis | stiffneck) = P(stiffneck | meningitis) x P(meningitis) / P(stiffneck) = 0.0002

```
>>> ms1.stiffneck.given(ms1.meningitis).pf(True) * ms1.meningitis.pf(True) / ms1.stiffneck.pf(True)
0.0002
```

As a final comment, note that what have been presented above still relies on an internal joint probability table, which could be prohibitive when number of variables becomes large. Better techniques exist however, based on _bayesian networks_. This is not yet available in the current version of Lea (1.x) but this is planned for forthcoming Lea 2.

# What's next? #

Thank you for reading the present advanced tutorial!

We hope that you enjoyed it and that you are now willing to experiment these techniques to solve your own probability problems.

You can find more examples in the [Examples](Examples.md) page of the Wiki.

Please send your comments, critics, suggestions, bug reports,… by E-mail to pie.denis@skynet.be .