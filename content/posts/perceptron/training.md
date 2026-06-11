+++
date = '2026-02-21T11:57:44+01:00'
draft = false
title = 'Perceptrons: Training (#2)'
tags = ['ml', 'math', 'perceptron']
+++
In [the last post](/posts/perceptron/introduction), I gave a relatively simple introduction and model for a perceptron. I feel like I 
didn't explain the algebraic definition quite well, so here's what I would like to add:

> The multiplication in the [algebraic representation](/posts/perceptron/introduction) of a perceptron is the *dot product*. It is defined as the sum of the products of each element:
$$ a \cdot b = \sum_i a_i b_i $$

Now that we've clarified that, here's my question from the last post: "How do we actually obtain a useful weights vector \\(w\\)
and a bias \\(b\\)?"

And a good parallel to look at is how we, humans, learn things. How does a child learn how to speak? It picks up words and even grammar 
from its surroundings. How does a student learn a math concept? It looks at the general rule first and then looks at examples 
(*usually*). The pattern here is that humans learn best by example. You *can*, of course, learn things without examples, and sometimes it
may be more efficient, but for the sake of analogy, I'd like to limit it to learning by example.

Depending on your background as you read this post, you may or may not know that any kind of model requires example data to learn. Modern
LLMs are trained on massive datasets of human speech, to then produce a replica of it. We then verify how well these models learned off
of the training data by giving it some input and checking how well the output matches what we expect. This data is called *test data*. 

As an example, let's find some parameters for a perceptron which emulates the `AND` logic gate. We'll have to define inputs and outputs
first (`0`  is mapped here to `-1`):


| Input A    | Input B    | Output    | Input Vector Repr. |
|------------|------------|-----------|--------------------|
| `-1`       | `-1`       | `-1`      | `{-1; -1}`         |
| `-1`       | `1`        | `-1`      | `{-1; 1}`          |
| `1`        | `-1`       | `-1`      | `{1; -1}`          |
| `1`        | `1`        | `1`       | `{1; 1}`           |

So we have to find a function that:
$$
sgn(f(x)) = \hat{y} \newline 
$$
or 
$$
sgn(w \cdot x + b) = \hat{y}
$$

where \\(\hat{y}\\) is the expected output.

If you want, take a minute to find a set of weights and a bias that fulfills these requirements. 

In our case, these are:
$$
w = \\{1; 1\\} \newline
b = 1
$$

We can check this on the inputs/outputs of the `AND` gate, and we'll find that they do indeed work as expected.

Since we're already modelling logic gates, we can try modelling another gate like `OR`:

| Input A    | Input B    | Output    | Input Vector Repr. |
|------------|------------|-----------|--------------------|
| `-1`       | `-1`       | `-1`      | `{-1; -1}`         |
| `-1`       | `1`        | `1`       | `{-1; 1}`          |
| `1`        | `-1`       | `1`       | `{1; -1}`          |
| `1`        | `1`        | `1`       | `{1; 1}`           |

Again, you can take a moment to find the parameters. There isn't necessarily always one correct solution for modelling perceptrons, by
the way:

$$
w = \\{1; 1\\} \newline
b = 1
$$

And finally, you could try this for the `XOR` gate:

| Input A    | Input B    | Output    | Input Vector Repr. |
|------------|------------|-----------|--------------------|
| `-1`       | `-1`       | `-1`      | `{-1; -1}`         |
| `-1`       | `1`        | `1`       | `{-1; 1}`          |
| `1`        | `-1`       | `1`       | `{1; -1}`          |
| `1`        | `1`        | `-1`      | `{1; 1}`           |

If you try to find some fitting weights and a bias for this gate however, you'll find it's not really possible. There's a reason for 
that, and it's one of the perceptron's limitations. Let's plot the `XOR` and the `AND` gate on a colored plane:

{{<figure src="/posts/xor_gate.png" title="XOR & AND Gate plotted on a 2D-Plane" caption="Green = `1`; Red = `-1`. X-Axis is Input A, Y-Axis is Input B">}}

{{<figure src="/posts/and_gate.png" title="AND">}}

You might notice that you can separate the outputs with a single line for `AND`, but not for `XOR`. This is called linear separability.

### Linear separability in Perceptrons
Linear separability is a property of two sets of points. Simply, this means that there is a line that can separate 
both sets.

-- TODO