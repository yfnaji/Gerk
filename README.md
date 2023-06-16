# Generalized Explicit Runge-Kutta (GERK)

A package for the curious mathematicians and engineers who want to experiment the Runge-Kutta method with their own coefficients.

[PyPI link](https://pypi.org/project/gerk/)

[GitHub link](https://github.com/yfnaji/Gerk)

# Contents

1. [Runge-Kutta Overview](#rko)
2. [What is Gerk?](#wig)
3. [How to use Gerk](#h2ug)
    * [Attributes](#attr)
    * [Condition Arguments](#cond-arg)
    * [Methods](#methods)
3. [Example](#example)
    * [Creating the Gerk Object](#cgo)
    * [plot example](#plot-eg)
    * [get_approximations example](#get-approx-eg)
    * [get_errors example](#get-err-eg)
    * [efficiency_graph example](#eff-graph-eg)
5. [Adaptive Runge-Kutta Methods](#ark)
6. [Adaptive Runge-Kutta Example](#ark-eg)

<h1 id="rko">Runge-Kutta Overview</h1>

Runge-Kutta methods aim to numerically solve ordinary differential equations of the form:

$$
\frac{\text{d}y}{\text{d}x}=f(x,y)
$$

with a given initial condition $(x_0, y_0)$. 

We define a member of the Runge-Kutta family with a *Butcher tableau*:
<!-- 
$$
  \begin{array}{c| c c c c}
    0\\
    c_2 & a_{21}\\
    c_3 & a_{31} & a_{32}\\
    \vdots & \vdots &  & \ddots\\
    c_r & a_{r1} & a_{r2} & \cdots & a_{rr-1}\\
    \hline
      & b_1 & b_2 & \cdots & b_{r-1} & b_r
  \end{array}\\ \\
$$ -->

<img width="247" alt="genrk" src="https://github.com/yfnaji/Gerk/assets/59436765/c35b7df0-0080-41bf-8f1d-04a53ad9dac5">

The above tableau is often abbreviated to

<!-- $$
  \begin{array}{c| c}
    c & A\\
    \hline
      & b
  \end{array}\\ \\
$$ -->

<img width="103" alt="abbrk" src="https://github.com/yfnaji/Gerk/assets/59436765/561bd4fb-d488-44cf-9246-56cb392ad66d">

The Butcher tableau presents the categories of coefficients that will be used in our Runge-Kutta method.

The $n^\text{th}$ evaluation of the solution will be denoted as $(x_n, y_n)$. We also define $h$ as the time step i.e. the step size from the previous approximation to the next, and therefore

$$
x_{n+1} = x_{n} + h
$$

Before defining $y_n$, we must familiarize ourselves with the array $k$. We define the $i^{\text{th}}$ row of $k$ at $(x_n, y_n)$ as:

$$
k_i(x_n, y_n) = f\left(x_n + c_i h, y_n + \sum_{j=1}^{i-1} a_{ij}k_j(x_n, y_n)\right)
$$

where $f$ is the function defined in the differential equation above. Note the recursion in the second argument of $f$ where we sum rows preceding $k_{i}$ and apply a scale factor of $a_{ij}$.

We now have everything we need to calculate $y_{n+1}$:

$$
y_{n+1} = y_{n} + h \sum_{i=1}^{r}b_i k_{i}(x_n, y_n)
$$

The idea is to calculate various slopes at point $y_n$ to ascertain a weighted of the ascent (or descent) and add it to the previous approximation.

<h1 id="wig">What is Gerk?</h1>

Most packages for the Runge-Kutta method usually have the coefficients $a_{ij}$, $b_i$ and $c_i$ determined beforehand for known methods such as the *Forward-Euler method*, the *1/4 rule*, *the 3/8 rule* etc, but do not allow one to customerize their own Runge-Kutta.

Gerk is an easy interface to allow the user to determine their own coefficient values for the Runge-Kutta method. The package can return the approximations in the form of an array, a plot of the approximated curve (using matplotlib) and also compare with real values (if available) to produce errors and even an efficiency graph!

<h1 id="h2ug">How to use Gerk</h1>

We can simply import `Gerk` in the following way:

```
from gerk import Gerk
```

<h2 id="attr">Attributes</h2>

As seen in mathematics above, there is quite a bit information required to execute the Runge-Kutta method. This has been broken down into arguments to be passed into the `Gerk` class:

- `A` The $A$ matrix in the Butcher tableau. This **must** be a lower triangular matrix that is formatted as a list of lists that contain floats, decimals or integers **Note**: Fractions must be in a string e.g. "1/3"
- `b` The $b$ array. Must be a list of floats, decimals or integers
- `c` The $c$ array. Must be a list of floats, decimals or integers
- `initial_conditions` A `tuple` that acts as the coordinate of the initial condition values $(x_0, y_0)$
- `final` The value of $x$ for which we terminate the Runge-Kutta method
- `time_steps` The number of times steps you want to apply on the from the starting point $x_0$ to `final`. Must be an integer
- `func` The function expression to be numerically integrated
- `real_values` (Optional) A `callable` which is the explicit solution to the initial value problem _**or**_ a list of real values **Note** the list must be of size `time_steps + 1`


<h2 id="cond-arg">Condition Arguments</h2>

There is no consensus to what conditions must hold regarding the coefficients you choose for your method, however, some known Runge-Kutta methods do consistently conform to some known conditions.

These conditions are

$$\sum_{i=1}^{r}b_i=1 \ \ \ \ \sum_{i=1}^{r}b_ic_i = 1/2 \ \ \ \ \sum_{j=1}^{r}a_{ij} = c_i$$

which can be enforced by the boolean parameters `condition_b`, `condition_bc` and `condition_Ac` respectively. These parameters have been set to `False` by default to allow the user to freely explore and experiment.

<h2 id="methods">Methods</h2>

* `solve` can be run to calculate the approximate value of $y_n$ for each time step
*  `efficiency_graph` plots the efficiency graph of the Runge-Kutta method for the defined system for time steps 100, 1,000, 10,000, 100,000 and 1,000,000

Once `solve` has been run, the following methods will be available to execute:

* `plot` Creates a graph of the approximated curve. If you have set the (optional) `real_values` argument, you can plot both the approximated and exact curves in one plot by setting `with_real=True`. You also have the option to amend `x_label` and `y_label` which are arugments of the method
* `get_approximation` returns the approximated values of $y$ in a list
* `get_errors` returns the error for each approximation $y_n$ in a list (only applicable if `real_values` argument was used)

<h1 id="example">Example</h1>

<h2 id="cgo">Creating the Gerk Object</h2>

In the example below, we will use the 3/8-th rule to approximate the following initial value problem:

$$
\frac{\text{d}y}{\text{d}x} = y
$$

with the initial condition $(0, 1)$ and will be making approximations up to $x=5$ with 1000 time steps.

The 3/8-th rule has the following Butcher tableau:

<!-- $$
  \begin{array}{c| c c c}
    0\\
    1/3 & 1/3\\
    2/3 & -1/3 & 1\\
    1 & 1 & -1 & 1\\
    \hline
      & 1/8 & 3/8 & 3/8 & 1/8
  \end{array}\\ \\
$$ -->

<img width="236" alt="rk_eg" src="https://github.com/yfnaji/Gerk/assets/59436765/0105ab38-e0b3-4b97-b5d7-d3fcc1eddb2b">

The $A$ lower triangular matrix in the Butcher tableau above can be implemented in the following way:

```
A = [ 
        ["1/3"], # Remember, fractions must be passed as strings
        ["-1/3", 1],
        [1, -1, 1]
]
```

Note that if you would rather use "square" matrices, you can also define `A` as:

```
A = [
        [0,0,0,0], 
        ["1/3", 0, 0, 0],
        ["-1/3", 1, 0, 0],
        [1, -1, 1, 0]
]
```

Either version of `A` is acceptable in `Gerk`.

Now for the `b` and `c` vectors:

```
b = ["1/8", "3/8", "3/8", "1/8"]

c = [0, "1/3", "2/3", 1]
```

We now have sufficient information to create `Gerk` object. However, we know the solution to the initial value problem is $y=e^x$. So in this case, we can utilise the `real_values` parameter by setting it to the lambda function:

```
lambda x,y: math.exp(x)
```


Now we are ready to create the `Gerk` object:

```
import math

rk_obj = Gerk(
        A=A, 
        b=b, 
        c=c, 
        initial_conditions=(0, 1), 
        time_steps=1000, 
        final=5,
        func=lambda x, y: y, # Remember, you must define both x and y as variables, even if one is not used
        real_values = lambda y: math.exp(y)
    )
```

We can now run `rk_obj.solve()`.

After running `solve`, 3 methods are now available to us:

<h2 id="plot-eg">plot example</h2>

We can now plot our approximated curve. Note that because we have defined `real_values`, we also have the option to plot the exact solution by setting the `with_real` flag to `True`.

```
rk_obj.plot(with_real=True)
```

<img width="500" alt="rk_example" src=https://user-images.githubusercontent.com/59436765/210466156-f86bfb1e-f088-4791-a16a-01220298e6bd.png>


<h2 id="get-approx-eg">get_approximations example</h2>

The approximated values $y_n$ for each timestep can be obtained by running `get_approximations` which returns a list of the values:

```
rk_obj.get_approximations[2:5]
```

> [Decimal('1.010062750717580079753282335'), Decimal('1.015132034738508612169525261'), Decimal('1.020226760387164317916560525')]

_Note_ that this is a _property_ and not a method, so no need for parentheses.

<h2 id="get-err-eg">get_errors example</h2>

The errors for each time step can also be obtained if the `real_value` parameter has been set. In this example, we set `real_values=lambda x: math.exp(x)` which will be used to calculate the error at each step. 

By simply calling `get_errors`, we obtain all the errors in the form of a list (again, no need for parentheses):

```
rk_obj.get_errors[2:5]
```

> [Decimal('0.00001258363341213062577786389261'), Decimal('0.00001897012278968249653135224731'), Decimal('0.00002542036040854158088425155254')]

<h2 id="eff-graph-eg">Efficiency Graph</h2>

We can create an efficiency graph for the Runge-Kutta method provided that `real_values` has been set to a callable function. This can be done by simply running:

```
rk_obj.efficiency_graph()
```

<img width="500" alt="efficiency_graph" src=https://user-images.githubusercontent.com/59436765/210466181-fbc3a2ea-0fac-44b9-a067-bf01bf72ee8f.png>


_Note:_ You do not need to run `solve()` beforehand to produce efficiency graph.

<h1 id="ark">Adaptive Runge-Kutta Methods</h1>

There is an alternate way to utilise the Runge-Kutta method by employing an additional distinct $b$ array. The Butcher tableau for such methods take the form:


<img width="242" alt="adaptive_butcher" src="https://user-images.githubusercontent.com/59436765/210662922-f5fbf612-a56b-4679-af2c-5eca4956771d.png">


where $b^*_i$ is the additional $b$ array.

This method is not too disimilar to the original Runge-Kutta method. We calculate the $k$'s in the same way as before, but there is an extra step when evaluating $y_{n+1}$. 

Although we do calculate $y_{n+1}$ in same way outlined above, we also calculate $\hat{y}_{n+1}$:

$$
y_{n+1} = y_{n} + h \sum_{i=1}^{r}b_i\cdot k_{i}(x_n, y_n) \ \ \ \ \ \ \ \ \ \hat{y_{n+1}} = y_{n} + h \sum_{i=1}^{r} b_{i}^{\ast}\cdot k_{i}(x_n, y_n)
$$

_Note_ that the calculation for $\hat{y}$ requires the use of $y_n$ and **not** $\hat{y}_n$.

For every time step, we calculate the following error:

$$
E := \left|y_{n+1}-\hat{y}_{n+1}\right|
$$

Now we define the tolerance, $\mathcal{E}$, which will act as the maximum acceptable value for $E$.

If $E<\mathcal{E}$, then we accept the value of $y_{n+1}$ and we increment $x_{n}$ by $h$ and start the process again for $x_{n+1}$ and $y_{n+1}$ as normal. 

However, if $E\geq\mathcal{E}$, we will need to adjust the value of $h$ and redo the calculation with this renewed time step value in an attempt to satisfy the condition $E<\mathcal{E}$. 

The value of $h$ will be adjusted as follows:

$$
h \rightarrow 0.9\cdot h \cdot\sqrt[n]{\frac{h}{\mathcal{E}}}
$$

where $n=\min\left(p,q\right)$, where $p$ and $q$ are the orders $^1$ of $b_i$ and $b^*_i$ respectively.

$^1$ A Runge-Kutta method with matrix $A$ and arrays $b$ and $c$ has order $p$ if

$$
\sum_{i=1}^{p}b_i = 1 \ \ \ \ \sum_{i=1}^{p}b_ic_i = 1/2 \ \ \ \ \sum_{j=1}^{p}a_{ij} = c_i  \ \ \ \ 
$$

These are similar to the conditions defined above with the parameters `condition_b`, `condition_bc` and `condition_Ac` respectively. 

However, it is difficult to calculate the order when these conditions are not met. Therefore, to simplify things, $h$ will be re-calculated in the same way whether these conditions are met or not. In the future, we aim to come up with a solution to reflect a more appropriate solution.

Note that you can impose the same conditions on $b^*_i$ with the parameters `condition_b_star` and `condition_b_star_c` for the first two conditions. Again, these are set to `False` by default.

<h1 id="ark-eg">Adaptive Runge-Kutta example</h1>

In this example, we will try to numerically integrate

$$
\frac{\text{d}y}{\text{d}x}=-2xy
$$

with initial conditions $(-5, 1.3887943865\times 10^{-11})$. Note that the exact solution is $y=e^{-x^2}$.

Here we will use the Fehlberg RK1(2) method which has the following Butcher tableau:

<!-- $$
  \begin{array}{c| c c c}
    0 \\
    1/2 & 1/2\\
    1 & 1/256 & 255/256\\
    \hline
      & 1/512 & 255/256 & 1/512\\
      & 1/512 & 255/256 & 0\\
  \end{array}\\ \\
$$ -->

<img width="252" alt="adaptrk" src="https://github.com/yfnaji/Gerk/assets/59436765/bd1e8482-0153-49b2-809e-5a9424ecccd2">

We require an additional argument for the $b^*$ vector, which in `Gerk` is called `b_star`:

```
A = [
        ["1/2"], # Remember, fractions need to be in strings
        ["1/256", "255/256"]
    ]

b = [
    "1/512", "255/256", "1/512"
]

b_star = [
    "1/256", "255/256", 0
]

c = [
    0, "1/2", 1
]
```

The idea of discretizations is meaningless in this context since the step size is constantly changing. Therefore, the `time_steps` parameter will be used as the *starting* time step, which we will set to 0.01 in this example.

We will set the tolerance to a punitive value of 0.00000001. _Note: if `tolerance` is not set, a default value of 0.001 will be used instead.

We can now create our `Gerk` object, `solve` and `plot`:

```
adj_rk = Gerk(
    A=A, 
    b=b, 
    c=c, 
    initial_conditions=(-5.0, 1.3887943865e-11),
    time_steps=0.001, # Remeber, this needs to be the step size NOT the number of discretizations
    final=5,
    func=lambda x,y: -2*x*y,
    real_values=lambda x: math.exp(-x**2),
    b_star=b_star,
    tolerance=0.00001
)

adj_rk.solve()

adj_rk.plot(with_real=True)
```

<img width="500" alt="rk_adaptive" src="https://user-images.githubusercontent.com/59436765/210466230-cbee11e0-02e5-49ab-92f1-553e302a972f.png">

In this case, the Runge-Kutta approximation is so accurate that we can barely see the exact curve!

Efficiency graphs are not available for the adaptive Runge-Kutta method as the number discretizations is not constant.

<h1>Additional Notes</h1>

* I have taken some inspiration from [this paper](https://www.mdpi.com/2297-8747/21/4/46) by M. A. Demba, N.Senu and F. Ismail which helped in understanding the adaptive RK method and from [Jim Verner's](https://www.sfu.ca/~jverner/) extensive research in exploring Runge-Kutta possibilities

* Future works will include
   1. Accept `numpy.array` as parameter input
   2. Extend and generalize the adaptive Runge-Kutta Method, an idea inspired by [M. A. Demba, N.Senu and F. Ismail](https://www.mdpi.com/2297-8747/21/4/46)

* Find solution to finding RK orders when conditions are not met
