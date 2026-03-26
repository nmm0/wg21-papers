---
title: "Discrete Convolution Algorithms for `mdspan`"
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
The type can be used for a discrete representation of a $r$-dimensional function, where $r$ is the rank of the `mdspan`.
The discrete convolution operation is commonly used in signal processing applications to modify a 


* Does it need to be odd?
