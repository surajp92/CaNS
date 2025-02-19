# about `dns.in`

Consider the following input file as example (corresponds to a turbulent plane channel flow):

~~~
512 256 144              ! itot, jtot, ktot
6. 3. 1.                 ! lx, ly, lz
0.                       ! gr
.95                      ! cfl
1. 1. 5640.              ! uref, lref, rey
poi                      ! inivel
T                        ! is_wallturb
100000                   ! nstep
F                        ! restart
10 10 100 500 10000 5000 ! icheck, iout0d, iout1d, iout2d, iout3d, isave
P P  P P  D D            ! cbcvel(0:1,1:3,1) [u BC type]
P P  P P  D D            ! cbcvel(0:1,1:3,2) [v BC type]
P P  P P  D D            ! cbcvel(0:1,1:3,3) [w BC type]
P P  P P  N N            ! cbcpre(0:1,1:3  ) [p BC type]
0. 0.  0. 0.  0. 0.      !  bcvel(0:1,1:3,1) [u BC value]
0. 0.  0. 0.  0. 0.      !  bcvel(0:1,1:3,2) [v BC value]
0. 0.  0. 0.  0. 0.      !  bcvel(0:1,1:3,3) [w BC value]
0. 0.  0. 0.  0. 0.      !  bcpre(0:1,1:3  ) [p BC value]
T F F                    ! is_forced(1:3)
1. 0. 0.                 ! velf(1:3)
F F  F F  F F            ! is_outflow(0:1,1:3)
2 2                      ! dims(1:2)
4                        ! numthreadsmax
~~~

---
---

~~~
512 256 144              ! itot, jtot, ktot
6. 3. 1.                 ! lx, ly, lz
0.                       ! gr
~~~

These lines set the computational grid.

`itot, jtot, ktot ` and `lx, ly, lz` are the **number of points**  and **domain length** in each direction.

`gr` is a **grid stretching parameter** that tweaks the non-uniform grid in the third direction; zero `gr` implies no stretching. See `initgrid.f90` for more details.

---

~~~
.95                      ! cfl
~~~

This line controls the simulation time step.

The time step is set to be equal to `cfl` **times the maximum allowable time step**.

---

~~~
1. 1. 5640.              ! uref, lref, rey
~~~

This line defines the flow governing parameters.

`uref`, `lref` and `rey` are a reference **velocity scale**, **length scale**, and **Reynolds number** defining the problem. The fluid kinematic viscosity is computed form these quantities.

---

~~~
poi                      ! inivel
T                        ! is_wallturb
~~~

These lines set the initial velocity field.

`initvel` **chooses the initial velocity field**. The following options are available:

* `zer`: zero velocity field
* `cou`: plane Couette flow profile with symmetric wall velocities equal to `uref/2`; streamwise direction in `x`
* `poi`: plane Poiseuille flow profile with mean velocity `uref`                    ; streamwise direction in `x`
* `log`: logarithmic profile with mean velocity `uref`                              ; streamwise direction in `x`
* `hcp`: half channel with plane poiseuille profile and mean velocity `uref`        ; streamwise direction in `x`
* `hcl`: half channel with logarithmic profile and mean velocity `uref`             ; streamwise direction in `x`
* `tgv`: Taylor-Green vortex

`is_wallturb`, if true, **superiposes a high amplitude disturbance on the initial velocity field** that effectively triggers transition to turbulence in a wall-bounded shear flow.

See `initflow.f90` for more details.

---

~~~
P P  P P  D D          ! cbcvel(0:1,1:3,1) [u BC type]
P P  P P  D D          ! cbcvel(0:1,1:3,2) [v BC type]
P P  P P  D D          ! cbcvel(0:1,1:3,3) [w BC type]
P P  P P  N N          ! cbcpre(0:1,1:3  ) [p BC type]
0. 0.  0. 0.  0. 0.    !  bcvel(0:1,1:3,1) [u BC value]
0. 0.  0. 0.  0. 0.    !  bcvel(0:1,1:3,2) [v BC value]
0. 0.  0. 0.  0. 0.    !  bcvel(0:1,1:3,3) [w BC value]
0. 0.  0. 0.  0. 0.    !  bcpre(0:1,1:3  ) [p BC value]
~~~

These lines set the boundary conditions (BC). 

The **type** (BC) for each field variable are set by a row of six characters, `X0 X1  Y0 Y1  Z0 Z1` where,

* `X0` `X1` set the type of BC the field variable for the **lower** and **upper** boundaries in `x`;
* `Y0` `Y1` set the type of BC the field variable for the **lower** and **upper** boundaries in `y`;
* `Z0` `Z1` set the type of BC the field variable for the **lower** and **upper** boundaries in `z`.

The four rows correspond to the three velocity components, and pressure, i.e. `u`, `v`, `w`, and `p`.

The following options are available:

* `P` periodic;
* `D` Dirichlet;
* `N` Neumann.

The **last four rows** follow the same logic, but now for the BC **values** (dummy for a periodic direction).

---

~~~
T F F                    ! is_forced(1:3)
1. 0. 0.                 ! velf(1:3)
F F  F F  F F            ! is_outflow(0:1,1:3)
~~~
These lines set the flow forcing and outflow regions. 

`is_forced`, if true in the direction in question, **forces the flow** with a pressure gradient that balances the total wall shear (e.g. for a pressure-driven channel). The three boolean values corresponds to three domain directions.

`velf`, is the **target bulk velocity** in the direction in question (where `is_forced` is true). The three values correspond to three domain directions.

`is_outflow`, if true, **prescribes an outflow condition** for the boundary-normal velocity, determined from the condition of zero divergence. The six boolean values denote the upper and lower boundaries of the domain, for each direction.

---

~~~
10000                    ! nstep
F                        ! restart
~~~
These lines set total number of time steps and wether the simulation should be restarted from a checkpoint file.

`nstep` is the **total number of time steps**.

`restart`, if true, **restarts the simulation** from a previously saved checkpoint file, named `fld.bin`.

---

~~~
10 10 20 5000 10000 2000 ! icheck, iout0d, iout1d, iout2d, iout3d, isave
~~~

These lines set the frequency of time step checking and output:

* every `icheck` time steps **the new time step size** is computed according to the new stability criterion and cfl (above);
* every `iout0d` time steps **history files with global scalar variables** are appended; currently the forcing pressure gradient and time step history are reported;
* every `iout1d` time steps **1d profiles** are written (e.g. velocity and its moments) to a file;
* every `iout2d` time steps **2d slices of a 3d scalar field** are written to a file;
* every `iout3d` time steps **3d scalar fields** are written to a file;
* every `isave`  time steps a **checkpoint file** is written (`fld.bin`) overwritting the last save.

1d, 2d and 3d outputs can be tweaked modifying files `out?d.h90`, and re-compiling the source. See also `output.f90` for more details.

---

~~~
2 2                      ! dims(1:2)
4                        ! numthreadsmax
~~~

These lines set the grid of computational subdomains and maximum number of threads.

`dims` is the **number of computational** subdomains in `x` and `y`.

`numthreadsmax ` is the **maximum number OpenMP threads**.
