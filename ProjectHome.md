# IMPORTANT #

Due to the closing of `code.google.com` this repository has been moved to
[github](https://github.com/sav-tools/wrapped-intervals).


<a href='Hidden comment: 
This project provides the implementation of a new interval analysis, called *wrapped interval analysis*. It is implemented in C++ using the [http://llvm.org LLVM] framework, and currently, it has only support for C programs which are translated to LLVM IR (Intermediate Representation) via [http://clang.llvm.org Clang].

=Interval Analysis=

The goal of interval analysis is to determine an approximation of the
sets of possible values that may be stored in integer-valued variables
at various points in a computation.  To keep this tractable, interval
analysis approximates such a set by keeping only its smallest and
largest possible value. Interval analysis is not new and in fact, it has had a long
history being a frequent example of abstract interpretation since the field
was defined by P. and R. Cousot in 1977.

One motivation for this project is that most programming languages, included C, provide one or more fixed-width integer types and arithmetic operations on these types do not have the usual integer semantics but instead, they obey laws of modular arithmetic. However, interval analysis implementations often assume that
integers have unlimited precision which is unsound in presence of overflows. One easy way of fixing this is to assume _top_ whenever overflow is possible but this can easily lead to an important loss of precision.

Example:

```
char modulo(char x, char y, int p){
  y=-10;
  if(p) x=0;
  else  x=100;
  while (x >= y){      // line 1
    x = x-y;           // line 2 
  }                    // line 3 
  return x;            // line 4

}
```

Traditional interval analysis would ignore the
fixed-width nature of variables, in this case leading to
the incorrect conclusion that line 4 is unreachable.
A (traditional) _width aware_ interval analysis can do better.
During analysis it finds that, just before line 2, x=[0,120].
Performing line 2"s abstract subtraction operation, it observes
the wrap-around, and finds that x = [-128,127] at line 3, and that
x = [-128,-11] at line 4 (with the help of the conditional branch x < y), and as the result of the function call. Although this result is correct it is a bit disappointing as we will discuss below.

The other motivation has been the fact that LLVM IR carefully specifies the bit-width of all integer values, but in most cases does not specify whether values are signed or unsigned.  This is because, for most operations, two"s complement arithmetic (treating the
inputs as signed numbers) produces the same bit-vectors as unsigned
arithmetic.  Only for operations that must behave differently
for signed and unsigned numbers, such as comparison operations,
does LLVM code indicate whether operands are signed or unsigned.
In general it is not possible to determine which LLVM variables
are signed and which are unsigned. One simple solution is to assume that all integers are either signed or unsigned but very importantly, no matter which decision is taken it can always lead to a loss of precision.

* If we treat all numbers are signed:
Assume x=0110  and y=[0001,0011] in 4 bits. Note that we represent intervals as bit-patterns. The operation x+y returns [0111,1001] ([7,9]) if variables are treated as unsigned. However, if they are treated as signed then the addition will be deemed to _wraparound_, given [1000,0111] which corresponds to the largest interval [-8,7].

* If we treat all numbers are unsigned:
Assume now x=1110 and y=[0001,0011] also in 4 bits. The operation x+y returns [1111,0001] ([-1,1])  if variables are treated as signed. However, if they are treated as unsigned then the addition will also wraparound.

= Our Contribution: Wrapped Interval Analysis =

We have implemented another interval analysis based on a new abstract domain called *wrapped intervals* which fits naturally with the LLVM IR. In wrapped intervals each bound is treated as merely a _bit-pattern_, considering only its signedness when necessary for the operation involved (e.g., comparisons and division). This interpretation based on bit-patterns allows wrapped intervals choosing always the best results between the unsigned and signed version.

Moreover, wrapped intervals are allowed to _wraparound_ without going to _top_ directly as fixed-width interval domain would do. The two cases where either the interval would overflow or not are kept track by having a special type of disjunctive interval information.

Based on our experimental evaluation with [http://www.spec.org/cpu2000/ SPEC CPU 2000] programs, wrapped intervals are more precise than classical interval analysis but still quite efficient (just a constant factor overhead with respect to the classical interval domain).

Coming back to the *modulo* example  with wrapped intervals.

Inspection of the code tells us that the result of such a call must always be smaller than -118, since at each iteration, x increases by 10, and if x = [-118,-11] at line 4, then on the previous iteration x = [-128,-11], so the loop would have terminated.
Thus we would have hoped for the analysis result
x = [-128,-119]. The traditional bounds analysis misses this result because it
considers ``wrap around"" to be a leap to the opposite end
of the number line, rather than the incremental step that it is.
Using our proposed domain gives the precision we hoped
for.  Finding again that x = [0,120] at line 2,
we derive the bounds x = [10,127] \/ x=[-128,-126] (x is in [10,127] _or_ in [-128,-126]) at line 3. On the next iteration we have x = [0,127] \/ x = [-128,-126] at line 1, x = [0,127] at line 2, and x = [10,127] \/ x = [-128,-119] at line 3. This yields x = [-128,-119] at line 4, and one more iteration proves this to be a fixed point.


= Get the code, build it, and play with it =

Click on [InstallationAndUsage here] for all the details. We also provide Doxygen documentation with the code. Click also [BriefDescriptionOfImplementation here] to read some brief description of the code.

= References =

"[http://clip.dia.fi.upm.es/~jorge/docs/wrapped-intervals-aplas12.pdf  Signedness-Agnostic Program Analysis: Precise Integer Bounds for Low-Level Code]" by
Jorge A. Navas, Peter Schachte, Harald Sondergaard and Peter J. Stuckey. Published in  [http://aplas12.kuis.kyoto-u.ac.jp  APLAS"12].

A great paper to understand the issues of integer overflow and defined vs undefined wraparound:

"[http://www.cs.utah.edu/~regehr/papers/overflow12.pdf  Understanding Integer Overﬂow in C/C++]" by
Will Dietz, Peng Li, John Regehr, and Vikram Adve. Published in ICSE 2012.



= Contact Us =

This project is open source. If you want to collaborate, report bugs, or give any feedback please feel free to email jorge.navas@unimelb.edu.au

= Acknowledgments =

This work has been supported by the Australian Research Council through grant DP110102579.
We thank also to the members of the project [http://code.google.com/p/range-analysis/ range-analysis] for fruitful discussions and for making LLVM SSI (Static Single information) construction pass available (if you are curious read [BriefDescriptionOfImplementation this page] to see why we use SSI form).
'></a>