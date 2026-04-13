# About CAsim
CAsim is a DSL for cellular automaton simulation. It provides a simple and expressive way to define them
and observe their evolution with a friendly and easy to use graphic interface.


# Installation guide

The program requires the OpenGL and GLUT c libraries. In Debian-based distros, they can be installed with:

`sudo apt install freeglut3-dev libgl1-mesa-dev`

Stack is also needed. Check its web page to install it: https://docs.haskellstack.org/en/stable/

To install CAsim, you can clone the repository:

`git clone git@github.com:joacozilli/CAsim.git`


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


# Cellular Automaton Definition

CAsim reads the cellular automaton definition from the provided file, which must be a text file. The syntaxis is described
as follows:

```
Automaton NAME {

    States := STATE_1 : COLOR_1 | ... | STATE_N : COLOR_N

    Neighborhood := (x_1,y_1) | ... | (x_m,y_m)

    Transition {
        TRANSITION_RULE
    }

    Default := DEFAULT_STATE
}
```
Where:
- `NAME` is the name of your cellular automaton.
- each `STATE_i` is the name of a possible state a cell can be, with `COLOR_i` the color it will be represented with in the simulation.
All the posible colors are: white, black, gray, red, lightred, blue, lighblue, yellow, darkyellow, green, darkgreen,
cyan, magenta, azure, orange, rose and violet.
- each `(x_i,y_i)` in `Neighborhood` is a pair of integers such that, for any cell `(a,b)`, `(a + x_i, b + y_i)` is a neighbor of `(a,b)`
- `TRANSITION_RULE` is, as the name indicates, the transition rule (or local rule) of the cellular automaton. More information below.
- `DEFAULT_STATE` is the default state of all the cells a the beginning. It must be a defined state in `States`.

The `examples` directory provides several examples of already defined cellular automatons.

# Transition Rule

The transition/local rule is what makes a cellular automaton interesting. It is a function that is applied to all cells in every instant,
which results in a new configuration. It takes the states of the cell and its neighbors implicitly as arguments and calculates the state of the
cell in the next instant.

The concrete syntax of the transition rule is given below:

```
RULE ::= STATEEXP
        | ’if’ BOOLEXP ’then’ RULE ’else’ RULE
        | ’case’ ’{’ COND’}’
        | ’let’ IDENT ’=’ INTEXP ’in’ RULE

COND ::= ’otherwise’ ’:’ ’{’ RULE ’}’
        | BOOLEXP ’:’ ’{’ RULE ’}’ COND

STATEEXP ::= ’#’ IDENT | ’cell’ | ’nei’ ’(’ NAT ’)’

INTEXP ::= INT | ’neighbors’ ’(’ STATEEXP ’)’ | ’-’ INTEXP | IDENT

BOOLEXP ::= ’False’ | ’True’
        | BOOLEXP ’and’ BOOLEXP
        | BOOLEXP ’or’ BOOLEXP
        | ’not’ BOOLEXP
        | INTEXP ’==’ INTEXP
        | INTEXP ’<=’ INTEXP
        | INTEXP ’<’ INTEXP
        | INTEXP ’>=’ INTEXP
        | INTEXP ’>’ INTEXP
        | INTEXP ’!=’ INTEXP
        | STATEEXP ’==’ STATEEXP
        | STATEEXP ’!=’ STATEEXP
        | STATEEXP ’in’ ’[’ SET ’]’
```