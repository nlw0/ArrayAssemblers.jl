# ArrayAssemblers

ArrayAssemblers offers two methods to assemble arrays and higher-order tensors out of arrays-of-arrays.

The motivation for this package comes mostly from two use patterns:

1. `block(f, a)` is an alternative to a `flatmap` functor. It works essentially the same as `[y for x in c1 for y in f(x)]`. It produces an array and supports do-syntax.
2. `lay(f, a)` works as a shortcut for `hcat(vector_of_vectors...)` or `reduce(hcat, vector_of_vectors)`, but it also supports do-syntax and can take generators.

## Examples
```
julia> using ArrayAssemblers

julia> a1 = [1 2; 3 4]
2×2 Matrix{Int64}:
 1  2
 3  4

julia> a2 = [5 6; 7 8]
2×2 Matrix{Int64}:
 5  6
 7  8

julia> block([a1 a2])
2×4 Matrix{Int64}:
 1  2  5  6
 3  4  7  8

julia> block([a1, a2])
4×2 Matrix{Int64}:
 1  2
 3  4
 5  6
 7  8

julia> block([a1;;; a2])
2×2×2 Array{Int64, 3}:
[:, :, 1] =
 1  2
 3  4

[:, :, 2] =
 5  6
 7  8

julia> lay(a:a+1 for a in 1:2:6)
2×3 Matrix{Int64}:
 1  3  5
 2  4  6

## flatmap behavior with `block`
julia> Zn = block(1:3) do n
           -n:2:n
       end
9-element Vector{Int64}:
 -1
  1
 -2
  0
  2
 -3
 -1
  1
  3

julia> Zn == [x for n in 1:3 for x in -n:2:n]
true

## Assembling a tensor with recursive calls to `lay` and `map` using do-syntax
julia> lay(0:1) do c
           lay(0:2) do b
               map(1:2) do a
                   (c * 3 + b) * 2 + a
               end
           end
       end

2×3×2 Array{Int64, 3}:
[:, :, 1] =
 1  3  5
 2  4  6

[:, :, 2] =
 7   9  11
 8  10  12
```

Image montage (tiling):
```
using TestImages, ImageView
myimages = ["cameraman" "plastic_bubbles_he_512" "woman_darkhair"; "resolution_test_512" "pirate" "walkbridge"]
imshow(block(testimage.(myimages)))
```
![Image montage example](docs/src/imagemontage.png)
