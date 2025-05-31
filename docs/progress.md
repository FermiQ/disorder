# `progress` Module

## Overview

The `progress` module, located in `src/progress.f90`, provides a simple text-based progress bar functionality. This is used in other parts of the `disorder` code, particularly in long-running calculations like the generation of irreducible configurations (`irrconfig` in `configurations.f90`), to give the user visual feedback on the progress of the computation.

The module defines a derived type `cls_prog` which encapsulates the state and appearance of a progress bar.

## Derived Type: `cls_prog`

This type holds all the information needed to display and update a progress bar.

*   **Components**:
    *   `i` (Integer(8)): The current progress value (e.g., number of items processed).
    *   `num` (Integer(8)): The total number of items to be processed (i.e., when `i` reaches `num`, the progress is 100%).
    *   `lens` (Integer(1), Default: 50): The display length of the progress bar in characters.
    *   `head` (Character(len=1), Default: '#'): The character used to represent the completed part of the progress bar.
    *   `tail` (Character(len=1), Default: '-'): The character used to represent the remaining part of the progress bar.

*   **Type-Bound Procedures**:
    *   `set`: Initializes or updates the parameters of the progress bar.
    *   `put`: Prints or updates the progress bar display on the standard output.

## Subroutines (Type-Bound Procedures)

*   **`set(prog, num, lens, head, tail)`**:
    *   **Description**: Initializes or modifies the properties of a progress bar instance.
    *   **Arguments**:
        *   `prog` (Class(cls_prog), Intent(inout)): The progress bar object to be configured.
        *   `num` (Integer(8), Intent(in)): The total number of steps or items for 100% completion.
        *   `lens` (Integer(1), Intent(in), Optional): The desired length of the progress bar in characters. If not present, the existing `prog%lens` (or default 50) is used.
        *   `head` (Character(len=1), Intent(in), Optional): The character for the filled part of the bar. If not present, the existing `prog%head` (or default '#') is used.
        *   `tail` (Character(len=1), Intent(in), Optional): The character for the empty part of the bar. If not present, the existing `prog%tail` (or default '-') is used.
    *   **Logic**: Assigns the provided values to the corresponding components of the `prog` object.

*   **`put(prog, i)`**:
    *   **Description**: Displays or updates the progress bar on the console. It calculates the percentage completed and represents it with `head` and `tail` characters.
    *   **Arguments**:
        *   `prog` (Class(cls_prog), Intent(inout)): The progress bar object. Its `i` component is updated with the current progress.
        *   `i` (Integer(8), Intent(in)): The current progress value (e.g., number of items processed so far).
    *   **Logic**:
        1.  Updates `prog%i` with the input `i`. Ensures `prog%i` does not exceed `prog%num`.
        2.  Calculates `nh`, the number of `head` characters to print, based on the current progress `prog%i`, total `prog%num`, and bar length `prog%lens`.
        3.  Determines the line ending character `br`:
            *   If `prog%i < prog%num` (progress is not complete), `br` is set to `char(13)` (carriage return). This moves the cursor to the beginning of the current line, allowing the next `put` call to overwrite the previous progress bar display, creating an animation effect.
            *   If `prog%i >= prog%num` (progress is complete), `br` is set to `char(10)` (line feed). This moves the cursor to the next line, so the completed progress bar remains visible and subsequent output starts on a new line.
        4.  Writes the progress bar to standard output, formatted as:
            `  [#######-------------------] XX.YY%`
            followed by the `br` character.

## Usage Example

```fortran
program test_progress
  use progress
  implicit none

  type(cls_prog) :: my_bar
  integer(8) :: total_items, i
  integer(1) :: bar_length
  character(len=1) :: fill_char, empty_char

  total_items = 1000
  bar_length = 40
  fill_char = '*'
  empty_char = '.'

  ! Initialize the progress bar
  call my_bar%set(num=total_items, lens=bar_length, head=fill_char, tail=empty_char)
  ! Or, using defaults for lens, head, tail:
  ! call my_bar%set(total_items)

  write(*,*) "Starting a long task:"
  do i = 0, total_items, 50 ! Update every 50 items
    ! ... do some work ...
    call my_bar%put(i)
    call cpu_time(t1) ! Simulate work delay
    do while (t2 < t1 + 0.1)
        call cpu_time(t2)
    end do
  end do
  ! Ensure the 100% is displayed if loop doesn't end exactly on total_items
  if (my_bar%i < total_items) then
     call my_bar%put(total_items)
  end if

  write(*,*) "Task completed!"

end program test_progress
```

This module provides a neat and user-friendly way to indicate progress for time-consuming operations within a command-line Fortran application.
