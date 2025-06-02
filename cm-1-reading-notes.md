# Notes

This set of notes is the same type of notes I'd be taking during a course.
It's a *summary* of what I learned and a *highlight* of key points/terms.

However, if I encounter a part of the book that I don't find 100% clear, and I
gain some *insight* in how to interpret it (or something similarly "novel"),
then I think it might make more sense for me to note it at that location in the
respective .ipynb file itself, so that I don't have to dig for such notes in the
future if I'm rereading the book itself instead of my notes.

# 01-g-h-filter

## Terms:

Terms related to **g-h filter** components and operation that also apply to
**Kalman filters**:

- **residual**: measurement minus prediction
- **state _estimate_**: the "**state**" itself is the true system configuration, but 
  we (probably) will never "know" it definitively; we just can calculate
  "**estimates**" of it.
- **system**: "the object that we want to estimate"
  - Some sources say "**plant**", and any term below that has "system" in it could
    also have "plant" instead.
- **process model** or **system model** 
  - The **system error** or **process error** is the error in this model.
- **system propagation**: step where process model creates new state prediction.
- **measurement update**: step where measurement is used to update state estimate.
- **epoch**: one iteration of _system propagation_ + _measurement update_.
  - A.k.a. an **evolution**, a.k.a. a **state/system/time evolution**.


In terms of error, we have:

- **random error**: the unpredictable noise
- **systemic (systematic?) error**, or **lag error**: error with a nonzero mean,
  e.g., the error introduced by assuming constant-velocity when in fact there is
  constant acceleration.
  - I've also seen this given names that include the word "**bias**".
  - Merriam-Webster says, "an error that is not determined by chance but is
    introduced by an inaccuracy (as of observation or measurement) inherent in
    the system." This seems to align with what other stats-oriented sources say
    less directly, and how it's separate from "_random error_".

Somewhat related to error:

- **ringing**: when the state estimate sort of bounces around in a sinusoidal way.

For a **g-h filter**, we have:

- **g**: the scaling used for _measurement_
  - A higher "g" means prioritizing the measurement over the prediction.
- **h**: the scaling used for _change in measurement over time_
  - Likewise, a higher "h" means we update faster based on measurements.

## Key Points:

There are a bunch of different filters that can be classified as g-h filters, 
including Kalman filters.

If we have some kind of data, we should not get rid of it, even if it is noisy!
What we do to accomodate noise is just acknowledge how noisy it might be.
At least, that's what the books says. If you take things to extremes, there are
certainly cases where the data is noisy enough that the extra processing FLOPs
would not be "worth it" for some problems. And I think there's some cut-off to
what counts as "data", because if you have a sensor that emits constant/random
measurements regardless of the true state, I think that is clearly useless.

I guess maybe in the case of a sensor emitting a constant/random measurement, if you
looked at the distribution of the "errors" from said measurement, it would have
a variance which somewhat shows the valid range of values the state can take.
But anyway, I'm probably overthinking this for now. It's definitely a good thing
to keep an open mind about and to not hastily throw away data.

_Finally_, when you get into the Bayes theorem math, it is possible that such
a measurement really does have zero affect on the calculations. It's probably
quick to show but I really do have zero time right now....

---

A small value of **g** will be slow to adapt to velocity changes, whereas
a large value will result in more noise (almost just copying measurements),

A small value of **h** will result in larger ringing waves. If it's too large,
then it also gets too influenced by noise (in some ways worse, where it will
"overshoot" the noise!!!).

## Elaborations:

$$
\begin{align*}
\dot{x}_{k+1} &= (1-h)\dot{x}_k + h\frac{z - x_{k}}{dt}\\
    &= \dot{x}_k + h\frac{z - (x_k + dt\cdot \dot{x}_k)}{dt}\\
    &= \dot{x}_k + h\frac{z - (\text{latest prediction})}{dt}
\end{align*}
$$

# 02-Discrete-Bayes

## Terms:

- **prior**: the probability before incorporating measurement or other info.
- **belief**: a measure of the strength of our knowledge about something.
    - This can be at any stage of our operations. After we measure, we update 
      our belief. Then we update our belief again with the process model.
      Etc....
- **multimodal**: where the probability distribution or pdf has multiple
  "modes", i.e. maximums i.e. peaks.
