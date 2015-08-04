
---


---


---

**_WARNING: The present page is no longer maintained!_**

**_You are invited to jump to [the new version](LeappSnippets.md)._**

---


---


---



# Next step: Lea PPL ... #

**COMING SOON...**

The expressions given in the Lea tutorial and example wiki pages are pure Python expressions. This works fine but it requires typing lot of method names and parentheses. This is normal and perfectly acceptable since Python is a _general-purpose language_. What would be nice is a _domain-specific language_ (_DSL_) dedicated to probabilities, which would be more concise and more expressive. The right term actually would be _probabilisitic programming language_, or _PPL_ - see [PROBABILISTIC-PROGRAMMING.org](http://probabilistic-programming.org/wiki/Home). I (Pierre Denis) have programmed a prototype that provides such PPL. It is basically a console interpreter that extends Python with a couple of symbols. Each typed statement is parsed and translated behind the scene into a pure Python statement that calls Lea method(s) ; this statement is then executed by the Python interpreter. Standard Python statements/expressions are accepted also, which makes the integration smooth.

This development will probably be integrated in the next major version of Lea (2.0). I haven't yet decided for a name for this language; candidates are "`leap`", "`leash`", "`Rachel`" or even "`RaShell`"...


---


# Lea PPL snippets #

What follow are session snippets made in this prototype Lea console interpreter. Note that, since this is still under development, the syntax and symbols can evolve.

`lea> ` is the prompt!

**REMINDER: THIS IS JUST A TEASER!** All this is not yet finalised nor published in current Lea package. It is hopeless to execute these statements now.


---




## Creating a probability distribution ##

```
lea> die1 = ?(1,2,3,4,5,6)
lea> die1
1 : 1/6
2 : 1/6
3 : 1/6
4 : 1/6
5 : 1/6
6 : 1/6
```

or, equivalently,

```
lea> die1 = ?([1,2,3,4,5,6])
lea> die1 = ?(range(1,7))
lea> die1 = ?(range(6)) + 1
lea> die1 = 1 + ?(range(6))
lea> die1 = ?(6-i for i in range(6))
lea> valuesTuple = (1,2,3,4,5,6)
lea> die1 = ?(valuesTuple)
lea> valuesIter = (6-i for i in range(6))
lea> die1 = ?(valuesIter)
lea> die1 = ?{1: 1, 2: 1, 3: 1, 4: 1, 5: 1, 6: 1}
lea> die1 = ?{1: 1/6, 2: 1/6, 3: 1/6, 4: 1/6, 5: 1/6, 6: 1/6}
```

Examples with strings:
```
lea> fairWeather = ?('sunny','cloudy','raining')
lea> fairWeather
 cloudy : 1/3
raining : 1/3
  sunny : 1/3
lea> weather = ?{'sunny': 68%, 'cloudy': 7%, 'raining': 25%}
lea> weather
 cloudy :  7/100
raining : 25/100
  sunny : 68/100
```

Since Python strings are iterable sequences, a letter frequency counter is made easy:

```
lea> ?('ALEA JACTA EST')
  : 2/14
A : 4/14
C : 1/14
E : 2/14
J : 1/14
L : 1/14
S : 1/14
T : 2/14
```

Note: to define a "singleton" distribution (certain value), append a coma (standard Python trick):

```
lea> ?('ALEA JACTA EST',)
ALEA JACTA EST : 1
```

Boolean random variables:

```
lea> flip = ?:(1/2)
lea> flip
False : 1/2
 True : 1/2
```

equivalent to

```
lea> flip = ?:(50%)
lea> flip = ?:(0.5)
lea> flip = ?(False,True)
lea> flip = ?{False: 1/2, True: 1/2}
lea> flip = ?{False: 50%, True: 50%}
lea> flip = ?{False: 0.5, True: 0.5}
```

and

```
lea> bflip = ?:(1/4)
lea> bflip
False : 3/4
 True : 1/4
```

equivalent to

```
lea> bflip = ?:(25%)
lea> bflip = ?:(0.25)
lea> bflip = ?{False: 3/4, True: 1/4}
...
```


---


## Cloning a distribution ##
(in order to have independent random variable)

```
lea> die2 = ?die1
```


---


## Adding distributions ##

```
lea> dice = die1 + die2
lea> dice
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

or, equivalently (well, if no conditional probability involved - see below!),

```
lea> dice = die1 + ?die1
lea> dice = sum((die1,die2))
lea> dice = sum(?die1 for i in range(2))
lea> dice = ?[2]die1
```


---


## Displaying probabilities ##

```
lea> :. dice
 2 : 0.027778
 3 : 0.055556
 4 : 0.083333
 5 : 0.111111
 6 : 0.138889
 7 : 0.166667
 8 : 0.138889
 9 : 0.111111
10 : 0.083333
11 : 0.055556
12 : 0.027778
lea> :% dice
 2 :   2.8 %
 3 :   5.6 %
 4 :   8.3 %
 5 :  11.1 %
 6 :  13.9 %
 7 :  16.7 %
 8 :  13.9 %
 9 :  11.1 %
10 :   8.3 %
11 :   5.6 %
12 :   2.8 %
lea> :% bflip
False :  75.0 %
 True :  25.0 %
```


---


## Extracting individual probabilities ##

```
lea> dice@7
1/6
lea> :. dice@7
0.16666666666666666
lea> :% dice@7
16.666667 %
lea> (dice <= 6) @True
5/12
lea> :% (dice <= 6) @True
41.666667 %
lea> bflip@True
1/4
lea> :. bflip@True
0.25
```

Note that `@True` can be shortcut as `@`:

```
lea> :% (dice <= 6) @
41.666667 %
lea> :. bflip@
0.25
```


---


## Evaluating conditions ##

```
lea> die1 == 2
False : 5/6
 True : 1/6
lea> die1 <= 2
False : 2/3
 True : 1/3
lea> %% die1 <= 2
False :  66.7 %
 True :  33.3 %
lea> (2 <= die1) & (die1 < 4)
False : 2/3
 True : 1/3
lea> ~((2 <= die1) & (die1 < 4))
False : 1/3
 True : 2/3
lea> (2 > die1) | (die1 >= 4)
False : 1/3
 True : 2/3
lea> (2 <= die1) & (die2 < 4)
False : 7/12
 True : 5/12
lea> (2 <= die1) & (dice < 4)
False : 35/36
 True :  1/36
lea> ?[50]die1 == 60
False : 808281277464764060643139600393722578321/808281277464764060643139600456536293376
 True :                             62813715055/808281277464764060643139600456536293376
lea> :. ?[50]die1 == 60
False : 1.000000
 True : 0.000000
```


---


## Calculating probability distribution indicators ##

```
lea> dice.mean
7.0
lea> dice.stdev
2.41522945769824
lea> dice.entropy
3.2744019192887714
```


---


## Generating random samples ##

```
lea> dice $
7
lea> dice $
5
lea> dice $(20)
(5, 4, 7, 10, 6, 9, 8, 6, 10, 4, 7, 6, 7, 6, 3, 10, 7, 5, 9, 6)
lea> flip $(20)
(False, True, True, False, True, True, False, False, False, True, False, True, True, True, False, True, False, False, True, True)
lea> bflip $(20)
(False, False, True, False, False, False, False, False, False, True, False, False, True, False, False, True, False, False, True, True)
```

Note that `dice $(20)` generates something _equivalent_ to the the results of any of the following expressions
```
lea> tuple(dice$ for i in range(20))
lea> tuple(die1$ + die2$ for i in range(20))
```
... but faster!

Controlling the frequencies in a random sample:
```
lea> ?(dice $(36000))
 2 :  931/36000
 3 : 2000/36000
 4 : 3065/36000
 5 : 3859/36000
 6 : 5088/36000
 7 : 6010/36000
 8 : 5010/36000
 9 : 3964/36000
10 : 3030/36000
11 : 2047/36000
12 :  996/36000
lea> :% ?(dice $(36000))
 2 :   2.7 %
 3 :   5.5 %
 4 :   8.6 %
 5 :  10.6 %
 6 :  13.6 %
 7 :  17.0 %
 8 :  13.9 %
 9 :  11.3 %
10 :   8.2 %
11 :   5.5 %
12 :   3.0 %
```

The return of Caesar, the latin parrot...
```
lea> latinParrot = ?('ALEA JACTA EST')
lea> ''.join(latinParrot $(30))
'TA   AECETAAATA AE ETJ AASLAAE'
```


---


## Calculating a cartesian product ##

```
lea> ?*(die1,die2)
(1, 1) : 1/36
(1, 2) : 1/36
(1, 3) : 1/36
(1, 4) : 1/36
(1, 5) : 1/36
(1, 6) : 1/36
(2, 1) : 1/36
...
(6, 5) : 1/36
(6, 6) : 1/36
lea> ?*(die1,die2,die1+die2)
 (1, 1, 2) : 1/36
 (1, 2, 3) : 1/36
 (1, 3, 4) : 1/36
...
(6, 5, 11) : 1/36
(6, 6, 12) : 1/36
```


---


## Applying a function ##

Putting a `?` in front of a function applied on Lea argument(s) builds a new Lea distribution, by applying the function on each inner values:

```
lea> 'die1 = ' + ?str(die1)
die1 = 1 : 1/6
die1 = 2 : 1/6
die1 = 3 : 1/6
die1 = 4 : 1/6
die1 = 5 : 1/6
die1 = 6 : 1/6
```

Note the difference if the `?` is omitted:
```
lea> 'die1 = ' + str(die1)
'die1 = 1 : 1/6\n2 : 1/6\n3 : 1/6\n4 : 1/6\n5 : 1/6\n6 : 1/6'
```
Without the `?` prefix, the `str` function is applied on the whole distribution, instead of individual values : the result is a standard Python string, not a Lea distribution.

User-defined functions

```
lea> def f(x,y):
 ...     return '%02d!' % (x+y)
lea> ?f(die1,die2)
02! : 1/36
03! : 2/36
04! : 3/36
05! : 4/36
06! : 5/36
07! : 6/36
08! : 5/36
09! : 4/36
10! : 3/36
11! : 2/36
12! : 1/36
```


---


## Applying an instance method ##

```
lea> die1.to_bytes(2,"big")
b'\x00\x01' : 1/6
b'\x00\x02' : 1/6
b'\x00\x03' : 1/6
b'\x00\x04' : 1/6
b'\x00\x05' : 1/6
b'\x00\x06' : 1/6
```


---


## Getting an instance attribute ##

```
lea> complexDice = die1 + die2*1j
lea> complexDice
(5+2j) : 1/36
(1+1j) : 1/36
(2+1j) : 1/36
...
(6+3j) : 1/36
(6+5j) : 1/36
lea> complexDice.imag
1.0 : 1/6
2.0 : 1/6
3.0 : 1/6
4.0 : 1/6
5.0 : 1/6
6.0 : 1/6
```


---


## Evaluating conditional probabilities ##

What follows the exclamation mark is a given fact, expressed as a condition expression. The expression before the exclamation mark is a distribution that is evaluated assuming that the given condition is certainly true.

```
lea> dice = die1 + die2
lea> dice ! dice < 5
2 : 1/6
3 : 2/6
4 : 3/6
lea> dice ! die1+die2 < 5
2 : 1/6
3 : 2/6
4 : 3/6
lea> dice ! (4 <= dice) & (dice < 7)
4 : 3/12
5 : 4/12
6 : 5/12
lea> dice ! (die1 <= 3) & (die2 <= 3)
2 : 1/9
3 : 2/9
4 : 3/9
5 : 2/9
6 : 1/9
lea> (5 <= dice) ! (die1 <= 3) & (die2 <= 3)
False : 2/3
 True : 1/3
lea> ?*(die1,die2) ! dice == 5
(1, 4) : 1/4
(2, 3) : 1/4
(3, 2) : 1/4
(4, 1) : 1/4
lea> ?*(die1,die2,dice) ! dice <= 5
(1, 1, 2) : 1/10
(1, 2, 3) : 1/10
(1, 3, 4) : 1/10
(1, 4, 5) : 1/10
(2, 1, 3) : 1/10
(2, 2, 4) : 1/10
(2, 3, 5) : 1/10
(3, 1, 4) : 1/10
(3, 2, 5) : 1/10
(4, 1, 5) : 1/10
```


---


## Indexing / slicing ##

Provided that the values in the distribution are indexable (e.g. strings, tuples), indexing made on a Lea instance is propagated in the inner values, obeying Python index numbering:

```
lea> name = ?('Gino', 'Guy', 'Jack', 'Lea', 'Leon', 'Loth', 'Lucienne', 'Pierre', 'Piotr', 'Rachel')
lea> name[0]
G : 2/10
J : 1/10
L : 4/10
P : 2/10
R : 1/10
lea> name[1]
a : 2/10
e : 2/10
i : 3/10
o : 1/10
u : 2/10
lea> name[-1]
a : 1/10
e : 2/10
h : 1/10
k : 1/10
l : 1/10
n : 1/10
o : 1/10
r : 1/10
y : 1/10
lea> name[0] + name[-1]
Go : 1/10
Gy : 1/10
Jk : 1/10
La : 1/10
Le : 1/10
Lh : 1/10
Ln : 1/10
Pe : 1/10
Pr : 1/10
Rl : 1/10
```

Slicing is also feasible:

```
lea> name[1:3]
ac : 2/10
ea : 1/10
eo : 1/10
ie : 1/10
in : 1/10
io : 1/10
ot : 1/10
uc : 1/10
uy : 1/10
lea> 'T'+name[1:] ! name[0]=='L'
     Tea : 1/4
    Teon : 1/4
    Toth : 1/4
Tucienne : 1/4
```

## Misc ##

### Playing cards ###

```
lea> cards = ?('A23456789TJQK') + ?('♥♣♦♠')
lea> cards
2♠ : 1/52
2♣ : 1/52
2♥ : 1/52
2♦ : 1/52
3♠ : 1/52
3♣ : 1/52
3♥ : 1/52
3♦ : 1/52
4♠ : 1/52
4♣ : 1/52
4♥ : 1/52
4♦ : 1/52
5♠ : 1/52
5♣ : 1/52
5♥ : 1/52
5♦ : 1/52
6♠ : 1/52
6♣ : 1/52
6♥ : 1/52
6♦ : 1/52
7♠ : 1/52
7♣ : 1/52
7♥ : 1/52
7♦ : 1/52
8♠ : 1/52
8♣ : 1/52
8♥ : 1/52
8♦ : 1/52
9♠ : 1/52
9♣ : 1/52
9♥ : 1/52
9♦ : 1/52
A♠ : 1/52
A♣ : 1/52
A♥ : 1/52
A♦ : 1/52
J♠ : 1/52
J♣ : 1/52
J♥ : 1/52
J♦ : 1/52
K♠ : 1/52
K♣ : 1/52
K♥ : 1/52
K♦ : 1/52
Q♠ : 1/52
Q♣ : 1/52
Q♥ : 1/52
Q♦ : 1/52
T♠ : 1/52
T♣ : 1/52
T♥ : 1/52
T♦ : 1/52
lea> cards[0]
2 : 1/13
3 : 1/13
4 : 1/13
5 : 1/13
6 : 1/13
7 : 1/13
8 : 1/13
9 : 1/13
A : 1/13
J : 1/13
K : 1/13
Q : 1/13
T : 1/13
lea> cards[1]
♠ : 1/4
♣ : 1/4
♥ : 1/4
♦ : 1/4
lea> cards $
'8♣'
lea> cards $_()
('K♣', '4♠', '7♥', 'T♥', 'A♥', '6♣', '6♦', '9♠', '5♣', 'Q♥', 'T♣', 'K♠', '4♥', 'T♦', '8♣', 'A♦', 'J♣', 'A♠', 'J♥', '3♠', '4♦', 'K♦', '8♥', '7♣', '8♦', '6♥', 'T♠', '5♥', '3♥', '9♦', 'J♦', '3♣', '2♥', '9♣', '7♠', 'Q♣', '5♦', '4♣', 'A♣', '9♥', '2♦', '6♠', 'K♥', '2♠', 'Q♦', '5♠', '7♦', 'J♠', 'Q♠', '8♠', '2♣', '3♦')
lea> cards $_(13)
('9♠', '4♦', 'K♠', '9♦', '6♦', 'A♦', '3♥', '8♣', 'T♥', '5♦', '7♦', '9♣', 'Q♥')
lea> ' '.join(cards $_(13))
'8♦ 7♦ 5♦ K♦ 6♠ K♠ T♥ 2♦ 3♠ 9♣ 7♥ 2♠ T♣'
lea> ?(cards$(52000))
2♠ : 1018/52000
2♣ :  993/52000
2♥ : 1001/52000
2♦ :  974/52000
3♠ :  952/52000
3♣ : 1047/52000
3♥ : 1056/52000
3♦ :  978/52000
4♠ : 1057/52000
4♣ :  995/52000
4♥ :  941/52000
4♦ : 1052/52000
5♠ :  972/52000
5♣ :  971/52000
5♥ :  954/52000
5♦ :  985/52000
6♠ : 1043/52000
6♣ :  979/52000
6♥ :  972/52000
6♦ :  970/52000
7♠ :  996/52000
7♣ : 1002/52000
7♥ : 1052/52000
7♦ : 1031/52000
8♠ :  991/52000
8♣ :  995/52000
8♥ : 1016/52000
8♦ : 1016/52000
9♠ :  990/52000
9♣ :  959/52000
9♥ : 1015/52000
9♦ :  989/52000
A♠ : 1030/52000
A♣ : 1004/52000
A♥ :  993/52000
A♦ : 1019/52000
J♠ : 1019/52000
J♣ :  963/52000
J♥ : 1024/52000
J♦ : 1046/52000
K♠ :  953/52000
K♣ :  984/52000
K♥ :  994/52000
K♦ : 1075/52000
Q♠ : 1042/52000
Q♣ :  967/52000
Q♥ :  935/52000
Q♦ :  977/52000
T♠ : 1029/52000
T♣ :  994/52000
T♥ : 1004/52000
T♦ :  986/52000
```

### Nice dice... ###

```
lea> diePic = ?('⚀⚁⚂⚃⚄⚅')
lea> diePic 
⚀ : 1/6
⚁ : 1/6
⚂ : 1/6
⚃ : 1/6
⚄ : 1/6
⚅ : 1/6
lea> ?[2]diePic 
⚀⚀ : 1/36
⚀⚁ : 1/36
⚀⚂ : 1/36
⚀⚃ : 1/36
⚀⚄ : 1/36
⚀⚅ : 1/36
⚁⚀ : 1/36
⚁⚁ : 1/36
⚁⚂ : 1/36
⚁⚃ : 1/36
⚁⚄ : 1/36
⚁⚅ : 1/36
⚂⚀ : 1/36
⚂⚁ : 1/36
⚂⚂ : 1/36
⚂⚃ : 1/36
⚂⚄ : 1/36
⚂⚅ : 1/36
⚃⚀ : 1/36
⚃⚁ : 1/36
⚃⚂ : 1/36
⚃⚃ : 1/36
⚃⚄ : 1/36
⚃⚅ : 1/36
⚄⚀ : 1/36
⚄⚁ : 1/36
⚄⚂ : 1/36
⚄⚃ : 1/36
⚄⚄ : 1/36
⚄⚅ : 1/36
⚅⚀ : 1/36
⚅⚁ : 1/36
⚅⚂ : 1/36
⚅⚃ : 1/36
⚅⚄ : 1/36
⚅⚅ : 1/36
lea> ''.join(diePic?(60))
'⚄⚃⚃⚅⚁⚂⚁⚁⚂⚅⚀⚃⚁⚂⚄⚀⚀⚃⚃⚁⚃⚀⚂⚃⚄⚁⚅⚄⚂⚄⚀⚄⚃⚀⚃⚂⚂⚀⚁⚀⚃⚃⚅⚄⚂⚅⚃⚅⚅⚀⚁⚅⚂⚁⚄⚁⚄⚃⚀⚂'
```


### Bit counter ###

variant 1 : from integer to string

```
lea> flip = ?(0,1)
lea> flip1 = ?(0,1)
lea> flip2 = ?flip1
lea> flip1 + flip2
0 : 1/4
1 : 2/4
2 : 1/4
lea> 'flip1 = ' + ?str(flip1) + ', flip2 = ' + ?str(flip2) + ' => flip1+flip2 = ' + ?str(flip1+flip2)
flip1 = 0, flip2 = 0 => flip1+flip2 = 0 : 1/4
flip1 = 0, flip2 = 1 => flip1+flip2 = 1 : 1/4
flip1 = 1, flip2 = 0 => flip1+flip2 = 1 : 1/4
flip1 = 1, flip2 = 1 => flip1+flip2 = 2 : 1/4
```

variant 2 : from string to integer

```
lea> c = ?('01')
lea> c4 = ?[4]c
lea> c4
0000 : 1/16
0001 : 1/16
0010 : 1/16
0011 : 1/16
0100 : 1/16
0101 : 1/16
0110 : 1/16
0111 : 1/16
1000 : 1/16
1001 : 1/16
1010 : 1/16
1011 : 1/16
1100 : 1/16
1101 : 1/16
1110 : 1/16
1111 : 1/16
lea> f = lambda s: sum(int(d) for d in s)
lea> count = ?f(c4)
lea> count
0 : 1/16
1 : 4/16
2 : 6/16
3 : 4/16
4 : 1/16
lea> ?*(c4,count)
('0000', 0) : 1/16
('0001', 1) : 1/16
('0010', 1) : 1/16
('0011', 2) : 1/16
('0100', 1) : 1/16
('0101', 2) : 1/16
('0110', 2) : 1/16
('0111', 3) : 1/16
('1000', 1) : 1/16
('1001', 2) : 1/16
('1010', 2) : 1/16
('1011', 3) : 1/16
('1100', 2) : 1/16
('1101', 3) : 1/16
('1110', 3) : 1/16
('1111', 4) : 1/16
lea> c4 + ' -> ' + ?str(count)
0000 -> 0 : 1/16
0001 -> 1 : 1/16
0010 -> 1 : 1/16
0011 -> 2 : 1/16
0100 -> 1 : 1/16
0101 -> 2 : 1/16
0110 -> 2 : 1/16
0111 -> 3 : 1/16
1000 -> 1 : 1/16
1001 -> 2 : 1/16
1010 -> 2 : 1/16
1011 -> 3 : 1/16
1100 -> 2 : 1/16
1101 -> 3 : 1/16
1110 -> 3 : 1/16
1111 -> 4 : 1/16
```

### Binary choices ###

```
lea> sendEMailToDREarlyNovember = ?:(99.9%)
lea> :% sendEMailToDREarlyNovember
False :   0.1 %
 True :  99.9 %
lea> 'I will ' + ?{'':99.9% ,'not ':0.1%} + 'send an e-mail to D. R. early November.'
I will not send an e-mail to D. R. early November. :   1/1000
    I will send an e-mail to D. R. early November. : 999/1000
```


---


## TODO ##

  * ~~custom output formatting (probabilities as decimal numbers or percentages)~~
  * extraction of individual probabilities as float or rational numbers
  * ~~easy access to indicators (mean, variance, etc)~~
  * dedicated syntaxes for creating/displaying boolean probability distributions
  * support for Lea advanced features ( revision, joint probabilities, ...)
  * ...