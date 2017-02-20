# Brief Guide to Python Examples
This subdirectory contains Python versions of some of the example programs.
Python has some advantages over Fortran:
it is an interpreted language, rather than a compiled one,
which allows faster code development,
and it has a natural (and readable) programming style.
Variables may be created and redefined at runtime, as required.
Object-oriented programming fits very well with the Python design,
but it is well suited to other styles as well.
It is widely used as a vehicle to introduce students to scientific programming.
For an excellent introductory text, see

* _Learning Scientific Programming with Python,_ C Hill (Cambridge University Press, 2015).

Python has one major drawback, when compared with Fortran and other compiled languages:
it is very slow in execution.
To partially counter this,
the [NumPy](http://www.numpy.org/ "NumPy home page") package offers more efficient handling of array structures.
The syntax of NumPy is surprisingly close to that of Fortran in some respects,
especially for combining arrays.
On top of this, NumPy and the [SciPy](https://www.scipy.org/ "SciPy home page") package
provide a huge variety of scientific libraries,
and our Python examples make extensive use of these.
Nonetheless,
any code which cannot be handled in vectorized form by NumPy will still run slowly.
Several strategies are available to address these issues
(for example, [Cython](http://cython.org/ "Cython home page"),
[SWIG](http://www.swig.org/ "SWIG home page"),
and F2PY which is part of NumPy),
but we do not attempt to follow these here.
As always,
the main aim of these examples is to illustrate ideas in the text,
not to provide programs for practical use.
The reader should feel free to experiment with ways to make the programs run faster!

In the past few years, the community has been making the transition from Python 2 to Python 3.
There are some incompatibilities between the two,
and since a choice had to be made,
we have settled on __Python 3__ for these examples.
We indicate this by the string
```
#!/usr/bin/env python3
```
at the top of each source file.
On many systems,
this will allow the files to be run directly by typing their name;
otherwise it will be necessary to type, for example, `python3 t_tensor.py`,
or just `python t_tensor.py`,
depending on your particular installation of Python.
The examples _will not work_ with Python 2!
They have been tested with Python 3.6.0.
For an introduction to the differences between Python 2 and Python 3,
see the [What's New in Python 3.0](https://docs.python.org/3/whatsnew/3.0.html "What's New in Python 3.0") page.
The most obvious changes are

1. `print(a)` is a function; `print a` will return an error.
2. An expression like `1/2` will return a float; if you want a truncated integer, use `1//2`.
3. Various methods, such as `dict.keys()`, return views,
and various functions, such as `zip`, return iterators, instead of lists.

Anyone coming from a Fortran background should note the use of indentation in Python
to indicate the range of conditional constructs and loops;
there is no `end if` statement!
Fortran experts
should also be aware that indices for arrays
(and other entities) follow the C-convention of numbering from 0 upwards.
Arrays cannot have negative indices; rather, the notation `a[-1]` refers to
numbering backwards from the end of the array.
A more subtle point is that, in the slice notation `a[i:j]`,
the upper index `j` is _excluded_, so for example `a[1:3]` consists of the elements
`a[1]` and `a[2]`,
whereas in Fortran the analogous notation `a(i:j)` refers to elements `i` through `j` _inclusive_.
Yet more subtlety lies in the distinction between
an assignment statement which makes a fresh copy of an object,
such as an array or array slice,
and an assignment which merely gives a name to a view of that object.
In the second case,
no new memory locations are used, and
changing the value associated with one of the objects will affect both of them:
they are really the same object.
This is a powerful, and memory-efficient, approach, but can be confusing!

## Data Input
In the Fortran examples we use a `namelist` to input a few parameters from standard input,
but Python does not have this.
Instead,
to provide a keyword based syntax,
we input these values using the widespread [JSON](http://www.json.org/ "JSON home page") format.
Typical input for a very simple example might be
```
{ "nblock":10, "nstep":1000, "dt":0.005 }
```
and the `"key":value` pairs may be set out on different lines if you wish.
The appearance is very similar to a Python dictionary,
and indeed the data is loaded and parsed into a dictionary for further processing.
Note carefully that we use a colon `:` rather than an equals sign to separate the
key from the value,
and that the keys should be enclosed in double quotes `"..."`.
To avoid some fairly basic exceptions later in the program,
we usually type-check the data
(Python enthusiasts may disapprove of this)
so integer values must not have a decimal point,
while floating-point values must have one
(and at least one digit following the point, for example `1.0`, otherwise JSON raises an exception).
String values need to be enclosed, like the keys, in double quotes.
The variables which may be set in this way are typically considered one by one in our programs:
those whose names correspond to keys supplied in the input file are given the input values,
while the others are given values taken from another `defaults` dictionary.

## Lennard-Jones simulation programs
State points for the Lennard-Jones potential,
in its cut (but not shifted) form, denoted (c),
cut-and-shifted form, denoted (cs),
and full form (f),
are discussed in the Fortran examples [GUIDE](../GUIDE.md).
Except where otherwise stated,
we use our default liquid state point (&rho;,_T_)=(0.75,1.0) for testing,
with _N_=256 atoms,
and compare with the same equations of state due to Thol et al.
The Python codes run much more slowly than the Fortran ones,
and so typically our default parameters carry out shorter runs
(e.g. 10 blocks of 1000 steps rather than 10 blocks of 20000 steps).

### Lennard-Jones MD programs
The module `md_lj_module.py` provides a force routine `force` built around the standard double loop,
and a function `force_faster` which uses NumPy library routines to replace the inner loop.
This is tantamount to _vectorizing_ the all-pairs calculation in a very simple way
(and another approach to the same problem appears in `pair_distribution.py`).
Similarly, a faster version of the `hessian` function is also supplied.
By default, the faster version is imported by the main programs tested below;
the import statement is easily changed to use the slower version,
but typically the difference in speed is an order of magnitude.

Source                 | &rho;    | _T_       | _E_ (cs)   | _P_ (cs) | _C_ (cs)  | _E_ (f)    | _P_ (f)  | _C_ (f)  
------                 | -----    | -----     | --------   | -------- | --------- | -------    | -------  | --------
Thol et al (2015) (cs) | 0.75     | 1.00      | -2.9286    | 0.9897   |  2.2787   |            |          |          
Thol et al (2016) (f)  | 0.75     | 1.00      |            |          |           | -3.7212    | 0.3996   | 2.2630  
`md_nve_lj.py`         | 0.75     | 1.001(1)  | -2.9280    | 0.98(1)  |  2.30(4)  | -3.7277    | 0.38(1)  |          
`md_nvt_lj.py`         | 0.75     | 1.00      | -2.95(3)   | 0.95(4)  |  0.6(1)   | -3.75(3)   | 0.35(4)  | 0.6(1)

* The `md_nvt_lj.py` program seems to give very low heat capacity, not clear why???

## Test programs for potentials, forces and torques
Two program files are provided: `test_pot_atom.py` and `test_pot_linear.py`,
for pair potentials between, respectively, atoms and linear molecules.
These load, at runtime, a module containing a function to calculate
the necessary potential, forces and torques.
The aim is to demonstrate the numerical testing of the analytical derivatives
which go into the forces and torques:
small displacements and rotations are applied in order to do this.
The test is performed for a randomly selected configuration.
Some parameters are used to prevent serious overlap,
which might produce numerical overflow,
while keeping the particles close enough together to give non-zero results.
The values of these parameters may be adjusted via the input file in individual cases.
To run the programs without any tweaking,
simply give (through standard input in the usual way)
a record containing the bare minimum information,
namely a string which identifies
the model of interest, for example `{"model":"at"}`.
The supplied examples (with their identifying strings) are, for `test_pot_atom.py`:

* `"at"` `test_pot_at.py` the Axilrod-Teller three-body potential
* `"bend"` `test_pot_bend.py` the angle-bending part of a polymer chain potential
* `"twist"` `test_pot_twist.py` the angle-torsion part of a polymer chain potential

and for `test_pot_linear.py`

* `"dd"` `test_pot_dd.py` the dipole-dipole potential
* `"dq"` `test_pot_dq.py` the dipole-quadrupole and quadrupole-dipole potential
* `"gb"` `test_pot_gb.py` the Gay-Berne potential
* `"qq"` `test_pot_qq.py` the quadrupole-quadrupole potential

## T-tensor program
The program `t_tensor.py` compares the calculation of multipole energies by two methods:
using explicit formulae based on trigonometric functions of the Euler angles,
and via the Cartesian T-tensors.
Two linear molecules are placed in random positions and orientations,
within a specified range of separations,
and some of the contributions to the electrostatic energies and forces are calculated.
The program may be run using an empty record `{}`,
so as to take the program defaults,
or various parameters may be specified.
Several of the tensor manipulations are neatly expressed using NumPy library functions
such as `outer` (outer product) and `einsum` (Einstein summation).

* How easy would it be to add quadrupole-quadrupole energy, quadrupole-dipole forces,
quadrupole-quadrupole forces, and all the torques, calculated both ways??

## Correlation function program
The aim of the program `corfun` is to illustrate the direct method, and the FFT method,
for calculating time correlation functions.
The program is self contained: it generates the time dependent data itself,
using a generalized Langevin equation,
for which the time correlation function is known.
The default parameters produce a damped, oscillatory, correlation function,
but these can be adjusted to give monotonic decay,
or to make the oscillations more prominent.
If the `origin_interval` parameter is left at its default value of 1,
then the direct and FFT methods should agree with each other to within numerical precision.
The efficiency of the direct method may be improved,
by selecting origins less frequently,
and in this case the results obtained by the two methods may differ a little.

In the current implementation,
the direct method makes little or no use of NumPy's efficient array manipulation functions,
and so is very slow.
For this reason,
the default run length is much shorter than for the Fortran example.
An alternative, much faster, direct method is also provided at the end of the program.
This uses the NumPy `correlate` library function.
The selected mode `'full'` corresponds to aligning the data array `v` with itself,
at all possible offsets that result in some overlap of values,
before computing the products (for each offset) and summing.
In this case, the resulting array is symmetric in time (offset),
and only the values from the mid-point onwards are required;
these must be normalized in the standard way
and identical results to the slow direct method (with `origin_interval=1`)
and FFT method,
are obtained.

## Diffusion program
The program `diffusion.py` reads in a sequence of configurations and calculates
the velocity auto correlation function (vacf),
the mean square displacement (msd), and
the cross-correlation between velocity and displacement (rvcf).
Any of these may be used to estimate the diffusion coefficient,
as described in the text.
The output appears in `diffusion.out`
It is instructive to plot all three curves vs time.

The input trajectory is handled in a crude way,
by reading in successive snapshots with filenames `cnf.000`, `cnf.001`, etc.
These might be produced by a molecular dynamics program,
at the end of each block,
choosing to make the blocks fairly small (perhaps 10 steps).
As written, the program will only handle up to `cnf.999`.
Obviously, in a practical application,
a proper trajectory file would be used instead of these separate files.

It is up to the user to provide the time interval between successive configurations.
This will typically be a small multiple of the timestep used in the original simulation.
This value `delta` is only used to calculate the time, in the first column of
the output file.
A default value of 0.05 is provided as a place-holder, but
the user really should specify a physically meaningful value;
forgetting to do so could cause confusion when one attempts
to quantify the results.

To make it easier to test this program,
we have also supplied a self-contained program `diffusion_test.py`,
which generates an appropriate trajectory by numerically solving
the simple Langevin equation for _N_ non-interacting atoms (_N_=250 by default).
For this model, one specifies the temperature and friction coefficient,
which dictates the rate of exponential decay of the vacf,
and hence the diffusion coefficient.
The exact results for the vacf, rvcf and msd are written out to `diffusion_exact.out`
for easy comparison with `diffusion.out`.
Here are some typical results using default program parameters throughout.
The vacf is in red, rvcf in blue, and msd in green;
every fifth point is shown for the results of `diffusion`,
while the exact results are indicated as lines.
For the default program parameters, the diffusion coefficient is _D_=1.
The results are very similar to those obtained from the analogous Fortran example.

![alt text](diffusion.png "diffusion test results")

## Pair distribution function
The program `pair_distribution.py` reads in a set of configurations and calculates
the pair correlation function _g(r)_.
We limit the number of configurations to a maximum of 1000 (numbered from 000 to 999)
simply so as to use a fixed naming scheme for the input configurations;
in a practical application, a trajectory file would be used instead.
The sum over all pairs is performed using a vectorized approach,
so as to take advantage of NumPy routines.
The coordinate array `r` is compared with a cyclically-shifted copy,
to give `n` (that is, _N_) separation vectors `rij` which are processed all at once.
Only `n//2` shifts are needed to cover every distinct `ij` pair.
There is a slight subtlety on the last shift, if `n` is even:
both `ij` and `ji` pairs appear,
and so the usual incrementing factor 2 is replaced by a factor 1.
The actual histogramming is conveniently performed
by the built-in NumPy `histogram` routine.

We have tested this on a set of 500 configurations
of _N_=256 Lennard-Jones atoms,
cut (but not shifted) at _R_<sub>c</sub>=2.5&sigma;,
at the usual state point &rho;=0.75, _T_=1.0.
The interval between configurations was 100 MC sweeps.
Using the default resolution of 0.02&sigma;,
identical results were obtained as for the Fortran example.

![alt text](gr.png "g(r) test results")

## Error calculation
The program `error_calc.py` is a self-contained illustration of the effects of
correlations on the estimation of errors for a time series.
We produce the series using a generalized Langevin equation,
in the same manner as for the correlation function program (see above).
Since the correlation time of the GLE is exactly known,
we can predict the effects, and compare with the empirical estimates
obtained by different methods.
The program contains extensive comments to explain what is being calculated at each stage.

## FFT program
The aim of `fft3dwrap.py` is to illustrate the way a standard Fast Fourier Transform
library routine is wrapped in a user program.
We numerically transform a 3D Gaussian function,
and compare with the analytically, exactly, known result,
User input defines the number of grid points and the box size;
sensible defaults are provided.
The library that we use for this example is the built-in NumPy one.

## Hit-and-miss and sample-mean
The two programs `hit_and_miss.py` and `sample_mean.py` illustrate two very simple
Monte Carlo methods to estimate the volume of a 3D object.
They are both described in detail at the start of Chapter 4.
No user input is required.
For the built-in values defining the geometry, the exact result is 5/3.

## Quantum simulation programs
The program `qmc_walk_sho.py` solves the diffusion equation in imaginary time
corresponding to the Schrodinger equation,
for a single simple harmonic oscillator.
Atomic units are chosen so that the effective diffusion coefficient is _D_=1/2.
A few hundred independent systems, or walkers, are simulated using a simple random walk
and a crude creation/destruction scheme based on the difference between the potential energy
and the trial energy.
The scheme is described in the text.
The value of the trial energy `et` is updated regularly,
and the hope is that, after convergence,
it will be equal to the correct ground-state energy for the system which, in this case, is 1/2.
The updating scheme, and several of the default parameters,
are taken from the following paper

* I Kostin, B Faber, K Schulten, _Amer J Phys,_ __64,__ 633 (1996).

Reasonable results for the energy and the ground-state wavefunction,
which is accumulated as a histogram of walker positions,
should be obtained using the default input values,
with an empty input record `{}`;
these defaults include setting `et` initially to the exact ground state energy.
Other values such as `{et:0.6}` may be supplied through standard input in the usual way.
This type of simulation is sensitive to the initial value,
and quite noisy:
possible improvements are discussed in general terms in the text.

The program `qmc_pi_sho.py` carries out a path integral Monte Carlo simulation
for a single simple harmonic oscillator,
at a specified temperature and ring-polymer size _P_.
Larger values of _P_ give energies closer to the exact quantum mechanical canonical ensemble average.
For this simple model,
exact results can also be calculated for the finite values of _P_ used in the simulation

* KS Schweizer, RM Stratt, D Chandler, PG Wolynes, _J Chem Phys,_ __75,__ 1347 (1981).
* M Takahashi, M Imada, _J Phys Soc Japan,_ __53,__ 3765 (1984).

and a routine to evaluate these is included in the example.
No special techniques are used to accelerate the simulation;
standard Metropolis moves are employed.
Default parameters correspond to _P_=8, _T_=0.2.
The table below is for test runs at various values of _P_,
keeping the same temperature,
which generates a range of average energies between
the classical limit _E_=0.2
and the quantum limit _E_=0.506784;
in each case we compare with the exactly-known value for the same _P_.

_P_  | _E_ (MC)  | _E_ (exact)
---- | --------  | -----------
 2   | 0.3217(3) | 0.321951
 3   | 0.3920(4) | 0.392308
 4   | 0.4314(5) | 0.431618
 5   | 0.4552(7) | 0.454545
 6   | 0.4686(4) | 0.468708
 7   | 0.4777(6) | 0.477941
 8   | 0.4847(9) | 0.484244