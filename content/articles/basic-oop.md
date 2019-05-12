---
title: "Basic OOP"
date: 2019-03-12T12:46:12+08:00
tags: ["oop"]
draft: true
---

In a previous interview, I've been asked a basic question about OOP: "What are
the four pillars of OOP?". It made me feel really bad about not being able to
give an answer because I use those concepts everyday. I figure it's time for a
refresher.

## What is an object?  
Those books where they tell you that a cat is an object never helped me. Using
cats and dogs as objects does not make it easier for a someone struggling to
get a gist of what an object is.  Since we're builing software, I'll be using
real world examples.

Let's pretend we're working for a e-commerce company and we've been asked to
work on processing the payments. 


In this system we could have a **CreditCard** object. We could also have an
**Invoice** object. Some other objects we could add are **SalesOrder** or
**Product**.

An object is a representation of real-world objects inside our system.
Although, they're not limited to items in the real-world (e.g. data
structures).


## Class vs Object 
Hopefully you now get the gist of what an object is. But what is a class and
how does it differ from an object?

To put it simply: a class describes the object.

### Properties 
What makes a Credit Card?  You can't have a credit card without a **credit card
number**, **expiry date**, a **cvv**, and the **card owner's name**. These make
our class' properties.


### Methods 
What can we do the credit card? We can **charge** the card or **save** it for
later.  These are the methods. A method is something we can do to an object.
Usually methods names are actions.



## The four pillars of OOP

### Encapsulation
We don't want our functions and variables scattered all around the code base.
To encapsulate means to group the relevant variables (properties and functions
(methods) together.

A class is an example of encapsulation.

### Inheritance
When working with OOP you can extend classes. This allows the extending class
to carry over the properties and methods of the class that was extended from.

If we have **Mastercard** extend from **CreditCard**. We

### Abstraction
### Polymorphism
