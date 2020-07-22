# Compiletime RNG (Linear Congruential Generator)

## What is a linear congruential pseudorandom number generator (LCG)?
The only thing that is necessary to unterstand LCG's is [Wikipedia](https://en.wikipedia.org/wiki/Linear_congruential_generator).
Not everything will be important,
but it will give you a really good understanding of it.

The LCG makes use of the recurrence relation
```
X(n+1) = ( a * X(n) + c ) % m
```
and consists of the following parts:
| Term | Description | Restriction |
| :--: | :---------: | :---------: |
| X(n+1) | next number | none |
| X(n) | current number | none |
| X(0) | starting value (seed) | 0<= X(0) < m |
| m | modulus | m > 0 |
| a | multiplier | 0 < a < m |
| c | constant | 0 <= c < m |

## Setting the parameters
Values are based on [*minstd_rand*](http://www.cplusplus.com/reference/random/minstd_rand/)
```cpp
// modulus (2^31 - 1) or 0x7FFFFFFF in hex
constexpr uint32_t m{ 2147483647 };
// multiplier
constexpr uint32_t a{ 48271 };
// constant / increment
constexpr uint32_t c{ 0 };

// seed based on compile time
constexpr uint32_t seed = ((__TIME__[7] - '0') * 1 + (__TIME__[6] - '0') * 10
                        + (__TIME__[4] - '0') * 60 + (__TIME__[3] - '0') * 600
                        + (__TIME__[1] - '0') * 3600 + (__TIME__[0] - '0') * 36000);

```
For a list of common parameters see [Wikipedia's section](https://en.wikipedia.org/wiki/Linear_congruential_generator#Parameters_in_common_use)

## Recursion
At compile time, you can't just use your normal functions for this.
You have to use recursion along with a technique called "Template Metaprogramming".
This is a whole topic on itself, but for this application it is sufficient to know that by
using templates you can resolve certain functions at compile time, if they meet some requirements.
The formula above increments it's sequence postion for the next number. This can't be done at compile time.
Therefore we use recursion in combination with a template:
```cpp
template <uint32_t Const>
struct MakeConst
{
    enum { Value = Const };
};

constexpr uint32_t RecursiveRNG(uint32_t num)
{
    return (c + a * ((num > 0) ? RecursiveRNG(num - 1) : seed)) & m;
}
```
Use it with
```cpp
MakeConst<RecursiveRNG(__COUNTER__ + 1)>::Value
```

## Sources
- https://gist.github.com/mattypiper/158b88765170cd72b7bbb0f6e1c5cddd#file-stringobf-cpp11-cpp-L19
- https://github.com/andrivet/ADVobfuscator/tree/master/Docs
