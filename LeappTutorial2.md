**Table of Content**



# Introduction #

The present page is the second part of advanced tutorial for the Lea package. It covers conditional probabilities and Bayes inference. It assumes that you have installed the latest version of Lea and that you are familiar with the techniques presented in the [tutorial](LeappTutorial.md) and in the section about conditional probabilities of [advanced tutorial: Part 1](LeappTutorial1.md).

The present tutorial uses the **Leapp** language. If you prefer a more standard yet longer syntax, then you can jump to [the present tutorial translated in Python](LeaPyTutorial2.md).

# References #

In some sections below, we shall use some examples found in the excellent  ["Artificial Intelligence: A Modern Approach" book](http://aima.cs.berkeley.edu/)  of Stuart Russell and Peter Norvig (second edition). The reader is invited to refer to this book (simply named the "AIMA book" in the following) for more details on presented methods and results.

_The examples below are reproduced with the kind permissions of Stuart Russell and Peter Norvig_.


# Reminder on conditional probabilities #

In the first part of advanced tutorial, you have seen how the `!` operator allows you to calculate conditional probabilities from dependant probability distributions. For example, this is how to calculate the sum of two dice if we know already that one of the die result is 1 or 2:

```
lea> dice ! die1 <= 2
2 : 1/12
3 : 2/12
4 : 2/12
5 : 2/12
6 : 2/12
7 : 2/12
8 : 1/12
```

OK, we have here a simple syntax to _calculate_ conditional probabilities. However, there is a whole class of probability problems where conditional probabilities P(A|B) are _given_. The general formula

> P(A | B) . P(B) = P(A and B)

allows you, when applied with care and patience, answering most problems. In the first part of the advanced tutorial, we have seen how to define dependant random variables through joint distributions. Basically, this boils down to define P(A and B and ...) for each conjunction of 'atomic' events A, B, ... . This method allows modelling very accurately the dependences and to answer any query, including conditional probabilities. However, it requires the knowledge of the probability of each atomic combination, which could be difficult to know. Also, it has the drawback that combinatoric explosion entails the need to enter huge buckets of probability values.

As alternative to joint distributions, Lea provides a more effective approach using _conditional probability tables_ (CPT). The idea is to reduce the degrees of freedom of joint distributions by modelling only causal relationships. In the following, we shall see that this approach is general and powerful when dealing with problems where conditional probabilities are given. We will see that the CPT technique is the cornerstone for Bayes inference.


# Introducing CPT with a beetle #

We shall start with the ["beetle" example (Wikipedia)](http://en.wikipedia.org/wiki/Bayes%27_theorem#Frequentist_example) :

"_An entomologist spots what might be a rare subspecies of beetle, due to the pattern on its back. In the rare subspecies, 98% have the pattern, or P(Pattern|Rare) = 98%. In the common subspecies, 5% have the pattern. The rare subspecies accounts for only 0.1% of the population. How likely is the beetle having the pattern to be rare, or what is P(Rare|Pattern)?_"

To solve this, we need to define two boolean probability distributions, namely `rare` and `pattern`. The first one is unconditional and easy to define:

```
lea> rare = ?:(0.1%)
```

For `pattern`, we cannot (without manual calculations) set it so easily. We need to set two conditional probabilities
  * P(Pattern | Rare) = 98%
  * P(Pattern | ~Rare) = 5%

Lea provides a special construct to set these probabilities:

```
lea> pattern = ?!( rare -> ?:(98%),
 ...              ~rare -> ?:( 5%))
```

This syntax, beginning with `?!`, defines a CPT. The idea is that `pattern` probability shall depend on the probability of `rare`. Note that that an alternative construction, although less general, shall be presented later (see section about `if_`).

Let us check that our definition is in line with what we need :

  * What is the probability of having pattern for a rare beetle?
```
lea> pattern ! rare
False :  1/50
 True : 49/50
lea> :%@ pattern ! rare
98.000000 %
```
  * What is the probability of having pattern for a common beetle?
```
lea> pattern ! ~rare
False : 19/20
 True :  1/20
lea> :%@ pattern ! ~rare
5.000000 %
```

OK, `pattern` gives back the data we put in it; this does not bring any new information, it is just a sanity test. Now, let us come back to the initial question:

  * What is the probability to be rare for a beetle having the pattern?
```
lea> :%@ rare ! pattern
1.924210 %
```

This result is really what Bayes inference is about. It can be checked by manual calculations using the Bayes theorem.

Once the CPT is defined, other calculations can be done easily:

  * What is the probability to be rare for a beetle NOT having the pattern?
```
lea> :%@ rare ! ~pattern
0.002107 %
```

  * What is the probability for any beetle to have the pattern?
```
lea> :%@ pattern
5.093000 %
```

  * What is the probability for a beetle to be rare AND to have the pattern?
```
lea> :%@ rare & pattern
0.098000 %
```

It is even possible to build a joint distribution giving all possible conjunctions (AND), by using the cartesian product:
```
lea> :. ?*(rare,pattern)
(False, False) : 0.949050
 (False, True) : 0.049950
 (True, False) : 0.000020
  (True, True) : 0.000980
```

This first example shows you the general process : you set up the input probabilities, conditional or not, in a declarative manner. Then, you type queries is a natural way and Lea apply conditional probability rules behind the scene.


# CPT with non-boolean probability distributions #

The previous example use boolean probability distributions, which is common with conditional probabilities. However, depending on the problem at hand, other types of distribution can be handled. To illustrate this point we shall re-model the previous problem with 2 variables, `kind` and `aspect`, which refer to string probability distributions:

```
lea> kind = ?{'rare': 0.1%, 'common': 99.9%}
lea> aspect = ?!( kind == 'rare'   -> ?{ 'pattern': 98% , 'no pattern':  2% },
 ...              kind == 'common' -> ?{ 'pattern':  5% , 'no pattern': 95% })
```

Now, `aspect` is a new CPT that gives probability distribution of 'pattern' vs 'no pattern', according to the value of `kind`. Now, the question "what is the probability to be rare for a beetle having the pattern?" can be restated in one of the following manners:

```
lea> kind ! aspect == 'pattern'
common : 4995/5093
  rare :   98/5093
lea> :%@ (kind == 'rare') ! aspect == 'pattern'
1.924210 %
lea> :% (kind ! aspect == 'pattern') @'rare'
1.924210 %
```

In the present example, using booleans or string attributes to model the problem is a matter of taste. However, in many situations, models go beyond
binary choices and cannot be represented by boolean distributions. For example, imagine that the entomologist wants split the kind 'common' into 'common\_A' and 'common\_B' as follows:

```
lea> kind2 = ?{'rare': 0.1%, 'common_A': 34.2%, 'common_B': 65.7% }
```

Also, he wants to split the aspect 'pattern' into 'stripped' and 'mottled', with given conditional probabilities:

```
lea> aspect2 = ?!( kind2 == 'rare'     -> ?{ 'stripped': 78% , 'mottled': 20% , 'no pattern':  2% },
 ...               kind2 == 'common_A' -> ?{ 'stripped':  1% , 'mottled':  4% , 'no pattern': 95% },
 ...               kind2 == 'common_B' -> ?{ 'stripped':  3% , 'mottled':  2% , 'no pattern': 95% })
```

Here are some examples of queries.

```
lea> :% aspect2
   mottled :   2.7 %
no pattern :  94.9 %
  stripped :   2.4 %
lea> :% kind2 ! aspect2 == 'stripped'
common_A :  14.3 %
common_B :  82.4 %
    rare :   3.3 %  
lea> :% kind2 ! aspect2 != 'no pattern'
common_A :  33.6 %
common_B :  64.5 %
    rare :   1.9 %
lea> (kind2[:3]=='com') ! aspect2 != 'no pattern'
False :   98/5093
 True : 4995/5093
lea> :%@ (kind2 == 'rare') ! aspect2 != 'no pattern'
1.924210 %
```

Note the consistency of the last result with the first beetle model:

```
lea> :%@ (kind=='rare') ! aspect=='pattern'
1.924210 %
```

This consistency is due to the fact that the entomologist has carefully made the model refinement so that (see above): P(common) = P(common\_A) + P(common\_B) and P(pattern) = P(stripped) + P(mottled).


# Bayesian networks #

The technique to build CPT can be used to define Bayesian networks, which model causality chains between uncertain events. There is no new syntax here but you shall see how multiple CPT can be connected together to define complex networks.

We shall use the well-known "burglary network" of J. Pearl, explained in [AIMA book](http://aima.cs.berkeley.edu/). You can find also good descriptions of this network on the Internet with the following keywords : _burglar_, _Bayes_.

Here is how to model this Bayesian network in Leapp.

```
lea> burglary = ?:(.001)
lea> earthquake = ?:(.002)
lea> alarm = ?! (  burglary &  earthquake -> ?:(.950),
 ...               burglary & ~earthquake -> ?:(.940),
 ...              ~burglary &  earthquake -> ?:(.290),
 ...              ~burglary & ~earthquake -> ?:(.001))
lea> johnCalls = ?! (  alarm -> ?:(.900),
 ...                  ~alarm -> ?:(.050))
lea> maryCalls = ?! (  alarm -> ?:(.700),
 ...                  ~alarm -> ?:(.010))
```

Note that we have not done more than building three CPT, using the syntax explained in the previous sections. Now, we are ready to query the network. Let us first make "forward" derivations (i.e from causes to effects).

  * What is the probability that Mary calls if the alarm is triggered? (this is a given!)
```
lea> :.@ maryCalls ! alarm
0.7
```

  * What is the probability that Mary calls if there is a burglary?
```
lea> :.@ maryCalls ! burglary
0.6586138
```

Now, we can also query the Bayes network in a "backward" manner (i.e. from effects to cause)

  * What is the probability that there is a burglary if alarm is triggered?
```
lea> :.@ burglary ! alarm
0.373551228281836
```

  * What is the probability that the alarm is triggered if Mary calls?
```
lea> :.@ alarm ! maryCalls
0.1500901177497596
```

  * What is the probability that there is a burglary if Mary calls?
```
lea> :.@ burglary ! maryCalls
0.05611745403891493
```

  * What is the probability that there is a burglary if Mary OR John calls (or both)?
```
lea> :.@ burglary ! maryCalls | johnCalls
0.014814211524985795
```

  * What is the probability that there is a burglary if Mary AND John call?
```
lea> :.@ burglary ! maryCalls & johnCalls
0.28417183536439294
```

It is also possible to get unconditional probabilities of events or conjunction of events

  * What is the probability that the alarm is triggered?
```
lea> :.@ alarm
0.002516442
```

  * What is the probability that Mary calls?
```
lea> :.@ maryCalls
0.01173634498
```

  * What is the probability that there is a burglary and Mary calls?
```
lea> :.@ burglary & maryCalls 
0.0006586138
```

  * What is the probability that there is a burglary, no earthquake, the alarm triggers and both John and Mary call?
```
lea> :.@ burglary & ~earthquake & alarm & johnCalls & maryCalls
0.0005910156
```

  * What is the probability that there is neither burglary nor earthquake, the alarm triggers and both John and Mary call?
```
lea> :.@ ~burglary & ~earthquake & alarm & johnCalls & maryCalls
0.00062811126
```

Note that the last result can be checked in the AIMA book.

As an academic exercise, you can very easily "flatten" the network to build a joint table giving the probabilities of each conjunction of events. This boils down to calculate the cartesian product between the 5 variables.

```
lea> ?*(burglary,earthquake,alarm,johnCalls,maryCalls)
(False, False, False, False, False) : 936742700619/1000000000000
 (False, False, False, False, True) :   9462047481/1000000000000
 (False, False, False, True, False) :  49302247401/1000000000000
  (False, False, False, True, True) :    498002499/1000000000000
 (False, False, True, False, False) :     29910060/1000000000000
  (False, False, True, False, True) :     69790140/1000000000000
  (False, False, True, True, False) :    269190540/1000000000000
   (False, False, True, True, True) :    628111260/1000000000000
 (False, True, False, False, False) :   1334174490/1000000000000
  (False, True, False, False, True) :     13476510/1000000000000
  (False, True, False, True, False) :     70219710/1000000000000
   (False, True, False, True, True) :       709290/1000000000000
  (False, True, True, False, False) :     17382600/1000000000000
   (False, True, True, False, True) :     40559400/1000000000000
   (False, True, True, True, False) :    156443400/1000000000000
    (False, True, True, True, True) :    365034600/1000000000000
 (True, False, False, False, False) :     56317140/1000000000000
  (True, False, False, False, True) :       568860/1000000000000
  (True, False, False, True, False) :      2964060/1000000000000
   (True, False, False, True, True) :        29940/1000000000000
  (True, False, True, False, False) :     28143600/1000000000000
   (True, False, True, False, True) :     65668400/1000000000000
   (True, False, True, True, False) :    253292400/1000000000000
    (True, False, True, True, True) :    591015600/1000000000000
  (True, True, False, False, False) :        94050/1000000000000
   (True, True, False, False, True) :          950/1000000000000
   (True, True, False, True, False) :         4950/1000000000000
    (True, True, False, True, True) :           50/1000000000000
   (True, True, True, False, False) :        57000/1000000000000
    (True, True, True, False, True) :       133000/1000000000000
    (True, True, True, True, False) :       513000/1000000000000
     (True, True, True, True, True) :      1197000/1000000000000
```

We see here the interest of using Bayesian networks, defined with only 10 probability values for the causal dependencies, instead of 32 for the joint tables.


# The else clause #

The CPT construction with the `?!` syntax requires to define a set of conditions that are
  1. mutually exclusive
  1. and together inclusive

The 'else' clause is a pseudo-condition marked with a `_` : it is a placeholder for "any case not covered by the other condition(s)". It allows you to comply automatically with the second rule (inclusiveness).

Let us reconsider the initial "beetle" example:

```
lea> pattern = ?!( rare -> ?:(98%),
 ...              ~rare -> ?:( 5%))
```

This can be rewritten using a "else" clause:

```
lea> pattern = ?!( rare -> ?:(98%),
 ...                  _ -> ?:( 5%))
```

For a more useful example, let us re-consider the burglary network. The `alarm` variable has been defined by enumerating the 4 truth conditions on `burglary` and `earthquake`. It is not mandatory to do so, actually. If probability data are lacking, conditions can be merged together. Consider a new model of alarm device for which the two cases "burglary without earthquake" and "earthquake without burglary" cannot be distinguished, these being associated to a .9 probability. Instead of writing this:

```
alarm2 = ?! (  burglary &  earthquake -> ?:(.950),
               burglary & ~earthquake -> ?:(.900),
              ~burglary &  earthquake -> ?:(.900),
              ~burglary & ~earthquake -> ?:(.001))
```

you could use the else clause:

```
alarm2 = ?! (  burglary &  earthquake -> ?:(.950),
              ~burglary & ~earthquake -> ?:(.001),
                                    _ -> ?:(.900))
```

Final note (for the cleverest!): in this specific case, another way to avoid repetition is to use the xor operator:

```
alarm2 = ?! (  burglary &  earthquake -> ?:(.950),
               burglary ^  earthquake -> ?:(.900),
              ~burglary & ~earthquake -> ?:(.001))
```


# Context-specific independence #

Imagine you have to take into account a new possible cause of alarm in the Bayesian network seen above, namely a checkup, which occurs with a probability of 1% : if there's a checkup, then the alarm is supposed to trigger with a probability 99% ; otherwise the alarm depends of occurrence of burglary or earthquake, as before. The fact that the alarm depends on burglary or earthquake only if no checkup is referred as _context-specific independence_.

The naive approach is to enumerate the 8 cases, as follows:

```
checkup = ?:(1/100)
alarmC = ?! (  checkup &  burglary &  earthquake -> ?:(.990),
               checkup &  burglary & ~earthquake -> ?:(.990),
               checkup & ~burglary &  earthquake -> ?:(.990),
               checkup & ~burglary & ~earthquake -> ?:(.990),
              ~checkup &  burglary &  earthquake -> ?:(.950),
              ~checkup &  burglary & ~earthquake -> ?:(.940),
              ~checkup & ~burglary &  earthquake -> ?:(.290),
              ~checkup & ~burglary & ~earthquake -> ?:(.001))
```

The drawback is that the four first clauses are highly redundant (the 99% probability has been written 4 times).

Fortunately, Lea permits several other constructions to factor out things. The following constructions are ALL equivalent :

_**warning**_ : requires Lea 2.1.1, at least!

```
alarmC = ?! (  checkup                           -> ?:(.990),
              ~checkup &  burglary &  earthquake -> ?:(.950),
              ~checkup &  burglary & ~earthquake -> ?:(.940),
              ~checkup & ~burglary &  earthquake -> ?:(.290),
              ~checkup & ~burglary & ~earthquake -> ?:(.001))

alarmC = ?! ( checkup -> ?:(.990),
              _       -> ?! ( burglary &  earthquake -> ?:(.950),
                              burglary & ~earthquake -> ?:(.940),
                             ~burglary &  earthquake -> ?:(.290),
                             ~burglary & ~earthquake -> ?:(.001)))

alarmC = ?! ( checkup -> ?:(.990),
              _       -> ?! ( burglary -> ?!( earthquake -> ?:(.950),
                                              _          -> ?:(.940)),
                              _        -> ?!( earthquake -> ?:(.290),
                                              _          -> ?:(.001))))
```

Note that there is absolutely no new syntax in these expressions ; it is just a matter of embedding CPT blocks one in the other. There is even shorter, if we allow us to reuse the initial alarm model :

```
alarmC = ?! ( checkup -> ?:(.990),
              _       -> alarm   )
```

This last expression is probably the best : it's the shortest and also the closest of the definition of the new model.


# The `if_` method #

For CPT with only two clauses, Lea provides the `if_` convenience method that mimics the if-then-else constructions found in programming languages. For instance, the example found in the beginning

```
lea> pattern = ?!( rare -> ?:(98%),
 ...              ~rare -> ?:( 5%))
```

can be rewritten as follows :

```
lea> pattern = Lea.if_(rare, ?:(98%), ?:(5%))
```

As in classical programming languages, the `if_` calls can be embedded in second or third arguments, in order to express cascaded conditions (see previous section).


# Prior probabilities #

The syntax to build CPT seen so far misses an important class of problems of conditional probability: those mixing a conditional probability and a prior, unconditional, probability.

We shall take here an example of the [AIMA book](http://aima.cs.berkeley.edu/), referred in the introduction. It models a causal a probability dependency between the meningitis and the stiff neck. The assumptions are the following:

  * the probability that a patient has meningitis is 1/50,000 (prior probability)
  * the probability that a patient has stiff neck is 1/20 (prior probability)
  * the meningitis causes stiff neck 50 % of the times (conditional probability)

OK, the probability of meningitis is easily tackled:
```
lea> meningitis = ?:(1/50000) 
```

For stiff neck however, should we try the CPT syntax, we are stuck:
```
lea> stiffneck = ?! (  meningitis -> ?:(1/2),
 ...                  ~meningitis -> ???? WHAT HERE ??? )
```

The problem is that the probability of stiff neck if NO meningitis is unknown, so we cannot fill in the second part of the CPT definition. By careful calculations, we could try to find this value such that the unconditional probability of stiff neck is equal to 1/20. Fortunately, this is not needed: Lea provides a special syntax, the `!!` operator, to define CPT taking into account prior probability. Here is how to define `stiffneck` from the given data:

```
lea> stiffneck = ?:(1/20) !! (meningitis -> ?:(1/2))
```

This says: "Without any information, the probability of stiff neck for a given patient is 1/20; however, if the patient has a meningitis, then this probability becomes 1/2". Let us check these statements:

```
lea> :@ stiffneck
1/20
lea> :@ stiffneck ! meningitis
1/2
```

Note that `stiffneck` object is a true CPT. Behind the scene, Lea has calculated the probability of stiff neck if no meningitis; then, it was able to fill in the missing data. You can get this value:

```
lea> :@ stiffneck ! ~meningitis
4999/99998
```

In the first statement, you could replace the "??? WHAT HERE ???" by `?:(4999/99998)` and get exactly the same CPT. Of course, this is not needed since Lea has made the work for you!

Now, we are able to compare the probability of meningitis, prior and after stiff neck information:

```
lea> :@ meningitis
1/50000
lea> :@ meningitis ! stiffneck
1/5000
```

This result is in line with the Bayes rule, since

> P(meningitis | stiffneck) = P(stiffneck | meningitis) x P(meningitis) / P(stiffneck)
> > = (1/2) . (1/50000) / (1/20)
> > = 1/5000
> > = 0.0002

We can check that also using Lea:

```
lea> :@ stiffneck ! meningitis
1/2
lea> :@ meningitis
1/50000
lea> :@ stiffneck 
1/20
lea> (stiffneck!meningitis)@ * meningitis@ / stiffneck@
1/5000
```

Note that some problems have no solutions. In such cases, Lea raises an exception with an error message giving the possible range for the prior probability. Example:

```
lea> stiffneck0 = ?:(1/200000) !! (meningitis -> ?:(1/2))
Lea Error: prior probability of 'False' is 199999/200000, outside the range [ 1/100000 , 99999/100000 ]
```

Prior probabilities work also with non-boolean distributions. Let us revisit the beetle example modelled with attribute. Imagine that the problem is restated as follows: the prior probability of a beetle with pattern is 5.093 %, in the rare subspecies, 98% have the pattern; the rare subspecies accounts for only 0.1% of the population. Note that, considering the initial problem, we have removed one data (conditional probability of pattern for common beetle) and added one data (prior probability pattern).

Here are the statements to model these data:

```
lea> aspect0 = ?{ 'pattern': 5.093% , 'no pattern':  94.907% }
lea> aspect1 = aspect0 !! ( kind == 'rare'-> ?{ 'pattern': 98% , 'no pattern':  2% })
```

Now, what is probability of pattern in common subspecies?

```
lea> :% aspect1 ! kind == 'common'
no pattern :  95.0 %
   pattern :   5.0 %
```

This is striking: we get back the same probability of 5% as the initial problem! Why is it so? The answer lies in the given value 5.093 % for prior probability of pattern. Actually, this value has been chosen on purpose, based on unconditional probability of pattern, as calculated in the initial problem (i.e. `aspect`, not `aspect1`):

```
lea> aspect
no pattern : 94907/100000
   pattern :  5093/100000
```

This shows the consistency between the two methods to define CPT.


# Markov chains #

Lea allows defining [Markov chains](http://en.wikipedia.org/wiki/Markov_chain).
We shall show here how to model the [stock market Markov chain example](http://en.wikipedia.org/wiki/Markov_chain#Example) given on Wikipedia.

To execute the examples shown in this section, you have to import the `markov` module present in the Lea package :

```
lea> import markov
```

Note: to be able to execute the examples shown in the present section, you need at least [Lea 2.0.0 beta 3](https://pypi.python.org/pypi/lea).

## Definition from transition probability matrix ##

The states defining the Markov chain can be any Python object. We shall use here the most simple option : strings. The three market states of the example shall be named "BULL", "BEAR" and "STAG". Here is how you define these states in Lea, along with the probability transition matrix (note that we omit the prompts, so you can copy-paste easily the multline statement) :

```
market = markov.Chain.fromMatrix(
                  ( 'BULL', 'BEAR', 'STAG'  ),
        ( 'BULL', (   900 ,    75 ,    25  )), 
        ( 'BEAR', (   150 ,   800 ,    50  )),
        ( 'STAG', (   250 ,   250 ,   500  )))
```

Note that the rows of the matrix represent _from_ state and columns represent _to_ states. For instance, the first row indicates the probabilities of transition starting from "BULL" : 90 % to stay "BULL", 7.5 % to become "BEAR" and 2.5 % to become "STAG".

We have here a Markov chain, named `market`, which captures the set of states and all probabilities of transition from state to state. These probabilities can be displayed by using the same methods as shown for simple distributions:

```
lea> :. market
BULL
  -> BEAR : 0.075000
  -> BULL : 0.900000
  -> STAG : 0.025000
BEAR
  -> BEAR : 0.800000
  -> BULL : 0.150000
  -> STAG : 0.050000
STAG
  -> BEAR : 0.250000
  -> BULL : 0.250000
  -> STAG : 0.500000
```

Take care however that `market` is a `markov.Chain` instance, NOT a Lea instance. Apart displaying itself, the operations you can do on `markov.Chain` are minimal : getting its inner states or given probability distributions of these states. We shall see this in the following.

## Querying the Markov chain ##

To make queries, we need to be able to define an initial state. Here is how to proceed :

```
lea> (bullState,bearState,stagState) = market.getStates()
```

Each of these 3 variables is Lea instance representing a certain distribution, i.e. having a probability 1. For instance, the `bearState` object represents the state BEAR:

```
lea> bearState
BEAR : 1
```

To get the "next state" distribution (i.e. the state at _next week_, in the example), use the `nextState()` method :

```
lea> bearState.nextState()
BEAR : 16/20
BULL :  3/20
STAG :  1/20
```

The object returned by `nextState(…)` is a Lea instance, on which you can apply usual techniques seen in the tutorials. For instance, using the decimal format display, we can check (again) that the given probability transition matrix has been set up correctly :

```
lea> :. bearState.nextState()
BEAR : 0.800000
BULL : 0.150000
STAG : 0.050000
```

To get the state probability distribution after _n_ steps, instead of chaining _n_ times the `nextState()` method, you can simply pass _n_ as argument :

```
lea> :. bearState.nextState(3)
BEAR : 0.568250
BULL : 0.357500
STAG : 0.074250
```

Note that you can check these results with Wikipedia's.

In the example above, we started from a certain state, which was "BEAR". To define an initial distribution mixing multiple states, you need to pass the state probability distribution to the `.getState` method :

```
lea> fpState = market.getState(?{'BULL': 10, 'BEAR': 5, 'STAG': 1})
BEAR :  5/16
BULL : 10/16
STAG :  1/16
```

In the present case, this distribution is a bit special : it is actually the "fixed point" ; you can verify that it does not change on next step :

```
lea> fpState.nextState()
BEAR :  5/16
BULL : 10/16
STAG :  1/16
```

## Definition from a sequence of states ##

There exists another way to define a Markov chain: the empirical way. Instead of setting a probability matrix, which could be hard to define, it is possible to define a Markov chain by providing a sequence of states supposed to be long enough to be representative of all transitions. Typically, this could be a a sample sequence of observed states for a given system.

This sort of definition is done through the `markov.Chain.fromSeq` method. Here is an example, using a very short sequence:

```
lea> market2 = markov.Chain.fromSeq(('BEAR','BEAR','BULL','BULL','BULL','BEAR','STAG','BULL','STAG','BEAR'))
lea> market2
BEAR
  -> BEAR : 1/3
  -> BULL : 1/3
  -> STAG : 1/3
BULL
  -> BEAR : 1/4
  -> BULL : 2/4
  -> STAG : 1/4
STAG
  -> BEAR : 1/2
  -> BULL : 1/2
```

You can verify that the transition probabilities are exactly the frequencies of transition in the given sequence. In particular, there is no transition from "STAG" to "STAG".


## Generating random sequence of states ##

Starting from a given state of a given Markov chain, it is possible to generate a random sequence of states, obeying the transition probabilities of that Markov chain. This is done by calling the `randomSeq(n)` method where `n` is the size of the sequence. Example :

```
lea> bearState.randomSeq(40)
('BEAR', 'BEAR', 'BEAR', 'BULL', 'BULL', 'BULL', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'STAG', 'BULL', 'BULL', 'BULL', 'BULL', 'BULL', 'BULL', 'BULL', 'BULL', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BEAR', 'BULL', 'BULL', 'BULL', 'BULL')
```

Now, to check the likelihood of the generated sequence, we can use the method `markov.Chain.fromSeq` seen above, which can be seen also as a transition frequency calculator. Let's use a random sequence of 10,000 states this time:

```
lea> stateSeqSample = bearState.randomSeq(10000)
lea> :. markov.Chain.fromSeq(stateSeqSample)
BEAR
  -> BEAR : 0.797604
  -> BULL : 0.148640
  -> STAG : 0.053756
BULL
  -> BEAR : 0.073104
  -> BULL : 0.898136
  -> STAG : 0.028760
STAG
  -> BEAR : 0.247453
  -> BULL : 0.254731
  -> STAG : 0.497817
```

We see that the frequencies of transition are close to the probabilities defined for the market Markov chain, as expected.


# What's next? #

Thank you for reading the present advanced tutorial!

We hope that you enjoyed it and that you are now willing to experiment these techniques to solve your own probability problems.

You can find more examples in the [Examples](LeappExamples.md) page of the Wiki.

Please send your comments, critics, suggestions, bug reports,… by E-mail to pie.denis@skynet.be .