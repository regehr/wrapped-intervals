# Checkout and installation of LLVM and Clang #

1) Get the required tools:
```
   Read http://llvm.org/docs/GettingStarted.html#requirements
```
2) Checkout LLVM:
```
   LLVM_ROOT=any_directory_you_like
   cd $LLVM_ROOT
   svn co -r 139364 http://llvm.org/svn/llvm-project/llvm/trunk llvm 
```
3) Checkout Clang (C/C++ frontend):
```
   cd $LLVM_ROOT/llvm/tools
   svn co -r 139290 http://llvm.org/svn/llvm-project/cfe/trunk clang
```
4) Build LLVM and Clang:
```
   cd $LLVM_ROOT/llvm 
   mkdir build (for building without polluting the source dir)
   cd build
   ../configure -enable-optimized=no CC="gcc -m32" CXX="g++ -m32"
   make
   This builds both LLVM and Clang for debug mode.
```

# Checkout and Installation of Wrapped Interval Analysis #
```
   svn checkout http://wrapped-intervals.googlecode.com/svn/trunk/ wrapped-intervals
   cd wrapped-intervals
  ./configure --with-llvmsrc=$LLVM_ROOT/llvm --with-llvmobj=$LLVM_ROOT/llvm/build
  ./make
   cd docs && make (optional)
   change $CLANG_PATH and $OPT_PATH in tools/run.sh
```
# Run #
```
Usage: tools/run.sh prog[.c|.bc]  Pass [options] 

Pass is the pass that we want to run: 
  -wrapped-range-analysis        fixed-width wrapped interval analysis
  -range-analysis                fixed-width classical interval analysis
    options:
      -widening n                n is the widening threshold (0: no widening)
      -narrowing n               n is the number of narrowing iterations (0: no narrowing)
      -alias                     by default, -no-aa which always return maybe. If enabled 
                                 then -basic-aa and -globalsmodref-aa are run to be more precise
                                 with global variables.
      -enable-optimizations      enable LLVM optimizations
      -inline n                  inline function calls whenever possible if the size of 
                                 the function is less than n. (0: no inlining). 
                                 A reasonable threshold n is, e.g., 255.
      -instcombine               remove redundant instructions.
                                 It can improve precision by removing problematic casting 
                                 instructions among many other things.
      -insert-ioc-traps          Compile .c program with -fcatch-undefined-ansic-behavior 
                                 which generates IOC trap blocks.  
                                 Note: clang version must support -fcatch-undefined-ansic-behavior    
                       
  general options:
    -help                          print this message
    -stats                         print stats
    -time                          print LLVM time passes
    -dot-cfg                       print .dot file of the LLVM IR
    -debug                         print debugging messages
```