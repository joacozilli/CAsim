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

where `FILEPATH` is the path to the file where the cellular automaton is defined.

[OPTIONS] is a series of following options:

- `-r N` the amount of rows of lattice will be N (default value: 100, min: 10, max: 1000)

- `-c M` the amount of columns of lattice will be M (default value: 100, min: 10, max: 1000)

- `-f CHOICE`, where `CHOICE` is toroidal or default, speficies the type of forntier in simulation (default value: default).
In default mode, the neighbors outside the lattice will be assumed of the default state.
In toroidal mode, the lattice is considered a toroid.

- `-s K` the number of steps per second will be `K` (default value: 10, min: 1, max: 30)

- `-g CHOICE`, where `CHOICE` is `seq` or `par`, specifies if global transition function will compute each cell
sequencially or in parallel respectively (default value: `par`). `par` option will used all available cores.


Once executed, a graphic window will open to start simulation. To manipulate it, CAsim offers the following inputs:

- `Left click` on a cell will rotate its state through those defined.

- `Space bar` pauses/resumes simulation.

- `N` to go one step forward in simulation. Only posible if simulation is paused.

- `R` restarts simulation to initial state.

- `WASD` to move camera.

- `C` camera goes back to the center.

- `mouse wheel/up` and `down/up arrows` to zoom in and out.

- `ESC` closes window and finishes execution.

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
(looking at the grid as cartesian coordinate system).
- `TRANSITION_RULE` is, as the name indicates, the transition rule (or local rule) of the cellular automaton. More information below.
- `DEFAULT_STATE` is the default state of all the cells at the beginning. It must be a defined state in `States`.


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

IDENT := any string valid as an identifier (starts with a letter, has no special characters
and isn't a key word of the language)
```

As you can see, the transition rule has three basic constructors:
- `if then else` for conditional expressions.
- `case` as syntatic sugar for nested `if`'s.
- `let in` to declare integer variables.

Earlier it was said that the local rule takes the states of the cell and its neighbors implicitly as arguments. This
values can be accessed with the key word `cell` (for the cell's state) and `nei(k)` (for the k-th neighbor's state, where
the k-th neighbor is the neighbor number k in the `Neighborhood` list provided in the definition).

To name a defined state in `States` in the rule, use the syntaxis `#STATE_NAME`.

It is provided the built-in function `neighbors(STATE_NAME)` that returns the number of neighbors that are in the
state `STATE_NAME`.

There is also the boolean constructor `STATE in [STATE_1,...,STATE_k]` that evaulates to `True` if `STATE` is in the
set of states described, otherwise evaluates to `False`. This is basically syntactic sugar for
`STATE == STATE_1 or ... or STATE == STATE_k`.

# Example

In this example we will take a look at the classic Game of Life by John Horton Conway. This cellular automaton is very simple:
- There are two states: `alive` and `dead`.
- The neighborhood used is the one known as Moore neighborhood, where the neighbors are the 8 adjacent cells.
- The transition rule is as follows:
    - An alive cell remains alive if exactly 2 or 3 neighbors are alive. Otherwise, it dies by undepopulation or overpopulation.
    - A dead cell becomes alive if exactly 3 neighbors are alive. Otherwise, it remains dead.

```
Automaton GameOfLife {

    States := alive : black | dead : white

    Neighborhood := (1,0) | (0,1) | (-1,0) | (0,-1) | (1,1) | (-1,-1) | (1,-1) | (-1,1)

    Transition {
        let x = neighbors(#alive) in
            if cell == #alive
                then
                    if x == 3 or x == 2 then #alive else #dead
                else
                    if x != 3 then #dead else #alive
    }

    Default := dead
}
```

As you can see, we defined the states `alive` and `dead` and we gave them te colors black and white. The Moore neighborhood is given
as the list of all adjacent cells and all cells are dead by default.

In the transition rule, we declared a variable `x` with the value `neighbors(#alive)`, that is, the amount of alive neighbors. We need
to use this number several times in the transition definition, to it is useful to store it in a variable. After that, it is simply a
matter of checking the conditions of the rule to determine the state of the cell in the next generation.

The `examples` directory provides several examples of already defined cellular automatons (like this one).