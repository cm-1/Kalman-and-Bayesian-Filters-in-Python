First exercise, creating a simple g-h filter:
```python
from kf_book.gh_internal import plot_g_h_results
def g_h_filter(data, x0, dx, g, h, dt):
    """
    Performs g-h filter on 1 state variable with a fixed g and h.

    'data' contains the data to be filtered.
    'x0' is the initial value for our state variable
    'dx' is the initial change rate for our state variable
    'g' is the g-h's g scale factor
    'h' is the g-h's h scale factor
    'dt' is the length of the time step 
    """
    ret_data = np.empty_like(data)
    x_est = x0

    for i, z in enumerate(data):
        pred_x = x_est + dx * dt

        residual = z - pred_x
        x_est = pred_x + g * residual
        ret_data[i] = x_est

        dx += h * residual/dt
    return ret_data
```

Second exercise: generating noisy measurements from "true" state:
```python
def my_noisy_data(start_val, change_per_step, num_steps, noise_amt):
    gt = start_val + change_per_step * np.arange(num_steps)
    return gt + (noise_amt * np.random.randn(num_steps))
```