- **likelihood**: a function from a model parameter to the probability of, given
  said model, seeing the data we saw. If our model parameter is named $\theta$,
  it is a function $L: \theta \to [0,1]$. The function notation may be either
  $L_x(\theta)$ or $L(\theta|x)$, where $x$ is the observed data.
    - For discrete probabilities, the function is $\theta \to P_\theta(X = x)$
      for data $x$ with "parameter" $\theta$. Alternatively, in contexts such as
      this chapter, it can be thought of as $P(X = x | \theta)$ (more elaboration below.)
    - For continuous probabilities, the function is $\theta->f(x|\theta)$ for
      probability density function $f$.
    - Above, it would be more "common" to use `\pazocal{L}` in place of $L$, but
      I used $L$ because it seems that certain markdown renderers don't support
      `\pazocal` (e.g., VS Code's) and to make the markdown source more readable.

Frequentist vs. Bayesian statistics:

- **frequentist statistics**: based on frequency with which events occur.
- **bayesian statistics**: takes past information into account.

## Key Points:
Prior is P(A), Posterior is P(A|Data)

**Bayesian filters** are a family of filters that Kalman Filters belong to.

We have a new _prior_ after each "Predict" stage in the Kalman filter, and we
have a new _posterior_ after each "Update"/"Measure" stage.

We use **likelihood** of measurement-given-model to update our **belief**.
We also use our process model to update our **belief**.

## Elaborations:

Below, I use `!` for "NOT" (i.e., negation) and `^` or `,` for "AND".

I'll first write out my train of thought "thinking ~~out loud~~ in markdown" 
that I did to reach my understanding of the likelihood/normalization math, in 
case the thought process is helpful (either for future me or someone else).
After that I'll give a more general subsection that summarizes these thoughts
and also quickly covers the system propagation step and other thoughts.

Some of this is also covered by the book author in later chapters, which I did
not realize until typing this out, but now there's no point in deleting it.

### The Math for the Likelihood and Normalization Calculations
From Baye's Theorem:
```
P(door|says_door) = P(door ^ says_door) / P(says_door)
```
Here, `P(door|says_door)` denotes the probability that the dog is _actually_ in 
front of the door, given that the sensor `says_door`. 

Meanwhile, from the Law of Total Probability:
```
P(says_door) = P(door ^ says_door) + P(!door ^ says_door)
```

By thinking about the possible "tree" of events and then plugging in the numbers
from the textbook's example:
```
P(door ^ says_door) = (3/10) * (3/4) = 9/40
P(!door ^ says_door) = (7/10) * (1/4) = 7/40
```

From all of the above, we can conclude:
```
P(says_door) = 16/40
P(door|says_door) = 9/16
p(door1|says_door) = 1/3 * P(door|says_door) = 3/16 = 0.1875
```
where "door1" represents the case of being in front of Door #1 in particular.

This is basically what the textbook does, except it is basically ignoring the
common denominators and letting the normalization take care of that.
```
P(door1|says_door) = P(door1 ^ says_door) / P(says_door) <-- common denominator for all P(...|says_door)

P(door1 ^ says_door) = (3/4) * 1/10
P(wallx ^ says_door) = (1/4) * 1/10 <-- 4, and thus 4*10=40 also "part of" common denominator.

P(door1 ^ says_door) = 3 * P(wallx ^ says_door)
P(door1 | says_door) = 3 * P(wallx | says_door)
```
Above, "wallx" refers to being in front of Wall #X for some X=0,1,2,....


### The Normalization Itself

The bit about:
```
posterior = likelihood*prior/normalization
```
is really saying:
```
P(A|B) = P(B|A)*P(A)/P(B),     e.g.
P(door1|says_door) = P(says_door|door1)*P(door1)/P(says_door)
```
For any **sensor reading** B, the P(B) is a common denominator for P(A|B) for all
choices of state estimate A, meaning the **normalization** does not need to
explicitly calculate P(B) using something like the Law of Total Probability.

However, the Law of Total Probability _is_ required for the
**system propagation** step, because (letting `A_{n,k}` represent the state
`A_n` holding at timestep `k`):
```
P(A_{j,k+1}) = P(A_{j,k+1} ^ A_{1,k}) + P(A_{j,k+1} ^ A_{2,k}) + ...
     = \sum_{i=1} P(A_{j,k+1}|A_{i,k})P(A_{i,k})
```
which is where the **convolution** comes into play.

### "Chaining" Conditionals

Now, the above all works for the very **first timestep**... but all of that
`P(state|measurement)` conditionality does not just disappear once you finish
the first epoch.... At **later timesteps**, you're really calculating
`P(state|measurement_1, measurement_2, ...)`. How can we prove that the math at
later steps is still valid, then?

If we assume a certain degree of "independence" between results, there is a way
to "simplify" our conditionals. 

Let S1A represent being in state A at timestep 1, let S2B represent being in
state B at timestep 2, and let M1 and M2 be a sensor measurements at timesteps
1 and 2.

If measurement noise is truly random, then P(S2B|S1A) and P(M1|S1A) are
independent, **even though P(S2B) and P(M1) are not**. 

We can then use the following facts:
```
If:        P(A^B|C) = P(A|C)P(B|C) (i.e., independence)
Then: (I)  P(A^B^C) = P(A|C)P(B^C)
      (II) P(A|B^C) = P(A|C)
```
You get (I) by multiplying each side of the hypothesis by P(C) and you get (II)
by dividing (I) by P(B^C).

Then from (II) and P(S2B|S1A) and P(M1|S1A) being independent, we have:
```
P(S2B^M1|S1A) = P(S2B|S1A)P(M1|S1A) <=> P(S2B|M1^S1A) = P(S2B|S1A)
```
meaning we don't need to worry about the last measurement during the system
propagation step.

Similarly, we can show that P(M2|S2B,S1A,M1) = P(M2|S2B), which means that
```
P(S2B|M2,M1,S1A) = P(S2B,M2,M1,S1A)/[P(M2|M1,S1A)P(M1,S1A)]
                 = P(M2|S2B,M1,S1A)P(S2B|M1,S1A)/P(M2|M1,S1A)
                 = P(M2|S2B)P(S2B|S1A)/P(...)
```

The below is a very rough "scratchpad" elaboration on the above that I did for
visualizing what the conditionals look like when you only condition on the
S0 state and the measurements, where the "S1" conditions are summed over.
```
P(S0i)
P(S1j|S0i)
P(S1j|M1,S0i) = P(S1j,S0i,M1)/P(M1,S0i) = P(S1j,S0i,M1)/[P(M1|S0i)P(S0i)]
              = P(M1|S1j,S0j)P(S1j|S0j)/(M1|S0i) = P(M1|S1j)P(S1j|S0i)/P(M1|S0i)

P(S2k|M1,S0i) = \sum_{j=1} P(S2k|S1j,M1,S0i)P(S1j,M1,S0i)/P(M1,S0i)
              = \sum_{j=1} P(S2k|S1j,M1,S0i)P(S1j|M1,S0i) 
              = \sum_{j=1} P(S2k|S1j)P(S1j|M1,S0i)

P(S2k|S0i,M1,M2) = P(S2k,S0i,M1,M2)/P(M2,M1,S0i)
                 = P(S2k,S0i,M1,M2)/P(M2|M1,S0i)P(M1,S0i)
                 = P(M2|S2k,(S0i,M1))P(S2k|M1,S0i)/P(M2|M1,S0i)
                 = P(M2|S2k)P(S2k|M1,S0i)/P(M2|M1,S0i)


P(S3m|S0i,M1,M2) = \sum_{k=1} P(S3m|S2k,M2,M1,S0i)P(S2k,M2,M1,S0i)/P(M2,M1,S0i)
                 = \sum_{k=1} P(S3m|S2k,(M2,M1,S0i))P(S2k|S0i,M1,M2)
...
```

# 03-Gaussians
## Terms:

A **Gaussian function** is a non-normalized Gaussian, while a **Gaussian distribution** is a normalized one.

Terms for shapes of probability distributions:

- **skew**: how asymmetrical a probability distribution is about the mean.
- **kurtosis**: how much of a distribution is concentrated in the tails vs. centre.
  - A normal distribution has an _"excess kurtosis"_ of 0, a uniform has negative excess kurtosis, and a Laplace distribution (whose tails "degrade" slower than a Gaussian) has positive excess kurtosis.
    - The above examples come from [Wikipedia's page for "kurtosis"](https://en.wikipedia.org/wiki/Kurtosis).
## Key Points:

The squaring in the calculation of variance rather than taking, e.g., absolute value, or 4th power, etc. was apparently something that Gauss, the technique's inventor, recognized was somewhat arbitrary.

Probability density represents _relative_ probabilities (e.g., relative amount of cars going a certain speed).

One can compare probability density to physical density, e.g., of a rock or sponge.
A sponge is a good example because density clearly varies across the sponge,
The mass of a region of a sponge is going to be the triple-integration of the density over said 3D region. 

You can also sort of use this as a physical analogy for how, just like how an infinitesimal point of a 3D object has no mass, the probability value for points in continuous distributions is zero...
I do not personally, at least right now, find this analogy better (or worse) than others I've heard for explaining why each point has zero probability,
but it's a new one, so I'm noting it in case it helps me or someone else in the future.

The sum $X + Y$ of two Gaussian **Random Variables** $X$ and $Y$ is also a Gaussian **Random Variable**. 
The product of two Gaussian **Distributions** is a Gaussian **Function**.

For a proof that $X+Y$ is also Gaussian, one can use a convolution of their probability distributions. This makes sense, because, for example, in the discrete case, what is P(X+Y) = 0?
It is P(X=0,Y=0) + P(X=1,Y=-1) + P(X=2,Y=-2) + ...


Motivation for using Gaussians: we can describe a whole, continuous distribution with just two numbers, and likewise, every sum/product with two numbers.

Limitation: if your noise is not a zero-mean Gaussian, then the Kalman filter will perform suboptimally.


