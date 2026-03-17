# About CAsim
CAsim is a DSL for cellular automaton simulation. It provides a simple and expressive way to define them
and observe their evolution with a friendly and easy to use graphic interface.


# Installation guide

The program requires the OpenGL and GLUT c libraries. In Debian-based distros, they can be installed with:

`sudo apt install freeglut3-dev libgl1-mesa-dev`

Stack is also needed. Check its web page to install it: https://docs.haskellstack.org/en/stable/


# Use guide

to compile, simply run:

`stack build`

once compiled, to execute the program use:

`stack exec CAsim-exe -- FILEPATH [OPTIONS]`

where FILEPATH is the path to the file where the cellular automaton is defined.

[OPTIONS] is a series of following options:

- -r N the amount of rows of lattice will be N (default value: 100, min: 10, max: 1000)

- -c M the amount of columns of lattice will be M (default value: 100, min: 10, max: 1000)

- -f CHOICE, where CHOICE is toroidal or default, speficies the type of forntier in simulation (default value: default).
In default mode, the neighbors outside the lattice will be assumed of the default state.
In toroidal mode, the lattice is considered a toroid.

- -s K the number of steps per second will be K (default value: 10, min: 1, max: 30)

- -g CHOICE, where CHOICE is seq or par, specifies if global transition function will compute each cell
sequencially or in parallel respectively (default value: par). par option will used all available cores.


Once executed, a graphic window will open to start simulation. To manipulate it, CAsim offers the following inputs:

- Left click on a cell will rotate its state through those defined.

- Space bar pauses/resumes simulation.

- N to go one step forward in simulation. Only posible if simulation is paused.

- R restarts simulation to initial state.

- WASD to move camera.

- C camera goes back to the center.

- mouse wheel/up and down arrows to zoom in and out.

- ESC closes window and finishes execution.

In the examples folder there are several cellular automatons given as examples.