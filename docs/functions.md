# `functions` Module

## Overview

The `functions` module, located in `src/functions.f90`, provides a collection of general-purpose mathematical and utility functions used by other modules within the `disorder` codebase. These include binary search algorithms, combinatorial calculations, vector/matrix operations, and sorting routines.

## Mathematical Functions

*   **`nchoosek(n, k)`**:
    *   **Description**: Calculates the binomial coefficient "n choose k", which is the number of ways to choose `k` items from a set of `n` items without regard to the order of selection.
    *   **Arguments**:
        *   `n` (Integer(2)): Total number of items.
        *   `k` (Integer(2)): Number of items to choose.
    *   **Returns**: `nchoosek` (Integer(8)): The result of C(n, k).
    *   **Notes**: Uses `real(8)` for intermediate calculations to handle potentially large numbers before converting to `integer(8)`. Adds 0.5 before truncation for rounding.

*   **`norm(A)`**:
    *   **Description**: Calculates the Euclidean norm (or magnitude) of a 3D vector `A`.
    *   **Arguments**:
        *   `A` (Real(8), Dimension(3)): The input vector.
    *   **Returns**: `norm` (Real(8)): The Euclidean norm, sqrt(A(1)² + A(2)² + A(3)²).

*   **`cross(A, B)`**:
    *   **Description**: Computes the cross product of two 3D vectors `A` and `B`.
    *   **Arguments**:
        *   `A` (Real(8), Dimension(3)): The first input vector.
        *   `B` (Real(8), Dimension(3)): The second input vector.
    *   **Returns**: `cross` (Real(8), Dimension(3)): The resulting vector from A x B.

*   **`trace(A)`**:
    *   **Description**: Calculates the trace of a 3x3 integer matrix `A` (sum of the diagonal elements).
    *   **Arguments**:
        *   `A` (Integer(1), Dimension(:,:)): The input 3x3 matrix. Assumed to be at least 3x3.
    *   **Returns**: `trace` (Integer(1)): The sum A(1,1) + A(2,2) + A(3,3).

*   **`det(A)`**:
    *   **Description**: Computes the determinant of a 3x3 integer matrix `A`.
    *   **Arguments**:
        *   `A` (Integer(1), Dimension(3,3)): The input 3x3 integer matrix.
    *   **Returns**: `det` (Integer(1)): The determinant of matrix `A`.

*   **`det2(A)`**:
    *   **Description**: Computes the determinant of a 2x2 real matrix `A`.
    *   **Arguments**:
        *   `A` (Real(8), Dimension(2,2)): The input 2x2 real matrix.
    *   **Returns**: `det2` (Real(8)): The determinant of matrix `A`.

*   **`det3(A)`**:
    *   **Description**: Computes the determinant of a 3x3 real matrix `A`. This is similar to `det(A)` but for real matrices.
    *   **Arguments**:
        *   `A` (Real(8), Dimension(3,3)): The input 3x3 real matrix.
    *   **Returns**: `det3` (Real(8)): The determinant of matrix `A`.

*   **`inverse(A)`**:
    *   **Description**: Calculates the inverse of a 3x3 real matrix `A` using the adjugate matrix method.
    *   **Arguments**:
        *   `A` (Real(8), Dimension(3,3)): The input 3x3 real matrix.
    *   **Returns**: `inverse` (Real(8), Dimension(3,3)): The inverse of matrix `A`.
    *   **Notes**: Relies on `det2` (for cofactors) and `det3` (for the main determinant).

## Utility Functions

*   **`binsearch_2(n, a)`**:
    *   **Description**: Performs a binary search on a sorted integer(2) array `a` to find the index of the element just less than or equal to `n`.
    *   **Arguments**:
        *   `n` (Integer(2)): The value to search for.
        *   `a` (Integer(2), Dimension(:)): The sorted array to search within.
    *   **Returns**: `binsearch_2` (Integer(2)): The index of the found element. If `n` is greater than or equal to the last element, it returns the index of the last element.
    *   **Notes**: The array `a` must be sorted in ascending order for the binary search to work correctly.

*   **`binsearch_8(n, a)`**:
    *   **Description**: Similar to `binsearch_2`, but operates on integer(8) arrays. Performs a binary search on a sorted integer(8) array `a` to find the index of the element just less than or equal to `n`.
    *   **Arguments**:
        *   `n` (Integer(8)): The value to search for.
        *   `a` (Integer(8), Dimension(:)): The sorted array to search within.
    *   **Returns**: `binsearch_8` (Integer(2)): The index (note: return type is integer(2)) of the found element.
    *   **Notes**: The array `a` must be sorted.

*   **`complement(a, m)`**:
    *   **Description**: Finds the elements in the range `1` to `m` that are *not* present in the input array `a`.
    *   **Arguments**:
        *   `a` (Integer(2), Dimension(:)): An array of integers.
        *   `m` (Integer(2)): The upper bound of the range to consider.
    *   **Returns**: `complement` (Integer(2), Dimension(m)): An array containing elements from `1` to `m` not found in `a`. The actual size of relevant data in the returned array will be `m - size(unique_elements_in_a)`. The rest of the elements will be 0.

*   **`width(i_1, i_2, i_4, i_8)`**:
    *   **Description**: Calculates the number of characters required to represent an integer, effectively its display width. It uses optional arguments for different integer kinds.
    *   **Arguments (Optional)**:
        *   `i_1` (Integer(1))
        *   `i_2` (Integer(2))
        *   `i_4` (Integer(4))
        *   `i_8` (Integer(8))
    *   **Returns**: `width` (Integer(1)): The length of the trimmed, left-adjusted string representation of the input integer.
    *   **Notes**: Only one optional argument should be provided per call.

*   **`sort(A, left, right)` (Recursive Subroutine)**:
    *   **Description**: Sorts an integer(2) array `A` in place using a recursive quicksort algorithm.
    *   **Arguments**:
        *   `A` (Integer(2), Intent(in out), Dimension(:)): The array to be sorted.
        *   `left` (Integer(2), Intent(in)): The starting index for the sort.
        *   `right` (Integer(2), Intent(in)): The ending index for the sort.
    *   **Notes**: This is a standard quicksort implementation.

## How These Functions Are Used

*   **Binary Search (`binsearch_2`, `binsearch_8`)**: Likely used in `configurations` module for quickly finding positions or indices within sorted arrays that might represent configuration states or mappings.
*   **`nchoosek`**: Central to combinatorial calculations in `configurations.f90` (e.g., in `confnum` and `matrixE`) to determine the number of possible atomic arrangements.
*   **Vector/Matrix operations (`norm`, `cross`, `trace`, `det`, `det2`, `det3`, `inverse`)**: Essential for symmetry operations in `symmetry.f90` (e.g., applying rotations, reflections, calculating cell volumes, transforming vectors).
*   **`complement`**: Could be used in `configurations.f90` or `groups.f90` to identify available sites or remaining elements in a set.
*   **`width`**: Used by `stdout.f90` for formatting output to ensure proper alignment of numbers.
*   **`sort`**: Used to bring configurations or lists of sites into a canonical order, which is crucial for identifying unique configurations (e.g., in `configurations.f90`'s `binary` or `multinary` subroutines).
