# code-analysis-tools
Guidelines and notes about useful tools to analyze and optimize code.

The idea is to gather some tools that we use to develop high performance code. We should provide information about how to install and use these tools.


## Debugging C++
gdb (linux) or lldb (macOS)

### Usage C++
`gdb <your-executable>`
It will enter gdb and you can set breakpoint `b file.cpp:40` or `breakpoint set -n functionName`.
Afterwards do `run`.

### Usage Python bindings
`gdb python`
In the next step, set again your breakpoints and do `run your_script.py`.

## Checking for speed
[FlameGraph](https://github.com/brendangregg/FlameGraph)

## Finding memory leacks
[Valgrind](https://valgrind.org/)

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
