# Notes

This set of notes is the same type of notes I'd be taking during a course.
It's a *summary* of what I learned and a *highlight* of key points/terms.

However, if I encounter a part of the book that I don't find 100% clear, and I
gain some *insight* in how to interpret it (or something similarly "novel"),
then I think it might make more sense for me to note it at that location in the
respective .ipynb file itself, so that I don't have to dig for such notes in the
future if I'm rereading the book itself instead of my notes.

## 01-g-h-filter

### Terms:

- **residual**: measurement minus prediction
- **state _estimate_**: the "**state**" itself is the true system configuration, but 
  we (probably) will never "know" it definitively; we just can calculate
  "**estimates**" of it.
- **system**: "the object that we want to estimate"
  - Some sources say "**plant**", and any term below that has "system" in it could
    also have "plant" instead.
- **process model** or **system model** 
  - The **system error** or **process error** is the error in this model.
- **system propogation**: step where process model creates new state prediction.
- **measurement update**: step where measurement is used to update state estimate.
- **epoch**: one iteration of _system propogation_ + _measurement update_.

For a **g-h filter**, we have:

- **g**: the scaling used for _measurement_
  - A higher "g" means prioritizing the measurement over the prediction.
- **h**: the scaling used for _change in measurement over time_
  - Likewise, a higher "h" means we update faster based on measurements.

### Key Points

There are a bunch of different filters that can be classified as g-h filters, 
including Kalman filters.

If we have some kind of data, we should not get rid of it, even if it is noisy!
What we do to accomodate noise is just acknowledge how noisy it might be.
At least, that's what the books says. If you take things to extremes, there are
certainly cases where the data is noisy enough that the extra processing FLOPs
would not be "worth it" for some problems. And I think there's some cut-off to
what counts as "data", because if you have a sensor that emits constant
measurements regardless of the true state, I think that is clearly useless.

I guess maybe in the case of a sensor emitting a constant measurement, if you
looked at the distribution of the "errors" from said measurement, it would have
a variance which somewhat shows the valid range of values the state can take.
But anyway, I'm probably overthinking this for now. It's definitely a good thing
to keep an open mind about and to not hastily throw away data.

### Elaborations:

$$
\begin{align*}
\dot{x}_{k+1} &= (1-h)\dot{x}_k + h\frac{z - x_{k}}{dt}\\
    &= \dot{x}_k + h\frac{z - (x_k + dt\cdot \dot{x}_k)}{dt}\\
    &= \dot{x}_k + h\frac{z - (\text{latest prediction})}{dt}
\end{align*}
$$
