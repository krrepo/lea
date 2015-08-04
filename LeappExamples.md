**Table of Content**



# introduction #

The following page provides several examples using Lea, Leapp flavor.

Unless otherwise specified, all the examples just require Lea module on top of a standard Python installation.

If some Lea statements are unclear, please refer to the [basic tutorial](LeappTutorial.md) or the [advanced tutorial](LeappTutorial1.md). Note that some of the use cases assumes a _good_ knowledge of the Python language!

# prerequisites #

In order to execute the statements given in the present page, you have to install the Lea package. Instructions are provided in [Installation page](Installation.md). Then, to start the Leapp console, execute in a terminal (`%` is the prompt):

```
% python -m leapp
```

The Lea module version and copyright notice shall be displayed; then, the following prompt shall appear:

```
lea> 
```

From here, all the examples provided in the present page can be executed. What follow the `lea>` prompt are the Leapp statements or expressions that _you_ type; then, below, the results (if any) of the execution are displayed.

# Sicherman dice #

I discovered the strange [Sicherman dice](http://en.wikipedia.org/wiki/Sicherman_dice) in the Scientific American's column of Martin Gardner.

Here is how you can verify that they result in the same probability distribution as two normal dice.

```
lea> dieS1 = ?(1,2,2,3,3,4)
lea> dieS2 = ?(1,3,4,5,6,8)
lea> dieS1 + dieS2
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


# nontransitive dice #

[Nontransitive dice](http://en.wikipedia.org/wiki/Non-Transitive_Dice) are an intriguing mathematical curiosity. It is another counterintuitive fact in the domain of probabilities.

In the following example, we consider the 3 special dice A, B and C (see face labels below).

```
lea> dieA = ?(3,3,5,5,7,7)
lea> dieB = ?(2,2,4,4,9,9)
lea> dieC = ?(1,1,6,6,8,8)
```

The calculated probabilities associated to inequalities show that
  * die A beats die B, with probability 5/9
  * die B beats die C, with probability 5/9
  * die C beats die A, with probability 5/9

```
lea> dieA > dieB
False : 4/9
 True : 5/9
lea> dieB > dieC
False : 4/9
 True : 5/9
lea> dieC > dieA
False : 4/9
 True : 5/9
```

This forms a strange cycle where, on the average, each die beats another die while it is itself beaten by the remaining die.

Other examples (Grime's dice), with thorough explanation and video, can be found [here](http://grime.s3-website-eu-west-1.amazonaws.com/).

# time management #

Suppose you have to do 2 tasks T1 and T2, which can last uncertain durations. After careful evaluations, you estimate the following probability distributions for the duration of each task:

  * T1 : 2 days (20%), 3 days (60%), 4 days (20%)
  * T2 : 3 days (20%), 4 days (80%)

To handle time calculation, we shall use the handy datetime built-in Python module. For convenience, we define the constant DAY representing the duration of one day:

```
lea> from datetime import *
lea> DAY = timedelta(1)
lea> T1 = ?{2: 20, 3: 60, 4: 20} * DAY
lea> T2 = ?{3: 20, 4: 80} * DAY
lea> T1
2 days, 0:00:00 : 1/5
3 days, 0:00:00 : 3/5
4 days, 0:00:00 : 1/5
lea> T2
3 days, 0:00:00 : 1/5
4 days, 0:00:00 : 4/5
```

Now assuming that T1 could start on 28th September 2020 and that T1 shall be completed before starting T2, what is the distribution of the end time?

```
lea> startTime = datetime(2020,9,28)
lea> endTime = startTime + T1 + T2
lea> endTime
2020-10-03 00:00:00 :  1/25
2020-10-04 00:00:00 :  7/25
2020-10-05 00:00:00 : 13/25
2020-10-06 00:00:00 :  4/25
```

For an "executive summary" point of view, the mean time of completion is

```
lea> : endTime.mean
2020-10-04 19:12:00
```

_Note (for the curious): How can Lea calculate a mean time, although it is impossible (and meaningless) to add dates together? Actually, the implementation of Lea's `mean` method works by calculating the mean of the differences between each values and the first value; then this mean difference is added to the first value. With numbers, this provides the same results as the simple algorithm, with the advantage of avoiding any risk of overflow; in the case of `datetime` objects, it sums actually `timedelta` objects (i.e. durations), thus avoiding a raised exception._

Here is how to produce figures on the overall duration of the tasks:

```
lea> endTime - startTime
5 days, 0:00:00 :  1/25
6 days, 0:00:00 :  7/25
7 days, 0:00:00 : 13/25
8 days, 0:00:00 :  4/25
lea> : (endTime - startTime).mean
6 days, 19:12:00
```

Assume now that there is no dependency between T1 and T2 and that you have enough resources to start T1 and T2 in parallel. Then, the distribution of the overall duration is no more T1+T2 but it is max(T1,T2), i.e. it is the duration of the longest task. Unfortunately, you can not write such simple expression because the Python `max(...)` function does not know how to handle Lea instances (actually, it produces a wrong result) ; you have to use the `Lea.max(...)` method:

```
lea> endTimeP = startTime + Lea.max(T1,T2)
lea> endTimeP
2020-10-01 00:00:00 :  4/25
2020-10-02 00:00:00 : 21/25
lea> : endTimeP.mean
2020-10-01 20:09:36
```

We see, as expected, that the mean end time here is earliest than in the sequential scenario.

# lottery #

```
lea> lottery = ?(range(1,46))
lea> lottery $
13
lea> lottery2 = lottery ! lottery != 13
lea> lottery2 $
24
lea> lottery3 = lottery2 ! lottery2 != 24
lea> lottery3 $
25
...
```

Same as (far more efficient)...

```
lea> lottery $_(6)
(13, 24, 25, 10, 1, 26)
```

# character frequency counter #

In the following example, we count the number of occurrences of each letter in a famous Latin sentence.
This shows how a frequency analysis can be done.

```
lea> freqLatin = ?('ALEA JACTA EST')
lea> freqLatin
  : 2/14
A : 4/14
C : 1/14
E : 2/14
J : 1/14
L : 1/14
S : 1/14
T : 2/14
```

From this basic frequency table, we can generate a random message that tends to obey the same frequency.

```
lea> ''.join(freqLatin$(20))
'AJ  LTC TAJA ATLEALJ'
```

You can notice how the 'A' occurs more than the other letters.


# the lifetime problem #

Suppose we have a device that has 1% to break every time it is switched on.
Let _u_ represents the probability that it is not broken after one switch on:

```
lea> u = ?:(99/100)
lea> u
False :  1/100
 True : 99/100
```

To determine the probability that the device is OK after two switches on, you may write:

```
lea> u & ?u
False :  199/10000
 True : 9801/10000
```

To determine the probability that it is OK after ten switches on, you may write:

```
lea> from operator import and_
lea> ?[10,and_]u
False :  9561792499119550999/100000000000000000000
 True : 90438207500880449001/100000000000000000000
```

You can verify the result as follows:

```
lea> :.@ ?[10,and_]u
0.9043820750088044
lea> 0.99 ** 10
0.9043820750088044
```

# 2D random walk #

We can use Python complex numbers to model a body (e.g. a drunk rabbit) that moves randomly in a plane : in a complex plane, the North direction is the imaginary 1j value, the East direction is the value 1, etc. Let us assume that the body moves at each time step East, North, West or South, with equal probabilities:

```
lea> b = ?(1,1j,-1,-1j)
lea> b
  1 : 1/4
-1j : 1/4
 1j : 1/4
 -1 : 1/4
```

Then, assuming that the body is initially located at the origin of the complex plane, the probability distribution of the body's position after two time steps is given by

```
lea> b + ?b
      0 : 4/16
(-1+1j) : 2/16
      2 : 1/16
 (1+1j) : 2/16
     2j : 1/16
(-1-1j) : 2/16
    -2j : 1/16
 (1-1j) : 2/16
     -2 : 1/16
```

After four time steps,

```
lea> ?[4]b
      0 : 36/256
      2 : 16/256
      4 :  1/256
     2j : 16/256
 (2+2j) :  6/256
     4j :  1/256
(-2+2j) :  6/256
(-1-3j) :  4/256
 (1-3j) :  4/256
(-3+1j) :  4/256
(-1+1j) : 24/256
 (1+1j) : 24/256
 (3+1j) :  4/256
(-1+3j) :  4/256
 (1+3j) :  4/256
(-1-1j) : 24/256
     -4 :  1/256
    -4j :  1/256
(-3-1j) :  4/256
(-2-2j) :  6/256
    -2j : 16/256
 (1-1j) : 24/256
 (2-2j) :  6/256
 (3-1j) :  4/256
     -2 : 16/256
```

Here is how to get the probability that, after four time steps, the body is inside a circle of radius 2, centred on the origin (remind that abs(_c_) gives the modulus of the complex number _c_):

```
lea> abs(?[4]b) <= 2
False : 15/64
 True : 49/64
```

To get the full distribution of the body's distance from the origin:

```
lea> abs(?[4]b)
            0 :  9/64
1.41421356237 : 24/64
            2 : 16/64
2.82842712475 :  6/64
3.16227766017 :  8/64
            4 :  1/64
```

You can check the consistency of the last results since 49/64 = (9+24+16)/64.


# RPG combat #

Imagine a Role-Playing Game combat situation opposing you (a warrior) against a giant troll. We shall use here a simplified version of the "d20 System" (Open Game License). To know whether you can hit the troll, you have to calculate your "attack roll" (AR) and the "armour class" (AC) of the troll; if your AR is greater or equal to the AC of the troll, then you hit him and causes him some "damage".

Let us first define the dice we need (using RPG standards: "D6", "2D6" and "D20"):
```
lea> D6 = ?(range(1,7))
lea> _2D6 = ?[2]D6
lea> D20 = ?(range(1,21))
```

On a given combat round, you hit the troll if your attack roll (D20 + bonus of 8) is greater or equal to the troll's armour class (24):

```
lea> targetArmorClass = 24
lea> attackRoll = D20 + 8
lea> hit0 = attackRoll >= targetArmorClass
lea> hit0
False : 3/4
 True : 1/4
```

Well,... but we forget exceptions to the the rule! It says that if the D20 throw is 1, then there is a certain miss; also, if the D20 throw is 20, then there is a certain hit. Let's make the correction with a CPT (Conditional Probability Table) :

```
lea> hit = ?! ( D20 ==  1 -> True ,
 ...            D20 == 20 -> False,
 ...                    _ -> hit0 )
```

Consider this as the probabilistic version of an if-then-else!  Note that, if you want to avoid the CPT, then you can have the same result using a boolean expression:
```
lea> hit = (hit0 & (D20 != 1)) | (D20 == 20)
```

In both cases, the result is unchanged:
```
lea> hit
False : 3/4
 True : 1/4
```
Actually, that does not change anything here (but it could have on a more unbalanced combat!).


Now, if you can hit the troll, your magic sword shall cause him a damage of 2D6 + a bonus of 3:

```
lea> damageIfHit = _2D6 + 3
lea> damageIfHit
 5 : 1/36
 6 : 2/36
 7 : 3/36
 8 : 4/36
 9 : 5/36
10 : 6/36
11 : 5/36
12 : 4/36
13 : 3/36
14 : 2/36
15 : 1/36
lea> damageIfHit.mean
10.0
```

If you are lucky to hit the troll, then the damage average is 10 points.

If you want to know the overall damage distribution considering that the hit is uncertain, then the most natural is to define a CPT :
```
lea> damage = ?! ( hit -> damageIfHit,
 ...                 _ -> 0          )
```

An alternative avoiding CPT is to the following expression, which uses the "absorbing" property of zero:
```
lea> damage = hit.map(lambda h: 1 if h else 0) * damageIfHit
```

In both cases, the resulting distribution of damage is as follows:
```
lea> damage 
 0 : 108/144
 5 :   1/144
 6 :   2/144
 7 :   3/144
 8 :   4/144
 9 :   5/144
10 :   6/144
11 :   5/144
12 :   4/144
13 :   3/144
14 :   2/144
15 :   1/144
lea> damage.mean
2.5
```

We see that the damage average has lowered to 2.5 points, since the chance to hit is 1/4.
Let us evaluate total damage distribution after 2 rounds of combat (with no reaction from the troll!):

```
lea> ?[2]damage
 0 : 11664/20736
 5 :   216/20736
 6 :   432/20736
 7 :   648/20736
 8 :   864/20736
 9 :  1080/20736
10 :  1297/20736
11 :  1084/20736
12 :   874/20736
13 :   668/20736
14 :   467/20736
15 :   272/20736
16 :    80/20736
17 :   104/20736
18 :   125/20736
19 :   140/20736
20 :   146/20736
21 :   140/20736
22 :   125/20736
23 :   104/20736
24 :    80/20736
25 :    56/20736
26 :    35/20736
27 :    20/20736
28 :    10/20736
29 :     4/20736
30 :     1/20736
```

If current troll's health point is 4, the probability to kill him in the next two rounds is calculated as follows:

```
lea> healthPoints = 4
lea> healthPoints - ?[2]damage <= 0
False : 9/16
 True : 7/16
```

To end, let's simulate 20 "real" rounds of combat:

```
lea> damage $(20)
(0, 0, 0, 0, 11, 0, 9, 7, 0, 0, 0, 10, 0, 0, 13, 0, 0, 9, 0, 0)
```

The reader can develop the case to calculate troll's attacks, to evaluate health points evolution after damages, simulate combats until dead, etc.

# rock-paper-scissors game #

Let us play the well-known rock-paper-scissors game! First, let us define a class that allows us to represent the three objects and the nontransitive win relationship:

```
class RPS(object):
    
    _curVal = 0
    
    def __init__(self,name):
        object.__init__(self)
        self.name = name
        self.val = RPS._curVal
        RPS._curVal += 1
        
    def __str__(self):
        return self.name
        
    __repr__ = __str__
    
    def __cmp__(self,other):
        delta = other.val - self.val
        if delta == 0:
            return 0
        if delta == 1 or delta == -2:
            return -1
        return +1 

r = RPS('rock')
p = RPS('paper')
s = RPS('scissors')
```

We can verify the win conditions by comparing the objects together:

```
lea> (r,p,s)
(rock, paper, scissors)
lea> r < p
True
lea> p < s
True
lea> s < r
True
```

Assuming that players follow a random, independent choice at each turn of play, let us define 3 behaviours:
  1. _balanced_ : the 3 objects have equal probabilities
  1. _morerock_ : the rock has twice more chances to be chosen than the others
  1. _morepaper_ : the paper has twice more chances to be chosen than the others

```
lea> balanced  = ?{r: 1, p: 1, s: 1}
lea> morerock  = ?{r: 2, p: 1, s: 1}
lea> morepaper = ?{r: 1, p: 2, s: 1}
lea> balanced
    rock : 1/3
   paper : 1/3
scissors : 1/3
lea> morerock
    rock : 2/4
   paper : 1/4
scissors : 1/4
lea> morepaper
    rock : 1/4
   paper : 2/4
scissors : 1/4
```

Now, we are able to compare these behaviours to get the probabilities of win. The easier approach is to use the usual comparison operators. For example, the chances of win of _morepaper_ over _morerock_ are given by

```
lea> morepaper > morerock
False : 5/8
 True : 3/8
```

The advantage of _morepaper_ is not blatant because this result does not show the probability of tied game. This can be found as follows

```
lea> morepaper == morerock
False : 11/16
 True :  5/16
```

A better approach consists in using the cmp(_a_,_b_) method, defined in Python 2.x : it returns -1, 0 or +1 depending whether _a_ is less than, equal to or greater than _b_, respectively.

For Python 3.x, since cmp is not defined, you may define it yourself or use the following trick:
```
lea> cmp = RPS.__cmp__
```

> The new approach requires to use the map method to be able to apply the cmp function on _morepaper_'s values, providing _morerock_'s values as second argument. We can then have the full picture of the results at once:

```
lea> ?cmp(morepaper,morerock)
-1 : 5/16
 0 : 5/16
 1 : 6/16
```

We see now clearly the advantage of choosing more rocks against an opponent that choose more papers. In the present case, it wins with probability 6/16 and loose with probability 5/16.

The skeptical reader can use the cartesian product to verify this results by consulting each possible draws of the game and add the atomic probabilities :

```
lea> ?*(morepaper,morerock)
      (paper, paper) : 2/16
    (scissors, rock) : 2/16
   (scissors, paper) : 1/16
(scissors, scissors) : 1/16
        (rock, rock) : 2/16
       (rock, paper) : 1/16
    (rock, scissors) : 1/16
   (paper, scissors) : 2/16
       (paper, rock) : 4/16
```

An even more practical way to check atomic results consists in letting Lea provides directly game results by injecting it in the cartesian product (as first position, so the ordering makes a smart grouping - but this works only on Python 2.x!) :

```
lea> ?*(?cmp(morepaper,morerock),morerock,morepaper)
   (-1, rock, scissors) : 2/16
      (-1, paper, rock) : 1/16
  (-1, scissors, paper) : 2/16
      (0, paper, paper) : 2/16
(0, scissors, scissors) : 1/16
        (0, rock, rock) : 2/16
   (1, paper, scissors) : 1/16
    (1, scissors, rock) : 1/16
       (1, rock, paper) : 4/16
```

Since the cmp method provides -1, 0, +1 values, it is easy to get the distribution of fails, ties and wins after a given number of games. For example, here are the scores after 3 games, where values indicates the difference won games - lost games from the "morerock" player's point of view:

```
lea> ?[3]cmp(morepaper,morerock)
-3 :  125/4096
-2 :  375/4096
-1 :  825/4096
 0 : 1025/4096
 1 :  990/4096
 2 :  540/4096
 3 :  216/4096 
```


Now, are there better strategies against _morepaper_?

Yes! In particular,

  * choose paper with 8 more chances than the others:

```
lea> evenmorepaper = ?{r: 1, p: 8, s: 1}
lea> ?cmp(evenmorepaper,morerock)
-1 : 11/40
 0 : 11/40
 1 : 18/40
```

  * choose always paper:

```
lea> alwayspaper = p
lea> ?cmp(alwayspaper,morerock)
-1 : 1/4
 0 : 1/4
 1 : 2/4
```


Now, what about the _balanced_ behaviour?

  * against _morerock_:

```
lea> ?cmp(balanced,morerock)
-1 : 1/3
 0 : 1/3
 1 : 1/3
```

  * against a fixed choice (e.g. paper):

```
lea> ?cmp(balanced,alwayspaper)
-1 : 1/3
 0 : 1/3
 1 : 1/3
```

  * against itself:

```
lea> ?cmp(balanced,?balanced)
-1 : 1/3
 0 : 1/3
 1 : 1/3
```

We demonstrate in these last few examples the known (and proven) fact that the balanced strategy gives a tied game on the long term, whatever opponent's behaviour. It is actually _the_ best strategy when nothing is known about the opponent's behaviour.

# von Neumann extractor #

Imagine you have a physical device able to produce a "true" random stream of bits. Great! Unfortunately, this device has a bias : the probability of having 1 or 0, although constant in time, is not 1/2; furthermore, this probability is unknown.

Question: _Can you find a mechanism to remove this bias and produce a stream of true random bits, with balanced probabilities (1/2)?_

John von Neumann has invented a simple yet awesome trick to do the job. See
[von Neumann extractor](http://en.wikipedia.org/wiki/Randomness_extractor#Von_Neumann_extractor).

Let us demonstrate the algorithm of von Neumann. Since we don't have the physical random device at our disposal, we shall first simulate the biased stream of bits. We use Lea to build an infinite random bits iterator, assuming a probability of 80% to have a 0 (we assume here that the reader is acquainted with Python's generators):

```
def genBiasedRandom():
    source = ?{0: 80, 1: 20}
    while True:
        yield source.random()

biasedRandomIter = genBiasedRandom()
```

We can make a trial by taking a sample of 500 bits on this biased random source and control the frequency:

```
lea> from itertools import islice
lea> biasedSample = tuple(islice(biasedRandomIter,500))
lea> biasedSample 
(0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 1, 1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 1, 1, 0, 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 1, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 1, 0, 0, 0, 0)
lea> :% ?(biasedSample)
0 :  79.4 %
1 :  20.6 %
```

We see that the frequency of 0 is close to 80 %, as expected.

Now, we build the von Neumann extractor, as an infinite random bits iterator that takes the biased source as input:

```
from itertools import islice

def genVonNeumannExtractor(randomIter):
    while True:
        (b0,b1) = islice(randomIter,2)
        if b0 != b1:
            yield b0

vonNeumannExtractorIter = genVonNeumannExtractor(biasedRandomIter)
```

Let us make a try, taking a sample of 500 bits from this device (note that the process shall likely consume a lot more bits on the biased source!) :

```
lea> unbiasedSample = tuple(islice(vonNeumannExtractorIter,500))
lea> unbiasedSample
(0, 1, 1, 0, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 1, 0, 1, 0, 1, 0, 0, 0, 1, 1, 0, 1, 0, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 0, 1, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 0, 1, 1, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 0, 0, 0, 1, 1, 0, 1, 0, 0, 1, 1, 0, 0, 1, 0, 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 1, 1, 1, 0, 1, 0, 0, 1, 0, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 0, 1, 0, 1, 1, 0, 0, 1, 0, 0, 1, 1, 1, 0, 0, 0, 1, 0, 1, 1, 1, 0, 0, 1, 0, 1, 0, 1, 1, 1, 0, 0, 1, 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 0, 0, 1, 1, 1, 0, 1, 1, 0, 0, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 1, 0, 0, 0, 1, 0, 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 1, 1, 1, 0, 0, 0, 0, 1, 1, 0, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 0, 0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1, 1, 0, 1, 0, 1, 0, 0, 1, 0, 0, 0, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 1, 0, 0, 0, 1, 0, 1, 0, 1, 0, 1, 0, 0, 0, 0, 1, 1, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 1, 0, 1, 0, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1)
lea> :% ?(unbiasedSample)
0 :  47.2 %
1 :  52.8 %
```

We can see that the frequencies are now close to 50%, as expected.

Thank you, _Doctor Miraculis_ !


# central limit law #

_a contribution of Nicky van Foreest_

Requires: [SciPy](http://www.scipy.org/), [numpy](http://www.numpy.org/) and [matplotlib](http://matplotlib.org/) (see [SciPy Stack](http://www.scipy.org/install.html))

From theory we know that the distribution of the sum of identical and independent random variables converges to the normal distribution. We can test this with Lea.

First we need some basic inputs.

```
lea> import numpy as np
lea> from scipy.stats import norm
lea> import matplotlib.pylab as plt
lea> from lea import Lea
```

Now make a uniform die with four sides and take the 5 fold sum.

```
lea> die = ?(0, 1, 2, 3)
lea> dieN = ?[5]die
```

We need the  support and the cumulative distribution of `dieN`.

```
lea> T = np.array(zip(*dieN.integral()))
lea> # the support is stored in T[0,:]
lea> supp = T[0,:]
lea> # and the distribution in T[1,:]
lea> F = T[1,:]
```

As yet, `F` is a distribution function, but not a probability
distribution function, that is bounded between 0 and 1. For this we
need to normalize `F`.

```
lea> F = F/float(F[-1])
```

The last step is to compare `F` to the cumulative distribution of the
normal distribution. We need a continuity correction, for otherwise
the normal distribution seems to be shifted to the right.

```
lea> mu = dieN.mean
lea> std = dieN.std
lea> FN = norm(loc = mu , scale = std).cdf(supp + 0.5)
```

Note for Lea 1.x users: use the syntax `.mean()` and `.stdev()` !

Let's compare.

```
lea> np.max(np.abs(F-FN))
0.0049414764166033631
```

This is truly astonishing. A 5-fold convolution is already this good.


# bullshit generator #

Some years ago, out of desperation, I wrote a [bullshit generator in Python](https://mail.python.org/pipermail/python-list/2009-March/543633.html).

Here is a typical sample of what it produces:

---

`As a matter of fact, the engine is provided by the interface where the actor on top of a peer-to-peer interoperability under the specification derivation rule is performed by the available SSO. The timing thread is generated by a full XML when the Web 2.0 that executes the issuer is generated by the artifact. The world-leading Web browser orchestrates a SOAP message. An aware issuer is envisioned in the generic design operator, which competes with the factory, in the aggregator that is unaffected. The authorized entity provides an interface to the fat client. The engine updates the integration. The algorithm of the model that is efficient sent to a technology thin client, which triggers a cluster, is provided by the version. The approach is repeatedly required by the action timing security under a space when the ancillary owner generates the XML-based UML model.`

---


The program works with a a small dictionary of words and a simplified English grammar (very close to the one wired in my brain, since I am not a native English speaker!). The words are classified in categories -nouns, verbs, etc-; to simplify, these words are chosen with equal probability, which was easily done with the built-in `random.choice` of Python (see `TerminalNode` class). For the grammar rules however, in order to avoid too long sentences, I put a probability weight on each derivation rule, so to make the simplest constructions the likeliest. Part of the complexity of the program was then in the algorithm to make the random choices according to the probability weights (see `Node` class).

In what follows, I show how to use Lea to greatly simplify the `Node` and `TerminalNode` classes. The statements to define English words and grammars are unchanged from the original version, excepting the swapping of probability weights and grammar rules to accommodate the `Lea.fromValFreqs` method. Note that Python's `random` module is no longer imported (although used by Lea internally).

```
'''
======================================================================
 Bullshit Generator 
    by Pierre Denis, 2009, 2014
======================================================================
'''

# --------------------------------------------------
# grammar engine
# --------------------------------------------------

from lea import Lea

class Node(object):

    def setTermsChoices(self,*termsChoices):
        self.termsChoices = Lea.fromValFreqs(*termsChoices)
        
    def getWords(self):
        terms = self.termsChoices.random()
        for term in terms:
            if isinstance(term,str):
                yield term
            else:
                for word in term.getWords():
                    yield word

    def getString(self):
        res = " ".join(self.getWords())
        res = ", ".join(w.strip() for w in res.split(",") if w.strip())
        if res.endswith(", "):
            res = res[:-2]
        return res[0].upper() + res[1:] + "."


class TerminalNode(object):

    def __init__(self,*words):
        self.words = ?(*words)

    def getWords(self):
        yield self.words.random()

# --------------------------------------------------
# grammar
# --------------------------------------------------

verb = TerminalNode(
    "accesses", "activates", "administrates", "aggregates", "builds",
    "calculates", "checks", "competes with", "completes", "complies with",
    "controls", "covers", "delivers", "dispatches", "eases", "encapsulates",
    "encompasses", "executes", "extracts", "features",
    "generates", "gets", "governs", "guides", "has", "increases",
    "inherits from", "is", "keeps track of", "leverages", "makes",
    "manages",
    "manages", "maximizes", "mitigates", "monitors", "must have", "needs",
    "offers", "opens", "operates on", "optimizes", "orchestrates",
    "overwrites", "performs", "populates", "precludes", "provides",
    "provides",
    "provides an interface to", "reads", "receives", "reduces",
    "reduces the need of", "registers", "regulates", "relies on",
    "requires",
    "resides on", "resides within", "retrieves", "retrieves the data in",
    "runs on",
    "schedules", "integrates with", "sends", "shall be",
    "shall have", "should be", "should have", "starts", "stores",
    "streamlines", "subscribes to", "subscribes to", "supersedes", "takes",
    "targets", "triggers", "updates", "validates", "writes")

passiveVerb = TerminalNode(
    "accessed by", "achieved by", "aggregated by", "applicable for",
    "asserted by", "authorized by",
    "based upon", "built from", "built upon", "collected by",
    "controlled by",
    "dedicated to", "deployed on", "derived from", "dispatched by",
    "driven by", "eased by", "enabled by", "envisioned in",
    "extracted from", "generated by", "in the scope of", "installed on",
    "integrated in",
    "located in", "managed by", "maximized by", "monitored by", "opened by",
    "optimized by", "orchestrated by", "packaged in", "performed by",
    "populated by", "processed by", "provided by", "provided by",
    "received by", "refreshed by", "registered in", "related to",
    "required by",
    "responsible for", "scheduled by", "sent to", "serialized by",
    "serialized in", "started in", "stored by", "stored in", "stored on",
    "the interface of", "updated by", "validated by")

aSimpleName = TerminalNode(
    "COTS", "GRID processing",
    "Java program", "LDAP registry", "Portal", "RSS feed", "SAML token",
    "SOAP message", "SSO", "TCP/IP", "UML model", "URL",
    "W3C", "Web", "Web 2.0", "Web browser", "Web page",
    "Web service", "back-end", "backbone", "bandwidth", "bean",
    "bridge", "browser", "bus", "business", "business model", "call",
    "catalogue", "class", "client", "cluster", "collection",
    "communication", "component", "compression",
    "concept", "conceptualization", "connexion", "console", "content",
    "context", "cookie", "customization", "data", "database",
    "datastore", "deployment",
    "derivation rule", "design", "development", "device", "directory",
    "discovery", "dispatcher", "document", "domain", "factory",
    "fat client", "feature", "file", "form", "frame", "framework",
    "function", "gateway", "genericity", "geomanagement", "goal",
    "governance", "granularity", "guideline", "header", "key", "layer",
    "leader", "library", "link", "list", "log file", "logic",
    "look-and-feel",
    "manager", "market", "mechanism", "message", "meta-model",
    "metadata", "model", "modeling", "module", "network", "performance",
    "persistence", "personalization", "plug-in", "policy", "port",
    "portal", "practice",
    "presentation layer", "privacy", "private key", "procedure",
    "process", "processor", "processing", "product", "protocol",
    "recommendation",
    "registration", "registry", "relationship", "resource",
    "responsibility", "role",
    "rule", "scenario", "scenario", "scheduler", "schema", "security",
    "server", "service", "service provider", "servlet", "session",
    "skeleton", "software", "solution", "source", "space",
    "specification", "standard", "state", "statement", "streaming",
    "style sheet", "subscriber", "subsystem", "system", "system",
    "table", "target", "task", "taxonomy", "technique", "technology",
    "template", "thin client", "thread", "throughput", "timing", "tool",
    "toolkit", "topic", "unit", "usage", "use case", "user",
    "user experience", "validation", "value", "version", "vision", "work",
    "workflow")

anSimpleName = TerminalNode(
    "API", "IP address", "Internet", "UDDI", "XML", "XML file",
    "abstraction", "access", "acknowledgment", "action", "actor",
    "administrator", "aggregator", "algorithm", "application", "approach",
    "architecture", "artifact", "aspect", "authentication", "availability",
    "encapsulation", "end-point", "engine", "engine", "entity",
    "entity", "environment", "event", "identifier", "information",
    "integration", "interface", "interoperability", "issuer", "object",
    "ontology", "operation", "operator", "operator", "opportunity",
    "orchestration", "owner")

aAdjective = TerminalNode(
    "BPEL",  "DOM", "DTD", "GRID", "HTML", "J2EE",
    "Java", "Java-based", "Java-based", "UML", "SAX", "WFS", "WSDL",
    "basic", "broad", "bug-free",
    "business-driven", "client-side", "coarse", "coherent", "compatible",
    "complete", "compliant", "comprehensive", "conceptual", "consistent",
    "control", "controller", "cost-effective",
    "custom", "data-driven", "dedicated", "distributed", 
    "dynamic", "encrypted", "event-driven", "fine-grained", "first-class",
    "free", "full",
    "generic", "geo-referenced", "global", "global", "graphical",
    "high-resolution", "high-level", "individual", "invulnerable",
    "just-in-time", "key",
    "layered", "leading", "lightweight", "logical", "main", "major",
    "message-based",
    "most important", "multi-tiers", "narrow", "native", "next",
    "next-generation",
    "normal", "password-protected", "operational", "peer-to-peer",
    "performant", "physical",
    "point-to-point", "polymorphic", "portable", "primary", "prime",
    "private", "proven", "public", "raw", "real-time", "registered",
    "reliable", "remote",
    "respective", "right", "robust", "rule-based", "scalable", "seamless",
    "secondary", "semantic", 
    "server-side", "service-based", "service-oriented", "simple", "sole",
    "specific", "state-of-the-art", "stateless", "storage", "sufficient",
    "technical", "thread-safe", "uniform", "unique", "used", "useful",
    "user-friendly", "virtual", "visual", "web-based", "web-centric",
    "well-documented", "wireless", "world-leading", "zero-default")

anAdjective = TerminalNode(
    "AJAX", "OO", "XML-based", "abstract", "ancillary", "asynchronous",
    "authenticated", "authorized", "auto-regulated", "available", "aware",
    "efficient",
    "international", "interoperable", "off-line", "official", "online",
    "open", "operational",
    "other", "own", "unaffected", "up-to-date")

adverb = TerminalNode(
    "basically", "comprehensively", "conceptually", "consistently",
    "definitely", "dramatically",
    "dynamically", "expectedly", "fully", "generally", "generically",
    "globally", "greatly", "individually", "locally", "logically",
    "mainly", "mostly", "natively",
    "officially", "physically", "practically", "primarily",
    "repeatedly", "roughly", "sequentially", "simply", "specifically", 
    "surely", "technically", "undoubtly", "usefully", "virtually")
                            
sentenceHead = TerminalNode(
    "actually", "as a matter of fact", "as said before", "as stated before",
    "basically", "before all", "besides this", "beyond that point",
    "clearly",
    "conversely", "despite these facts", "for this reason",
    "generally speaking",
    "if needed", "in essence", "in other words", "in our opinion",
    "in the long term", "in the short term", "in this case", "incidentally",
    "moreover", "nevertheless", "now", "otherwise", "periodically",
    "roughly speaking", "that being said", "then", "therefore",
    "to summarize", "up to here", "up to now", "when this happens")

(name, aName, anName, nameTail, adjective, nameGroup,
 simpleNameGroup, verbalGroup, simpleVerbalGroup, sentence,
 sentenceTail) = [Node() for i in range(11)]

aName.setTermsChoices(
    (( aSimpleName,      ), 50 ),
    (( aSimpleName, name ),  5 ),
    (( aSimpleName, name ),  8 ),
    (( aName, nameTail   ),  5 ))

anName.setTermsChoices(
    (( anSimpleName,      ), 50 ),
    (( anSimpleName, name ),  8 ),
    (( anName, nameTail   ),  5 ))

nameTail.setTermsChoices(
    (( "of", nameGroup        ), 2 ),
    (( "from", nameGroup      ), 2 ),
    (( "under", nameGroup     ), 1 ),
    (( "on top of", nameGroup ), 1 ))

name.setTermsChoices(
    (( aName,  ), 1 ),
    (( anName, ), 1 ))

adjective.setTermsChoices(
    (( aAdjective,  ), 1 ),
    (( anAdjective, ), 1 ))

nameGroup.setTermsChoices(
    (( simpleNameGroup,                                   ), 10 ),
    (( simpleNameGroup, passiveVerb, nameGroup            ),  1 ),
    (( simpleNameGroup, "that", simpleVerbalGroup         ),  1 ),
    (( simpleNameGroup, ", which", simpleVerbalGroup, "," ),  1 ))

simpleNameGroup.setTermsChoices(
    (( "the", name             ), 40 ),
    (( "the", adjective, name  ), 20 ),
    (( "a", aName              ), 10 ),
    (( "an", anName            ), 10 ),
    (( "a", aAdjective, name   ),  5 ),                
    (( "an", anAdjective, name ),  5 ))  

verbalGroup.setTermsChoices(
    (( verb, nameGroup                      ), 10 ),
    (( adverb, verb, nameGroup              ),  1 ),
    (( "is", passiveVerb, nameGroup         ), 10 ),
    (( "is", adverb, passiveVerb, nameGroup ),  1 ),
    (( "is", adjective                      ),  1 ),
    (( "is", adverb, adjective              ),  1 ))

simpleVerbalGroup.setTermsChoices(
    (( verb, simpleNameGroup ), 2 ),
    (( "is", adjective       ), 1 ))

sentence.setTermsChoices(
    (( nameGroup, verbalGroup                     ), 20 ),
    (( sentenceHead, "," , nameGroup, verbalGroup ),  4 ),
    (( sentence, sentenceTail                     ),  4 ))

sentenceTail.setTermsChoices(
    (( "in", nameGroup                 ), 12 ),
    (( "within", nameGroup             ),  5 ),
    (( "where", nameGroup, verbalGroup ),  5 ),
    (( "when", nameGroup, verbalGroup  ),  5 ),
    (( "because it", verbalGroup       ),  2 ),
    (( "; that's why it", verbalGroup  ),  1 ))

# --------------------------------------------------
# main program
# --------------------------------------------------
try:
    import win32com.client
    voice = win32com.client.Dispatch("sapi.SPVoice")
except:
    voice = None

print ("press <enter> to resume, 'q'+<enter> to quit\n")

while True:
    print ('')
    for i in range(8):
        generatedSentence = sentence.getString()
        # PY3: print (generatedSentence,end='')
        print generatedSentence,
        if voice:
            voice.speak(generatedSentence)
    # PY3: cmd = input()
    cmd = raw_input()        
    if cmd.strip().lower() == "q":
        break
```

Note: for Python 3 users, just replace the two statements marked with `# PY3:`

You can run this program and verify that it produces the same kind of gobbledygook than the original version. Let him the last word:

---

`The official operator is performed by the most important actor. The interoperability is off-line. The environment is authorized by the rule-based identifier that needs the availability thread. The W3C is located in a high-resolution style sheet because it is surely point-to-point. A session sends the use case owner that guides the asynchronous content. The approach aggregator, which shall have the algorithm, shall be an identifier. The OO link administrates a controller information that executes an identifier when the thread-safe opportunity engine user experience that populates a bridge roughly extracts the most important API. The environment is derived from the dispatcher from a unit acknowledgment of an interface.`

---
