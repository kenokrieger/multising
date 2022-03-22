# <img src="https://kenokrieger.com/wp-content/uploads/2022/03/multising_logo.png" height="100" alt="logo">

<img src="https://kenokrieger.com/wp-content/uploads/2022/03/license-1.png" alt="MIT License">

Multising is an executable to simulate the Ising market model for large lattices
on a CPU or GPU. It was implemented in Python, C and CUDA C. All implementations
can be used with an identical configuration file (see [Usage](#Usage) below).
Details about the theoretical background and implementation can be found below.

Link to the respective repositories:

- [Python](https://github.com/kenokrieger/pymarket)

- [C/C++](https://github.com/kenokrieger/cising)

- [CUDA C](https://github.com/kenokrieger/multispin)

## Theoretical Background

This particular model attempts to predict the behaviour of traders in a market
governed by two simple guidelines:

- Do as neighbours do

- Do what the minority does

mathematically speaking is each trader represented by a spin on a three dimensional
grid. The local field of each spin *S*<sub>i</sub> is given by the equation below

<img src="https://kenokrieger.com/wp-content/uploads/2022/03/local_field.png" alt="field equation" height="100">

where *J*<sub>ij</sub> = *j* for the nearest neighbors and 0 otherwise. The spins
are updated according to a Heatbath dynamic which reads as follows

<img src="https://kenokrieger.com/wp-content/uploads/2022/03/spin_updates.png" alt="Heatbath equation" height="100">


The model is thus controlled by the three parameters

- *alpha*, which represents the tendency of the traders to be in the minority

- *j*, which affects how likely it is for a trader to pick up the strategy of
its neighbour

- *beta*, which controls the randomness

For *alpha* = 0 the model is equivalent to the ising model.

(For more details see <a href="https://arxiv.org/pdf/cond-mat/0105224.pdf">
S.Bornholdt, "Expectation bubbles in a spin model of markets: Intermittency from
frustration across scales, 2001"</a>)

## Implementation

All implementations share the precomputation of probabilities and the Checkerboard
algorithm. The multispin coding is only used in the optimised CUDA C GPU
implementation.

### Checkerboard Algorithm

The main idea behind the checkerboard algorithm is to separate an array containing
spins on a lattice in two arrays each containing the nearest neighbours of the
other. You can think of these lattice entries as tiles on a chessboard. One
array contains the white tiles and the other the black tiles (cf. figure 1 below).
<img src="https://kenokrieger.com/wp-content/uploads/2022/03/metropolis3d.png" alt="3d metropolis algorithm" height="350"></br>
**Figure 1: Checkerboard Algorithm in 3 dimensions**

The numbers in the figure represent the indices of each array element in a 3
dimensional flattened array.

## Precomputation

Looking at the equation from the theoretical background one can see, that for
each iteration there only exists a finite number of possible values for the
probability *p* which does not change during the update. In 2 dimensions, for
example, there exist only 10 possible values (spin sums -4, -2, 0, 2, 4 combined
with spin orientations +1 or -1). These probabilities can be precomputed to
drastically reduce the computation effort for each individual spin update.

### Multispin Coding

Multispin coding stores spin values in individual bits rather than full bytes
leading to more efficient memory usage and thus faster computation times.
The spin values are mapped from (-1, 1) to the binary tuple (0, 1). This
allows for each spin to be resembled by an individual bit. An unsigned long
long (with size of 64 bits), for example, can store 16 spins. The remaining
bits are left untouched to enable a fast computation of the nearest neighbours
sum. Technically, only 3 bits are needed to store values up to 7, but that would
lead to more complicated edge cases.

#### Original Idea

Originally multispin coding was proposed for a 1 dimensional infection chain
model where one person catches an infect if one of their neighbours is infected.
A very efficient way to tackle this problem is to store the spins in individual
bits in three separate arrays containing the "*left*", "*right*" and "*center*"
spins. One bitwise or is then sufficient to update for example the center
arrays (cf. figure 2 below).

<img src="https://kenokrieger.com/wp-content/uploads/2022/03/multispin1d.png" alt="3d metropolis algorithm" height=300></br>
**Figure 2: One dimensional infection chain with multispin**

#### Multispin Neighbour Sum

To compute the sum of the four nearest neighbours on a lattice, the neighbours
of the compacted spins can be used. However one entry needs to be altered
slightly depending on colour and row parity. By summing over the neighbours of
the compacted array, all four original nearest neighbour sums are basically
computed in parallel. The neighbours in the front and back lattices do not need
to be altered and can simply be added to the sum. The main concept is visualised
in figure 3

<img src="https://kenokrieger.com/wp-content/uploads/2022/03/multispin_sum.png" alt="3d metropolis algorithm" height="350"></br>
**Figure 3: Computing the neighbour sum in the multispin coding approach**

## Usage

The program expects a file "multising.conf" in the directory it is called from.
This file contains all the values for the simulation like grid size and parameters.
The path to the file can also be passed as the first argument in the terminal.

Example configuration file:

```
grid_height = 512
grid_width = 512
grid_depth = 512
total_updates = 100000
seed = 2458242462
alpha = 15.0
j = 1.0
beta = 0.6
init_up = 0.5
```

The parameter **grid_depth** is only needed for 3 dimensional lattices.
For **alpha = 0** this model resembles the standard Ising model.

## Links

- [Paper](https://arxiv.org/pdf/cond-mat/0105224.pdf)

- [Thesis](https://kenokrieger.com/wp-content/uploads/2022/03/Thesis_Keno_Krieger.pdf)
