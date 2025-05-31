# `groups` Module

## Overview

The `groups` module, located in `src/groups.f90`, plays a crucial role in processing symmetry information to classify and group atomic sites. After the `symmetry` module identifies equivalent sites (via `eqamat`), the `groups` module takes this equivalency information to partition the sites involved in substitution into distinct, ordered sets or "groups." This grouping is essential for the `configurations` module to efficiently generate irreducible atomic configurations, as it ensures that symmetrically identical sites are treated collectively.

## Key Subroutines

*   **`grouping(group, eqamat, k, na, a, x, symbol, natom, site)`**:
    *   **Description**: This is the main subroutine of the module. It takes the equivalency matrix (`eqamat`) and identifies distinct groups of symmetrically related atomic sites. It then defines the boundaries of these groups in the `group` array. If necessary, it reorders atoms in the coordinate list (`x`) and the `eqamat` to ensure a canonical ordering, which simplifies downstream processing.
    *   **Arguments**:
        *   `group` (Integer(2), Allocatable, Intent(out)): Output array. `group(i)` stores the starting index (in the ordered list of sites) of the i-th group of symmetrically equivalent sites. `group(ng+1)` marks the end of the last group.
        *   `eqamat` (Integer(2), Intent(in out)): The equivalency matrix from the `symmetry` module. Each column `eqamat(:,o)` represents a symmetry operation, and `eqamat(i,o)` is the site that site `i` moves to under that operation. This can be modified if reordering occurs.
        *   `k` (Integer(2), Intent(in)): The number of atoms of the first substituting species. Used to determine how many groups are relevant for substitution.
        *   `na` (Integer(2), Intent(in)): The total number of atoms on the sublattice being substituted.
        *   `a` (Real(8), Intent(in)): Lattice vectors (not directly used for grouping logic but passed to `writepos` if reordering happens).
        *   `x` (Real(8), Intent(in out)): Atomic coordinates. Can be modified if reordering occurs.
        *   `symbol` (Character(len=2), Intent(in)): Atomic symbols (passed to `writepos` if reordering happens).
        *   `natom` (Integer(2), Intent(in)): Number of atoms of each type (passed to `writepos` if reordering happens).
        *   `site` (Integer(1), Intent(in)): The index of the atomic species being substituted.
    *   **Logic**:
        1.  Iterates through all sites (`1` to `na`) involved in the substitution.
        2.  For each unvisited site `i`, it identifies all other sites `ic` that are symmetrically equivalent to `i` by applying all symmetry operations in `eqamat`. These form a "group."
        3.  It marks all sites in this newly found group as visited (`occ(ic) = .false.`) and records them in the `order` array. The size of this group is stored in `deg(ng)`.
        4.  After identifying all groups, it sorts the sites within each group by their original indices (`call sort(order(ib:ie), ...)`).
        5.  **Reordering Check**: It checks if the atoms are already ordered according to these symmetric groups. Specifically, it verifies if the `j`-th atom in the sorted `order` array is indeed at index `j`. If not (`order(j) /= j`), it means the original `SPOSCAR` file did not list atoms in an order that aligns with the symmetry grouping. In this case:
            *   The `reorder` subroutine is called to physically reorder the atomic coordinates in `x` and update `eqamat` to reflect this new atom indexing.
            *   A message "NOTE: The order of atomic positions has been changed !" is printed.
            *   A new structure file `SPOSCAR_NEW` is written with the reordered atoms (using `writepos` from the `structure` module).
        6.  **Group Finalization**: It determines the number of groups (`ng`) relevant for placing the `k` atoms of the first substituting species. The logic `sum(deg(1:i-1)) >= na-k+1` seems to aim to define enough groups to accommodate placing `k` atoms, ensuring that if `k` atoms are placed starting from the first group, they don't exceed the total available sites minus those needed for other species.
        7.  The `group` array is allocated and populated. `group(1)` is always 1. `group(i)` for `i > 1` is `sum(deg(1:i-1)) + 1`, effectively storing the cumulative sum of group sizes to mark the starting index of each group in the (potentially reordered) list of sites.

*   **`reorder(order, eqamat, natom, x, site)`**:
    *   **Description**: This subroutine is called by `grouping` if the atomic sites in the input structure file are not sorted according to their symmetry groups. It reorders the atomic coordinates in `x` and updates the `eqamat` to be consistent with the new atomic indexing.
    *   **Arguments**:
        *   `order` (Integer(2), Intent(in)): The desired order of atomic indices. `order(j)=i` means original atom `i` should now be at position `j`.
        *   `eqamat` (Integer(2), Intent(in out)): The equivalency matrix, which is updated to reflect the new atom indexing.
        *   `natom` (Integer(2), Intent(in)): Number of atoms of each type, used to determine the slice of `x` to reorder.
        *   `x` (Real(8), Intent(in out)): Atomic coordinates, reordered in place.
        *   `site` (Integer(1), Intent(in)): The index of the atomic species being substituted.
    *   **Logic**:
        1.  Computes the inverse order mapping (`iorder`): `iorder(order(j))=j`. So, `iorder(i)` gives the new position of original atom `i`.
        2.  Updates `eqamat`: For each symmetry operation, if atom `j` (in the new order) moves to `eqamat(j, op)`, this target site is updated based on where the original `order(j)` atom moved under that operation, and then mapped back to the new ordering using `iorder`.
        3.  Reorders the relevant slice of the `x` coordinate array: `x(:,ib:ie) = x(:,order+ib-1)`. The `ib` and `ie` define the block of coordinates belonging to the `site` being substituted.

## Role in Symmetry Handling

The `groups` module acts as a bridge between raw symmetry operation data (`eqamat` from `symmetry` module) and the combinatorial generation of configurations (`configurations` module).

1.  **Classification**: It classifies the `na` sites involved in substitution into `ng` distinct sets based on symmetry equivalence. All sites within a single group are indistinguishable from each other by symmetry operations of the parent lattice.
2.  **Canonical Ordering**: By potentially reordering atoms and `eqamat`, it ensures that these groups are contiguous and ordered in the site list. This simplifies the logic in `irrconfig` as it can iterate through placing atoms group by group.
3.  **Defining Scope**: The `group` array tells `irrconfig` the start and end indices for each symmetry-distinct block of sites. This is crucial for constructing configurations systematically. For example, when `irrconfig` decides to place an atom of a certain type, it knows it can place it at the beginning of a group, and symmetry will dictate the placement of others or how to count such arrangements.

Without this grouping step, `irrconfig` would have a much harder time avoiding overcounting configurations or ensuring that only symmetrically unique arrangements are generated. The `group` array essentially provides a more structured view of the sites, tailored for efficient generation of irreducible configurations.
The optional reordering and writing of `SPOSCAR_NEW` is a side-effect that makes the internal representation consistent with a canonically ordered structure file, which can be useful for verification or further analysis.
