!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
**_Caution: work in progress..._**
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

# Lea's Philosophy #

This page presents the leading design ideas of Lea.

  1. Any Lea instance represents a discrete probability distribution.
  1. Such distribution is _immutable_.
  1. Most operations on Lea instances produces new Lea instances.
  1. The operations are performed in a _lazy_ way.
  1. Probabilities are defined and stored as natural numbers, representing rational numbers having a common denominator.
  1. Operations made on Lea instances are propagated to their values.
  1. Operations involving more than one Lea instance use the cartesian product.
  1. The same instance occurring several times in the same expression is treated consistently, through a value binding mechanism.
  1. Any non-Lea object mixed in an operation with a Lea instance is automatically coerced into a Lea instance representing a "certain" value.
  1. Lea is an abstract class.
  1. Each Lea's concrete subclasses defines a specific method to produce the values-probability pairs.
  1. The Alea subclass defines actual "atomic" value-probability pairs.