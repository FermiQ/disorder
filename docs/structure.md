# `structure` Module

## Overview

The `structure` module, located in `src/structure.f90`, is responsible for handling the input and output of crystal structure files, specifically in the VASP `POSCAR` format. It provides subroutines to read a `POSCAR` file into memory (`readpos`) and to write structural data from memory into a `POSCAR` file (`writepos`).

## Subroutines

*   **`readpos(a, x, symbol, natom, poscar)`**:
    *   **Description**: Reads a crystal structure from a VASP `POSCAR` formatted file. It parses lattice vectors, atomic symbols, atom counts, and atomic positions. It handles both Cartesian and Direct (fractional) coordinates, converting Cartesian to Direct if necessary.
    *   **Arguments**:
        *   `a` (Real(8), Dimension(3,3), Intent(out)): The lattice vectors. `a(:,i)` is the i-th lattice vector.
        *   `x` (Real(8), Allocatable, Dimension(:,:), Intent(out)): Atomic coordinates in Direct (fractional) units. `x(:,i)` contains the coordinates for the i-th atom.
        *   `symbol` (Character(len=2), Allocatable, Array, Intent(out)): Atomic symbols for each atom type.
        *   `natom` (Integer(2), Allocatable, Array, Intent(out)): Number of atoms for each corresponding atomic symbol in `symbol`.
        *   `poscar` (Character(len=*), Intent(in)): The filename of the `POSCAR` file to read (e.g., "SPOSCAR").
    *   **Logic**:
        1.  Opens the specified `poscar` file.
        2.  Reads the comment line (and discards it).
        3.  Reads the universal scaling factor (`scal`).
        4.  Reads the three lattice vectors and scales them by `scal`.
        5.  Determines the number of atomic species (`ntype`) and reads their symbols (`symbol`) and counts (`natom`). This part involves a loop with `backspace` to correctly read the atom symbols and counts, as VASP `POSCAR` format can have symbols on one line and counts on the next, or sometimes on the same line if an older format is used (though this code seems to expect them on separate lines primarily for symbols).
        6.  Calculates the total number of atoms (`satom`).
        7.  Allocates the `x` array for atomic positions.
        8.  Reads the coordinate type line. It checks if it's 'Selective dynamics' (starts with 'S'), and if so, reads the next line for the actual coordinate type ('Direct' or 'Cartesian').
        9.  Reads all atomic positions into `x`.
        10. Closes the file.
        11. If the coordinates were 'Cartesian' (or 'C'), it converts them to Direct (fractional) coordinates by multiplying with the inverse of the lattice matrix `a`. `matmul(inverse(a),x)`. The `inverse` function is expected to be available from the `functions` module (though not explicitly imported with `only`).

*   **`writepos(a, x, symbol, natom, poscar)`**:
    *   **Description**: Writes a crystal structure to a VASP `POSCAR` formatted file. Atomic coordinates are written in Direct (fractional) units.
    *   **Arguments**:
        *   `a` (Real(8), Dimension(3,3), Intent(in)): The lattice vectors.
        *   `x` (Real(8), Dimension(:,:), Intent(in)): Atomic coordinates in Direct (fractional) units.
        *   `symbol` (Character(len=2), Array, Intent(in)): Atomic symbols for each atom type.
        *   `natom` (Integer(2), Array, Intent(in)): Number of atoms for each corresponding atomic symbol.
        *   `poscar` (Character(len=*), Intent(in)): The filename for the output `POSCAR` file.
    *   **Logic**:
        1.  Opens the specified `poscar` file for writing.
        2.  Writes the filename itself as the first comment line.
        3.  Writes a universal scaling factor of "1.0".
        4.  Writes the three lattice vectors, formatted to high precision.
        5.  Writes the atomic symbols on one line.
        6.  Writes the number of atoms for each species on the next line.
        7.  Writes "Direct" to indicate fractional coordinates.
        8.  Writes the atomic coordinates for all atoms, formatted to high precision.
        9.  Closes the file.

## Dependencies

*   **`functions` module**:
    *   The `readpos` subroutine implicitly depends on an `inverse` function (likely `functions.inverse`) for converting Cartesian coordinates to Direct coordinates. This function is not explicitly imported via an `only` clause but is expected to be available.

## Usage Context

This module is fundamental for the `disorder` program:

*   `readpos` is used at the beginning of the `disorder` program to read the initial supercell structure from the `SPOSCAR` input file.
*   `writepos` is used by the `outfiles` module (specifically `outposcar` subroutine) to generate individual `POSCAR` files for each unique atomic configuration found, if requested by the user (`lpos = .true.`). It's also used by the `groups` module to write `SPOSCAR_NEW` if atom reordering occurs.

The module ensures that the `disorder` program can interface with standard VASP structure files for both input and output, making it compatible with other tools in the VASP ecosystem.
