---
title: "Standard Algorithms for `mdspan`"
document: 
date: today
audience: Library Evolution
author:
  - name: Nicolas Morales
    email: <nmmoral@sandia.gov>
---

# Revision History


## PXXXXR0: 2025-7 Post-Sofia Mailing

- Initial paper revision

# Motivation and Background

[@P0009] introduced to C++ the `mdspan` type, a non-owning multidimensional view.
A distinct property of `mdspan` is that it has a customizable layout that determines how multidimensional indices are mapped to offsets off the `mdspan`'s internal data handle.
Furthormore, `mdspan`s may have customized accessors; these determine how `mdspan` obtains references to data from offsets.
The standard `std::default_accessor` simply dereferences the offset data handle, but custom accessors could generate proxy references or even perform accesses over a network!

A user of `mdspan` may find themselves wanting to apply some of C++'s standard algorithms to an `mdspan`.
However, they would find some difficulty; `mdspan` does not provide `begin()` or `end()` iterators (nor range access).
Furthermore, it is not clear what exactly these iterators would be!

Of course you could imagine what a trivial `mdspan`'s begin and end iterators would be; they could be simply `mds.data_handle()` and `mds.data_handle() + mds.size()`.
For anything more complex than a contiguous `mdspan` with pointer-based `data_handle`s however, we quickly run into problems.
One might need to wrap `data_handle` in some type of iterator; the iterator dereference would just call `access()` on the `mdspan` accessor.
However, iterating is less straightforward.

This is because the index provided to an `mdspan`'s accessor is **not** constrained in any way to be between `0` and `mds.size()`.
So for example, take this (admittedly contrived) mapping and accessor:

```
class weird_accessor {
  // ...
  // Precondition: i >= 1000
  constexpr reference access(data_handle_type p, size_t i) const noexcept {
    return p[i - 1000];
  }
  // ...
};

// mapping
class weird_layout::mapping : public layout_left::mapping {
  // ...
  template<class... Indices>
  constexpr index_type operator()(Indices... i) const noexcept {
    return layout_left::mapping::operator()(i...) + 1000;
  }
  // ...
};
```

This would rule out trying to iterate in the 1-dimensional space in $\left[0, size\right)$.

The only correct way to iterate over every element in an `mdspan` then, is to traverse every element in every extent of every rank.
This now leads to some ambiguity.
Which rank shall be the fastest varying?
What direction is the most useful to step through that rank?

Therefore, this paper proposes that new algorithms (and their associated parallel versions) should be adapted for `mdspan`s, and that operate on whole `mdspan`s rather than iterators or ranges.

# General Design Considerations

## What algorithms are suggested to adapt?

- `for_each`
- `all_of`
- `any_of`
- `none_of`
- `contains`
- `find`
- `find_if`
- `find_if_not`
- `find_last`
- `find_last_if`
- `find_last_if_not`
- `find_first_of`
- `count`
- `count_if`
- `equal`
- `copy`
- `move`
- `transform`
- `replace`
- `replace_if`
- `replace_copy`
- `replace_copy_if`
- `fill`
- `generate`*
- `shuffle`
- `sample`
- `min_element`
- `max_element`
- `minmax_element`
- `inner_product`
- `reduce`
- `transform_reduce`
- `generate_random`

## What algorithms could be implemented but limited to rank-1 views?

- `contains_subrange` - Ranges are meaningless inside an mdspan, so cannot find it
- `find_end` - Similar, we cannot find a sequence of objects since we are not sequenced.
- `adjacent_find` - Same argument
- `mismatch` - Same argument
- `search` - Same argument
- `search_n` - Same argument
- `starts_with`
- `ends_with`
- `copy_n`
- `copy_backward`
- `move_backward`
- `fill_n`
- `reverse`
- `reverse_copy`
- `rotate`
- `rotate_copy`
- `shift_left`
- `shift_right`
- All sorting related algorithms
- `iota`
- `accumulate` - Reduce is more correct instead
- `adjacent_difference`
- `partial_sum`
- `exclusive_scan`
- `inclusive_scan`
- `transform_exclusive_scan`
- `transform_inclusive_scan`
- All ranges fold operations

Though it's worth noting, we could produce a way to get iterators/range from one particular rank.


## What algorithms should not be included?

- All remove algorithms



## Why not allow only whole mdspans and not subsets of elements in mdspans?

This is actually a trick queestion.
This paper actually does allow partial mdspans to be used in an algorithm.
`submdspan` after all, yields an `mdspan` (usually with a different layout) and thus can be used to create `mdspan`s for use in these algorithms.

One interesting benefit of this is safety; `submdspan` already has preconditions; if we assume any `mdspan` we pass is a valid one, we cannot get out of bounds access during the execution of one of these algorithms.

##


