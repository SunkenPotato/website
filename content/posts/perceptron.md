+++
date = '2026-02-20T21:05:17+01:00'
draft = false
title = 'Perceptron'
tags = ['ml', 'math', 'perceptron']
+++

Hello! This is the first part of a relatively introductory series to Machine Learning/Artificial Intelligence, as well as 
the first part of the "Perceptron" subseries.

Today, I'd like to cover how a very basic Perceptron, or an artificial neuron works. They're the building blocks of
neural networks and play a significant role in the development of AI.
<!--more-->

Perceptrons were first emulated by Frank Rosenblatt in 1957 on an IBM 704, the first computer produced *en masse* with
support for FPA (Floating Point Arithmetic). Algebraically, a perceptron is quite simple. It can be thought of as the 
following function:
$$
  f(x) = w \cdot x + b
$$
where:
- \\(w, x \in \mathbb{R}^n \\)
- \\(b \in \mathbb{R}\\)

Obviously, \\(x\\) is the input. More specifically, it's a vector, so a "list" of inputs, which we multiply by our
weights vector \\(w\\) and then add the bias \\(b\\) to.

So, really, the perceptron is just a fancy linear function, which we all learned in seventh grade with the function \
\\(y = mx + b\\). To actually extract some functionality from this perceptron though, it might be helpful to think back
to how an actual neuron works. Given some set of inputs, a neuron either does or does not fire. We can express this as
`1` or `0`. 

But the function we just defined up there isn't guaranteed to return either 1 or 0!

That's true, which is why after we apply the first function to our inputs, we pass it through a special function you 
may or may not have heard of: \\(sgn\\). If you haven't, here's a quick definition:

> \\(sgn\\) returns either `-1` or `1` depending on the sign of `x`. That is, if it's negative, it returns `-1`, and if it's positive,
> it returns `1`. The special edge case is \\(sgn(0)\\), which usually is equal to 0, but for perceptrons, it's easier to define it as
> -1.

So, mathematically, we've just modeled a neuron!

You might be asking now: "Okay, but what good is this? What can I do with it?". Which is most certainly a fair question. Perceptrons are 
only good for answering "yes" or "no" to a given question.

...which, when you think about it, seems kind of wrong to use "only".

You could ask the question "Is this an apple?" or "Is transaction this credit card fraud?" or 
"Based on the attendance hours, grades, and missing homework of this student, is it likely that they will fail?"

That concludes the first part of the perceptron series. In the next post, I'll try to explain how we actually obtain those magical 
values \\(w\\) and \\(b\\) by training the Perceptron.

Have a nice week! 

sp.