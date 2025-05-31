# `stdout` Module

## Overview

The `stdout` module, located in `src/stdout.f90`, is dedicated to managing formatted output to the standard output stream (usually the console) and handling fatal error messages. It provides a centralized way to print banners, informational messages at different stages of the `disorder` program, and a standardized error message format before program termination.

This module relies on the `functions` module, specifically the `width()` function, for dynamically adjusting the formatting of output strings based on the numerical width of integers to be printed.

## Standard Output Subroutines

These subroutines are called at various points in the `disorder` program to inform the user about its progress and key findings.

*   **`stdout_0`**:
    *   **Description**: Prints the initial program banner.
    *   **Output**:
        *   An ASCII art logo for "Disorder".
        *   The current date and time (obtained via `call system('printf "    `date +%F %H:%M:%S`"')`).
        *   The program version (e.g., "0.6.1-VASP").
    *   **Called by**: `disorder` program at the very beginning.

*   **`stdout_1(natom, atom, site, k, symbols)`**:
    *   **Description**: Prints information obtained after reading the `INDSOD` and `SPOSCAR` input files.
    *   **Arguments**:
        *   `natom` (Integer(2), Array): Number of atoms of each type from `SPOSCAR`.
        *   `atom` (Character(len=2), Array): Symbols of atom types from `SPOSCAR`.
        *   `site` (Integer(1)): Index of the atomic site to be substituted.
        *   `k` (Integer(2), Array): Number of atoms for each substituting species from `INDSOD`.
        *   `symbols` (Character(len=2), Array): Symbols of the substituting species from `INDSOD`.
    *   **Output**:
        *   "Reading INDSOD and SPOSCAR ..."
        *   Summary of atom types and total atoms found (e.g., "Found 2 types and 64 atoms ( 32 Si 32 O )").
        *   Identifies the site to be substituted and by which atoms (e.g., "The Si site will be substituted by 2 Cu 30 Si").
    *   **Called by**: `disorder` program after successfully reading and validating inputs.

*   **`stdout_2(no, nr, nt, cls, sys)`**:
    *   **Description**: Prints information related to space group operations found by the symmetry analysis.
    *   **Arguments**:
        *   `no` (Integer(2)): Total number of symmetry operations.
        *   `nr` (Integer(2)): Number of rotation operations.
        *   `nt` (Integer(2)): Number of pure translation operations.
        *   `cls` (Character(len=20)): Point group class (e.g., "Oh", "C4v").
        *   `sys` (Character(len=20)): Crystal system (e.g., "CUBIC", "TETRAGONAL").
    *   **Output**:
        *   "Searching Space Group Operations ..."
        *   Number of operations found (e.g., "Found 48 operations ( 48 rotations and 0 pure translations )").
        *   Crystal system and point group (e.g., "The supercell is an FCC lattice with a point group of Oh").
        *   "Preprocessing Work Related To Combinatorics ..."
    *   **Called by**: `symmetry` module (or a routine that calls symmetry analysis like `eqamatrix` in the main `disorder` program) after space group analysis. *(Correction: While it reports symmetry findings, it's typically called from the main `disorder` program after calling `eqamatrix` and before `irrconfig`)*.

*   **`stdout_3(nc)`**:
    *   **Description**: Prints the total number of atomic configurations found before eliminating duplicates.
    *   **Arguments**:
        *   `nc` (Integer(8)): Total number of configurations.
    *   **Output**:
        *   "Found [nc] atomic configurations"
        *   "Eliminating Duplicate Configurations ..."
    *   **Called by**: `configurations` module, specifically from the `preprocessing` part of `irrconfig`, after calling `confnum`.

*   **`stdout_4(n)`**:
    *   **Description**: Prints the final number of irreducible (unique) configurations found.
    *   **Arguments**:
        *   `n` (Integer(4)): Number of irreducible configurations.
    *   **Output**:
        *   "Found [n] irreducible configurations"
    *   **Called by**: `configurations` module, specifically from the `postprocessing` part of `irrconfig`.

*   **`stdout_5(time)`**:
    *   **Description**: Prints the program termination message, including elapsed time and citation information.
    *   **Arguments**:
        *   `time` (Integer(4)): Total elapsed CPU time in seconds.
    *   **Output**:
        *   "Program finished ! Elapsed time : HH hour MM min SS sec"
        *   A formatted box with citation information for the `Disorder` program/method.
    *   **Called by**: `disorder` program at the very end.

## Error Handling Subroutine

*   **`stderr(message)`**:
    *   **Description**: Handles fatal errors. It prints a custom error message provided by the caller, followed by a standardized error banner, and then terminates the program execution using `stop`.
    *   **Arguments**:
        *   `message` (Character(len=*)): The specific error message to be displayed.
    *   **Output**:
        *   The `message` string.
        *   An ASCII art "ERROR" banner.
    *   **Program Flow**: Terminates the program immediately after printing the messages.
    *   **Called by**: Various modules/subroutines throughout the codebase whenever a fatal error condition is detected (e.g., file not found, invalid input parameter).

## Dependencies

*   **`functions` module**:
    *   Uses the `width()` function to dynamically determine the character width needed for printing integers neatly within formatted strings. This is used in `stdout_1`, `stdout_2`, `stdout_3`, and `stdout_4`.
*   **System Calls**:
    *   `stdout_0` uses `call system('printf "    `date +%F %H:%M:%S`"')` to print the current date and time. This introduces a dependency on the underlying operating system's `date` and `printf` commands.

The `stdout` module ensures consistent and informative output throughout the execution of the `disorder` program, making it easier for users to understand the program's state and any errors that may occur.
