# code-analysis-tools
Guidelines and notes about useful tools to analyze and optimize code.

The idea is to gather some tools that we use to develop high performance code. We should provide information about how to install and use these tools.


## Debugging C++
Use either gdb (linux) or lldb (macOS). The commands specified below work for both. This is useful if you have segfaults and you want to investigate the backtrace for example.

### Usage C++
`gdb <your-executable>`
It will enter gdb and you can set breakpoint `b file.cpp:40` or `breakpoint set -n functionName`.
Afterwards do `run`.

### Usage Python bindings
`gdb python`
In the next step, set again your breakpoints and do `run your_script.py`.  

Once a program crashs, use `bt` to show the full backtrace.

## Checking for speed
[FlameGraph](https://github.com/brendangregg/FlameGraph)

## Finding memory leacks
[Valgrind](https://valgrind.org/)

## Check Eigen malloc 
Use Eigen tools to make sure you are not allocation dynamic memory. see for example [here](https://stackoverflow.com/questions/33664976/avoiding-eigens-memory-allocation). We could point to ProxDDP or ProxQP and show how we use macros ect.

## Checking for memory alloctions
GUI to check how much memory is allocated in every function when executing a program.  
-> Valgrind + [KCachegrind](https://github.com/KDE/kcachegrind)

### Install
`sudo apt-get install valgrind kcachegrind graphviz`
### Usage
```bash
valgrind --tool=massif --xtree-memory=full <your-executable>
kcachegrind <output-file-of-previous-cmd>
```

## External resources
Some very [useful advices to optimize your C++ code that you should have in mind](https://cpp-optimizations.netlify.app/).

> Narrator: "also quite amusing to read..."

A nice online course to [get started with C++](https://gitlab.inria.fr/formations/cpp/gettingstartedwithmoderncpp/tree/master) explaining most of the basic concepts in c++14/17.
