# Searcher support for ranges::search

## Abstract 

This paper propose to add searcher support for ranges::search, to be on par with std::search.

## Motivation and Scope

The current std::search function support searchers, but not ranges::search, this is confusing when you want to migrate to the new ranges library.

```cpp
std::string haystack{"hello world !"};
std::string needle{"orl"};
std::boyer_moore_searcher srchr{needle.begin(), needle.end()};

auto result_std{std::search(haystack.begin(), haystack.end(), srchr)}; // This works
auto result_rng{std::ranges::search(haystack, srchr)}; // This doesn't work in C++20
```

The scope of this paper is limited to searcher support for ranges::search, no new searchers with sentinel support are proposed.

## Design decisions

Two new constrained overloads for the CPO ranges::search will be added, one taking a range and a searcher, the other taking a iterator / sentinel and a searcher. Thus, making these calls valid for a type *S* modeling `searcher` and a type *R* modeling `range`:

```cpp
S srchr{/*...*/};
R rng{/*...*/};

auto rng_call_ret = std::ranges(rng, srchr);
auto iter_call_ret = std::ranges(std::ranges::begin(rng), std::ranges::end(rng), srchr);
```

The searcher parameter will be constrained with a new `searcher` concept that will ensure that the type is callable with an iterator and a sentinel of some range *R*, and that its return type can be destructured into two variables (with a structured binding) that can be used to construct a subrange of type *R*.

Basically this code should compile for a type *S* to model the concept `searcher` for a range of type *R*:

```cpp
S srchr{/*...*/};
R rng{/*...*/};

auto [begin_ret, end_ret]{std::invoke(srchr, std::ranges::begin(rng), std::ranges::end(rng))};
std::ranges::subrange<std::ranges::iterator_t<R>> sub_rng{begin_ret, end_ret};
```

The limitation of this design is that if a type T model both `searcher` and `range`, the call:

```cpp
R rng{/*...*/};
T rng_and_srchr{/*...*/};

std::ranges::search(rng, rng_and_srchr); // search(rng, rng) or search(rng, srchr) ?
```

Will be ambiguous, when in C++20 it was valid. But because this would be very unlikely to happen it is not considered as an issue.

A WIP implementation based on range-v3 can be found here : [https://godbolt.org/z/dHWPfj](https://godbolt.org/z/dHWPfj).

## Wording

TODO.