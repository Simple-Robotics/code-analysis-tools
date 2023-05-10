# code-analysis-tools
Guidelines and notes about useful tools to analyze and optimize code.

The idea is to gather some tools that we use to develop high performance code. We should provide information about how to install and use these tools.
Let's begin by collecting all instructions here and later move them to subfolders for each tool if readme gets confusing.

## Debugging C++ code
If you execute compiled experimental code or run a python script that is corresponding c++ bindings [Segmentation Faults](https://en.wikipedia.org/wiki/Segmentation_fault). Specific tools can help you to fix them.  

**FIRST: compile in Debug mode**  

Use either gdb (linux) or lldb (macOS). The commands specified below work for both. 
### Usage C++
```bash
# Start debugging session
gdb <your-executable>

# set breakpoints in two different ways, either on specific line for function
b file.cpp:40`
breakpoint set -n functionName

# start
run
```
See [here](https://web.stanford.edu/class/archive/cs/cs107/cs107.1194/resources/gdb) for further details.

### Usage Python bindings
```bash 
gdb python
# set breakpoints ect
run your_script.py
```

Once a program crashs, use `bt` to show the full backtrace.

## Performance analysis 
Checking how much time is spent for every function. Can help you to find the bottleneck in your code.
[FlameGraph](https://github.com/brendangregg/FlameGraph) is a nice visual tool to display your stack trace.

### Install 1
Use [Rust-powered flamegraph](https://github.com/flamegraph-rs/flamegraph) -> **fast**  

```bash
# Install tools needed for analysis like perf
sudo apt-get install linux-tools-common linux-tools-generic linux-tools-`uname -r`

# Install rust on linux or macOS
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh

# Now you have the rust package managar cargo and you can do
cargo install flamegraph

# Necessary to allow access to cpu (only once)
echo -1 | sudo tee /proc/sys/kernel/perf_event_paranoid
echo 0 | sudo tee /proc/sys/kernel/kptr_restrict

# Command to produce your flamegraph (use -c for custom options for perf)
flamegraph -o sparse_wrapper.svg -v -- example-cpp
```

### Install 2
Use classic perl [flamegraph](https://github.com/brendangregg/FlameGraph)

```bash
# Install tools needed for analysis like perf
sudo apt-get install linux-tools-common linux-tools-generic linux-tools-`uname -r`

# Copy the repo
https://github.com/brendangregg/FlameGraph.git

# Necessary to allow access to cpu (only once)
echo -1 | sudo tee /proc/sys/kernel/perf_event_paranoid
echo 0 | sudo tee /proc/sys/kernel/kptr_restrict

# Run perf on your executable and create output repot in current dir
perf record --call-graph dwarf example-cpp
perf script > perf.out
# You can read the report with cat perf.out ..

cd <cloned-flamegraph-repo>
./stackcollapse-perf.pl <location-of-perf.out> > out.folded
./flamegraph.pl out.folded > file.svg
# Now open the file.svg in your favorite browser in enjoy the interactive mode
```
As the process with the default flamegraph repo is quite a pain, you can write your own script like @ManifoldFR in [proxDDP](https://github.com/Simple-Robotics/proxddp/blob/wj/nnl-rollout/scripts/make_flamegraph.sh).

## Finding memory leacks
[Valgrind](https://valgrind.org/) can automatically detect many memory management and threading bugs.
```
sudo apt install valgrind
# Use valgrind with input to check for mem leak
valgrind --leak-check=yes myprog arg1 arg2
```
Check [here](https://stackoverflow.com/questions/5134891/how-do-i-use-valgrind-to-find-memory-leaks) and [doc](https://valgrind.org/docs/manual/quick-start.html) for further explanation.

## Check Eigen malloc 
Use Eigen tools to make sure you are not allocating memory where you do not want to do so -> it is slowing down your program. Check [proxqp](https://github.com/Simple-Robotics/proxsuite/blob/794607d4e35626fc4d5bb704f4f1796347412e71/include/proxsuite/fwd.hpp#L40) or [here](https://stackoverflow.com/questions/33664976/avoiding-eigens-memory-allocation).  

The macros defined in `ProxQP` allow us to do
```cpp
PROXSUITE_EIGEN_MALLOC_NOT_ALLOWED();
output = superfast_function_without_allocations();
PROXSUITE_EIGEN_MALLOC_ALLOWED();
```
and if this code is compiled in `Debug` mode, we will have assertation errors if eigen is allocation memory inside the function.

## Checking for memory allocations
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

## Debug Github CI
If you have a pipeline that is failing, and you would like to check some quick fixes directly on the CI machine [debug-via-ssh](https://github.com/marketplace/actions/debug-via-ssh) is precious:
Sign into your account on [ngrok](https://ngrok.com/) (you can use github) and follow the readme to set it up locally (2mins).
Copy the token you obtained from ngrok into the secrets section of your repo, specify a password for the SSH connection also as token of the repo. 
Copy this to your workflow at the position where you would like to stop:
```bash
- name: Start SSH session
  uses: luchihoratiu/debug-via-ssh@main
  with:
    NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
    SSH_PASS: ${{ secrets.SSH_PASS }}
```
Run the CI and follow the output. Note: the option `continue-on-error: True` can be very useful the continue a failing workflow until the point where you ssh to it.
