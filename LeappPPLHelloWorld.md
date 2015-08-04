# P("Hello world!") = 0.28 #

## a translation from Figaro to Leapp ##

Since version 2, Lea is bundled with _Leapp_, a _PPL_ (_Probabilistic Programming Language_) that provides the Lea user with a very concise syntax. Leapp is provided as an alternative to native Python syntax.

The claim that Leapp is a PPL requires some substantiation.

Probabilistic programming is a research field on its own, with multiple active developments (see [PROBABILISTIC-PROGRAMMING.org](http://probabilistic-programming.org/wiki/Home)). Of course, having a piece of software that makes probability calculations, even very convoluted ones, is not _per se_ indicative of PP. On the contrary, in our humble opinion, having the ability to model a probabilistic system in a natural, declarative way and let the inferences (convolutions, Bayes rules, etc) run behind the scene is one of the characteristics of PP.

Now, instead of providing scholastic arguments that Leapp _is_ a PPL, we propose hereafter a more pragmatic approach.

The afore-mentioned PROBABILISTIC-PROGRAMMING.org page makes a reference to the book ["Practical Probabilistic Programming"](http://www.manning.com/pfeffer/) of Avi Pfeffer. The first section of this book, entitled "Probabilistic Programming in a Nutshell", concludes with a "Hello World!" program, written in Scala and using Figaro. As explained by the author, this program features most essential ingredients of PP and is dramatically simpler than the Java's equivalent given before.

In the following, we present the exact translation of the Figaro's "Hello World!" in Leapp (see [Lea advanced tutorial: Part 2](LeappTutorial2.md)) :

```
sunnyToday = ?:(0.2)
greetingToday    = ?!(    sunnyToday -> ?{ "Hello world!": 0.6, "Howdy, universe!": 0.4 },
                                   _ -> ?{ "Hello world!": 0.2, "Oh no, not again": 0.8 })
sunnyTomorrow    = ?!(    sunnyToday -> ?:(0.8 ),
                                   _ -> ?:(0.05))
greetingTomorrow = ?!( sunnyTomorrow -> ?{ "Hello world!": 0.6, "Howdy, universe!": 0.4 },
                                   _ -> ?{ "Hello world!": 0.2, "Oh no, not again": 0.8 })

# predict
: 'Today’s greeting is "Hello world!" with probability...'
:.@ greetingToday == "Hello world!"

# infer
: 'If today’s greeting is "Hello world!", today’s weather is sunny with probability...'
:.@ sunnyToday ! greetingToday == "Hello world!"

# learn and predict
: 'If today’s greeting is "Hello world!", tomorrow’s greeting will be "Hello world!" with probability...'
:.@ (greetingTomorrow == "Hello world!") ! greetingToday == "Hello world!"
```

Here is the output of this Leapp program :

```
Today’s greeting is "Hello world!" with probability...
0.28
If today’s greeting is "Hello world!", today’s weather is sunny with probability...
0.42857142857142855
If today’s greeting is "Hello world!", tomorrow’s greeting will be "Hello world!" with probability...
0.3485714285714286
```

The reader can verify the correctness of these results [here](https://bigsnarf.wordpress.com/2014/05/24/probabilistic-programming-with-scala-hello-world/).

From this example, we have hopefully shown that Leapp exhibits the same core PPL features as Figaro's, hence that it can itself be tagged as a PPL. Of course, there is no doubt that Figaro and other PPL have far more functions to offer.

We provide hereafter some other query samples, which are meant to show that Lea / Leapp is not a carbon copy of an existing PPL.

```
: 'If tomorrow’s greeting is not "Hello world!", today’s and tomorrow’s weathers are both sunny with probability...'
:.@ (sunnyToday & sunnyTomorrow) ! greetingTomorrow != "Hello world!"

: 'If today’s and tomorrow’s weathers are both sunny, tomorrow’s greeting is not "Hello world!" with probability...'
:.@ (greetingTomorrow != "Hello world!") ! sunnyToday & sunnyTomorrow 

: 'If tomorrow’s greeting starts with a "H", today’s weather is sunny with probability...'
:.@ sunnyToday ! greetingTomorrow[0] == "H"

: 'Tomorrow’s greeting is the same as today’s greeting with probability...'
:.@ greetingTomorrow == greetingToday

: 'Tomorrow’s greeting probabilities are...'
:. greetingTomorrow

: 'If today’s greeting is "Hello world!", tomorrow’s greeting probabilities are...'
:. greetingTomorrow ! greetingToday == 'Hello world!'

: 'The full joint of today’s and tomorrow’s greeting probabilities is...'
:. ?*(greetingToday,greetingTomorrow)
```

which produces

```
If tomorrow’s greeting is not "Hello world!", today’s and tomorrow’s weathers are both sunny with probability...
0.08888888888888889
If today’s and tomorrow’s weathers are both sunny, tomorrow’s greeting is not "Hello world!" with probability...
0.4
If tomorrow’s greeting starts with a "H", today’s weather is sunny with probability...
0.4666666666666667
Tomorrow’s greeting is the same as today’s greeting with probability...
0.6096
Tomorrow’s greeting probabilities are...
    Hello world! : 0.280000
Howdy, universe! : 0.080000
Oh no, not again : 0.640000
If today’s greeting is "Hello world!", tomorrow’s greeting probabilities are...
    Hello world! : 0.348571
Howdy, universe! : 0.148571
Oh no, not again : 0.502857
The full joint of today’s and tomorrow’s greeting probabilities is...
        ('Hello world!', 'Hello world!') : 0.097600
    ('Hello world!', 'Howdy, universe!') : 0.041600
    ('Hello world!', 'Oh no, not again') : 0.140800
    ('Howdy, universe!', 'Hello world!') : 0.041600
('Howdy, universe!', 'Howdy, universe!') : 0.025600
('Howdy, universe!', 'Oh no, not again') : 0.012800
    ('Oh no, not again', 'Hello world!') : 0.140800
('Oh no, not again', 'Howdy, universe!') : 0.012800
('Oh no, not again', 'Oh no, not again') : 0.486400
```

We hope that these examples demonstrate that Leapp can do more than "Hello world!" and has genuine PPL genes!