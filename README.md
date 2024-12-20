# A Simple Graph Processing Software for Shared Disaggregated Memory

A single threaded graph processing framework where is memory is manually
managed.
Useful for disaggregated memories.

This is a simple framework for graph processing.
I have implemented this to quickly test multihost graph processing.

Depends on:
* cpp-arg-parse (`github.com/kaustav-goswami/cpp-arg-parse`)
* cmake

```sh
make -j4
# The master node is called via
./simple_graph \
    --algorithm allocator \
    --graph tests/graph.csr \
    --total-hosts 3 \
    --host-id 0
# The worker nodes are called via
./simple_graph --algorithm bfs \
    --graph tests/graph.csr \
    --total-hosts 3 \
    --host-id 1
./simple_graph --algorithm dfs \
    --graph tests/graph.csr \
    --total-hosts 3 \
    --host-id 2
```

The graph is read from a file in csr format and is stored in class Graph as a
CSR graph in memory.

`class Graph` is a host-side object, that has a pointer to the start of the
remote memory.
It assumes that the remote memory is a memory-mapped range on the
disaggreged/remote device.
Only the allocator initializes the graph, which also needs to be the master
node.
Once the setup is complete, a synchronization variable is set in the shared
memory range (needs to be uncacheable; must be hardware managed).
The workers then begin working on the algorithm specified.

Current implementation is single-threaded.

## Graph Format

The input graphs can be **directed** and **weighted**.
The program expects that the graph will be in csr format with:
```
<Vertex Count>
<Edge Count>
<Values of the Row Pointer Array separated by " ">
<Values of the Column Index Array separated by " ">
<(Optional) Weights Array separated by " ">
```

Sample graphs are stored in `tests/` directory.

The graph is stored pretty much the same way in the memory with the addition
of some more metadata.
All entries are assumed to be `uint64_t`.
Following the a flat representation of the mmaped region.
```
_____________________________________________________________________ .. ______
| V | E | size | size | size | sync | row ptr | col idx | weights         | | |
|___|___|______|______|______|______|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|__ .. _|_|_|
```

The start of the graph is stored as _graph in class Graph.
Each entry is separated by sizeof(uint64_t).

## Building

```sh
git clone git@github.com:kaustav-goswami/simple-graph.git
cd simple-graph
git submodule init
git submodule update
mkdir build
cd build
cmake ..
make -j4
```

## Supported Graph Algorithms

* BFS
* DFS
* BC
* SSSP
* TC
* CC
* PR

Cite: The algorithm implementation code is mostly AI generated.
