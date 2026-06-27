# RGD
Readable Graph Description

A simple, readble, flexible graph data format.


## Features
- The things you expect:
    - Nodes
    - Edges: Directed, Undirected, Bi-Directional
- Graph metadata
- Multiple graphs per file
- TOML inspired property descriptions
- Hypergraphs (and directed hyperedges)
- TGF compatibility


## Why?
There is a gap between *simple* graph formats, and *featureful* formats.
RGD is meant to close that gap by providing a minimal syntax for expressing
complex relationships.


## Example

```rgd
# graph
started = 2026-03-01
update-frequency = "2/wk"

# nodes
A: role = "Collector"
{B, C}: role = "Observer"
D: role = "Validator"

# edges
C -> A: dates = [2026-03-02, 2026-03-15]
B -> A: dates = [2026-03-12]
D -> A: dates = [2026-03-02, 2026-03-15], results = [true, true]
```

Easy to simplify

```rgd
{A, B, C}
# edges
B -> A: [1, 2, 3]
C <- A: [0.1, 0.2, 0.3]
```


Hypergraphs!

```rgd
{A, B, C, D, E, F}
#
{A, B, C}:  role = "worker"
{D, E}:     role = "manager"
{F}:        role = "leader"
```


TGF Compatible

```rgd
A leader
B follower
C follower
#
C A at 2026-06-01T12:05Z
B A at 2026-06-02T01:35Z
```
