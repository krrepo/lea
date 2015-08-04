**!!!! Wiki pages in progress !!!!**

You, as non-Denis phenomenon, are not supposed to read what follows!


---


## Values with explicit probabilities ##

Another way to define Lea distributions is to provide explicit pairs (value,probability), using `Lea.fromValFreqs(…)` method. For instance, the following statement is another way to define the unfair coin shown above.

```
>>> biasedFlip = Lea.fromValFreqs(('Head',1),('Tail',2))
```

This way of doing is especially handy for non-uniform definitions. By using `Lea.fromValFreqsDict(…)` method, a Python dictionary can be passed also:

```
>>> myPythonDict = {'Head': 1, 'Tail': 2}
>>> biasedFlip = Lea.fromValFreqsDict(myPythonDict)
```

Note that the displayed probabilities are always simplified to have the lowest common denominator. We shall see later how to display the probabilities as percentages. Note also that values given with probability 0, if any, are removed automatically:

```
>>> Lea.fromValFreqs(('Head',1),('Tail',99),('SHAZAM!',0))
Head :  1/100
Tail : 99/100
```

Lea provides a couple of convenience methods for a couple of standard probability distributions, viz. boolean, Bernoulli, binomial and Poisson. These are described hereafter.

### Random boolean variables ###

Based on what has been seen above, you can model random boolean variable by defining the probabilities of True and False values, such as the total is 1. For example, you could model a probability of 27 % that it shall rain tomorrow as

```
>>> rain = Lea.fromValFreqs((True,27),(False,73))
>>> rain
False : 73/100
 True : 27/100
```

Note that there is of course a redundancy in such display since the probability of False can be calculated from the probability of True. We shall see later a more handy way to display probabilities of random variables.

Since boolean variables are very common, Lea provides a convenience method, `Lea.boolProb(n,d)` that models a statement that has probability _n_/_d_ to be true. For instance, the example above can be rewritten as

```
>>> rain = Lea.boolProb(27,100)
```

### Bernoulli distribution ###

The method `Lea.bernoulli(n,d)` defines the value 1 with probability _n_/_d_ and 0 with probability 1 - _n_/_d_

```
>>> Lea.bernoulli(3,10)
0 : 7/10
1 : 3/10
```

### Binomial distribution ###

The method `Lea.binomial(x,n,d)` defines the number of successes among a number _x_ of independent experiments, each having probability _n_/_d_ of success.

```
>>> Lea.binom(6,3,10)
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

Because the Poisson distribution has, in theory, a infinite number of possible values, Lea provides an approximation where values having a probability below 1e-20 are dropped. This threshold can be changed by issuing the needed precision as a second argument, e.g. `Lea.poisson(2,1e-30)`.


---


# Context-Specific Independence #

Imagine you have to take into account a new possible cause of alarm in the Bayesian network seen above, namely a checkup, which occurs with a probability of 1% : if there's a checkup, then the alarm is supposed to trigger with a probability 99% ; otherwise the alarm depends of occurrence of burglary or earthquake, as before. The fact that the alarm depends on burglary or earthquake only if no checkup is referred as _context-specific independence_.

The naive approach is to enumerate the 8 cases, as follows:

```
checkup = ?:(1/100)
alarm2 = ?! (  checkup &  burglary &  earthquake -> ?:(.990),
               checkup &  burglary & ~earthquake -> ?:(.990),
               checkup & ~burglary &  earthquake -> ?:(.990),
               checkup & ~burglary & ~earthquake -> ?:(.990),
              ~checkup &  burglary &  earthquake -> ?:(.950),
              ~checkup &  burglary & ~earthquake -> ?:(.940),
              ~checkup & ~burglary &  earthquake -> ?:(.290),
              ~checkup & ~burglary & ~earthquake -> ?:(.001))
```

The drawback is that the four first clauses are highly redundant (the 99% probability has been written 4 times).

Fortunately, Lea permits several other constructions to factor out things. The following constructions are ALL equivalent.

```
alarm2 = ?! (  checkup                           -> ?:(.990),
              ~checkup &  burglary &  earthquake -> ?:(.950),
              ~checkup &  burglary & ~earthquake -> ?:(.940),
              ~checkup & ~burglary &  earthquake -> ?:(.290),
              ~checkup & ~burglary & ~earthquake -> ?:(.001))

alarm2 = ?! ( checkup -> ?:(.990),
              _       -> ?! ( burglary &  earthquake -> ?:(.950),
                              burglary & ~earthquake -> ?:(.940),
                             ~burglary &  earthquake -> ?:(.290),
                             ~burglary & ~earthquake -> ?:(.001)))

alarm2 = ?! ( checkup -> ?:(.990),
              _       -> ?! ( burglary -> ?!( earthquake -> ?:(.950),
                                              _          -> ?:(.940)),
                              _        -> ?!( earthquake -> ?:(.290),
                                              _          -> ?:(.001))))
```

Note that there is absolutely no new syntax in these expressions ; it is just a matter of embedding CPT blocks one in the other. There is even shorter, if we allow us to reuse the initial alarm model :

```
alarm2 = ?! ( checkup -> ?:(.990),
              _       -> alarm   )
```

This last expression is probably the best : it's the shortest and also the closest of the definition of the new model.