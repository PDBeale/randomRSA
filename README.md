# Class of scalable parallel and vectorizable pseudorandom number generator based on non-cryptographic RSA exponentiation ciphers

## Computer and Software configuration and Installation instructions
We recommend to use software packages listed below to achieve optimum performance out of our library.
- Linux Operating System
- [Intel C Compiler](https://software.intel.com/en-us/intel-compilers)
	- tested on Intel(R) C++ Intel(R) 64 Compiler for applications running on Intel(R) 64, Version 17.0.4.196 Build 20170411
Copyright (C) 1985-2017 Intel Corporation.
- [Intel MPI Library](https://software.intel.com/en-us/parallel-studio-xe/choose-download/free-trial-cluster-linux-fortran)

## How to use our software
Once, you `git clone` our source code.

Load the following modules
- `module load intel`
- `module load impi`
- `module load intel_cluster_tools`
- `module load hdf5`

After that, you do `make` on our directory, and then you will see `randomRSAlib.a`. In order to run our code, you can relocate `randomRSAlib.a` and copy `randomRSA.h` to your workspace directory.

At your workspace directory

`mpiicc -O2  -qopenmp  <your .c file> randomRSAlib.a -o <name of output of your choice>`

To execute output files

`mpirun -np 8 ./<name of output of your choice>`

## How to use functions in our software : [pdf version available](https://github.com/PDBeale/randomRSA/blob/master/readme.pdf)
The two key public functions available to the user are:

`int vrandomRSA(double * randomRSA, long nrandomRSA);` returns a vector of double precision floating point pseudorandom numbers uniformly distributed on [0, 1). This is our recommended way to use the generator, as it returns large numbers of pseudorandom numbers to the user most efficiently. The user, of course, should use each pseudorandom number once, and only once, to assure all random tests in the user code are uncorrelated. The user defined vector `double * randomRSA` is a double precision array created by the user with `long nrandomRSA` elements.

`double randomRSA();` returns one double precision floating point pseudorandom number per call. The pseudo- random sequence is uniformly distributed on [0, 1). This function is available so the user can easily substitute this RSA-based pseudorandom number generator in place of another generator that returns one pseudorandom number per call.


Both functions will initialize themselves into unique states on the first call in each MPI process using the current time in microseconds since 00:00:00 1/1/1970, and values mpirank and mpisize determined by MPI calls. If the user is not using multiple MPI processes, or MPI has not been initialized before the first call to either generator, the values are set to mpirank=0 and mpisize=1. The user can initialize the generator in the same manner using
`int randomRSA init();`

If the user would like to initialize the generator into a reproducible state using a user chosen seed for debugging purposes, the user can call `int randomRSA init seed(uint64 t seed);` before calls to `randomRSA` or `vrandomRSA`.

`int randomRSA init seed(uint64 t seed);` will initialize each MPI process with the user value `seed`, and the MPI-determined mpirank and mpisize values.

If the user would like to initialize the generator into a reproducible state in each MPI process using a user chosen seed for debugging purposes, the user can use
`int randomRSA init seed MPI(uint64 t seed, int mpirank, int mpisize);` which will initialize each MPI process using seed and user chosen values for `mpirank` and `mpisize`.

If any of the initialization routines are called after the generator has already been initialized, the state of the generator is reinitialized.

The user can access the RSA encryption parameters using the routines
`int randomRSA_get_exponent(uint32 t *exponent);` which returns the RSA exponent e,

`int randomRSA_get_primes(uint32 t *prime1, uint32 t *prime2, uint64 t *composite);` which returns the 32-bit RSA primes p1 and p2, and 64-bit RSA composite n = p1*p2,

`int randomRSA get skipprime(uint64 t *skipprime);` which returns the 64-bit prime q = 2^63 − 25 used in the linear congruential skip generator (q is the largest prime less than 2^63),

`int randomRSA get primitiveroot(uint32 t *primitiveroot);` which returns the 32-bit primitive root mod q used in the linear congruential skip generator.

The user can change the values of the exponent e using
`int randomRSA set exponent(uint32 t exponent);`, where exponent must be an odd integer in the range 3 to 257 (recommended values are e = 3, 5, 9, 17),

The user can change the values of primes p1 and p2 using
`int randomRSA_set_primes(uint32 t prime1, uint32 t prime2);`, where prime1 and prime2 set the values of p1 and p2. The primes p1 and p2 must be safe primes, i.e p1, (p1 − 1)/2, p2, and (p2 − 1)/2 must all be prime. The generator also requires 2^32 > p1 > p2 > 2^31.

The skip prime q, and the primitive root a cannot be changed by the user since they are selected from a very limited set of well-tested values.

The user can get the full internal state of the generator using
`int randomRSA_get_state(uint32 t *prime1, uint32 t *prime2, uint32 t *exponent, uint32 t *primitive- root, uint32 t *index, uint64 t *hash, uint64 t *messages, uint64 t *skips);`.

The user must also declare 64-bit arrays `uint64_t *messages` and `uint64_t *skips` with length `int randomRSA_vectorsize();` before calling `int randomRSA_get_state`.


The user can reset the generator using the saved state by calling
`int randomRSA set state(uint32 t prime1, uint32 t prime2, uint32 t exponent, uint32 t primitive- root, uint32 t index, uint64 t hash, uint64 t *messages, uint64 t *skips);`.

The user must call `int randomRSA` get state before calling `int randomRSA_set_state`, and the user may not make any changes to the stored state before resetting the generator.

This is controlled using a hash function to set and test `hash`. The unsigned 64-bit arrays `messages` and `skips` must be created with length `RSAVECTORSIZE`.

The user can get the total number of pseudorandom numbers returned to the user by calling `long randomRSA_randsreturned();`.

The code can be compiled into an object file using the intel MPI C compiler and the OpenMP library using <br>
`mpiicc -O2 -qopenmp -c randomRSA.c`

The code is vectorized using OpenMP to share the vector calculation across multiple cores on each independent MPI process. The user needs to inform the operating system of the number of cores committed to the calculation before running user code that calls the generator by using the interactive terminal mode, bash shell command <br>
`export OMP NUM THREADS=XX`

When using the code in slurm batch mode, one can set the number of cores assigned to each process using

`#SBATCH –nodes=YY` <br>
`#SBATCH –cpus-per-task=XX`

On the University of Colorado Boulder Summit supercomputer, the code can generate between 100 million and 200 million pseudorandom numbers per second per node when 24 cores were assigned to each node. If a job is limited to a single core, the code generates 10 million to 15 million per second.



## Contributors
Author : Professor Paul Beale, Department of Physics, University of Colorado Boulder </br>
Author : Jetanat Datephanyawat, Department of Physics, University of Colorado Boulder
