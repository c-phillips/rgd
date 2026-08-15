# RGD
Readable Graph Description

A simple, readable, flexible graph data format.


## Features
- The things you expect:
    - Nodes
    - Edges: Directed, Undirected, Bi-Directional
- Graph metadata
- Multiple graphs per file
- TOML inspired property descriptions
- Multigraphs
- Hypergraphs (and directed hyperedges)
- TGF compatibility


## Why?
There is a gap between *simple* graph formats, and *featureful* formats.
RGD is meant to close that gap by providing a minimal syntax for expressing
complex relationships.


## Examples

```rgd
# graph
started = 2026-03-01
update-frequency = "2/wk"

# nodes
A: role      = "Collector"
{B, C}: role = "Observer"
D: role      = "Validator"

# edges
C -> A: dates = [2026-03-02, 2026-03-15]
B -> A: dates = [2026-03-12]
D -> A: dates = [2026-03-02, 2026-03-15], results = [true, true]
```

Easy to read and write

```rgd
{A, B, C}
# edges
B -> A: [1, 2, 3]
C <- A: [0.1, 0.2, 0.3]
```

Object representation

```rgd
Alice: {
    affiliation = "Topgood University",
    interests = [
        "graph theory",
        "communications",
        "machine learning",
    ]
}

Bob: {
    affiliation = "Prettynice College",
    interests = [
        "human machine interaction",
        "statistical mechanics",
    ]
}

# edges
Alice -- Bob &1: {
    entry = "article",
    type  = "coauthor",
    date  = 2022-01-01,
    doi   = "https://doi.org/10.1145/359340.359342"
    abstract = """
    An encryption method is presented with the novel
    property that publicly revealing ...
    """
}
Alice -> Bob &2: {
    entry = "relationship",
    type  = "advisor",
}
```

Multiple graphs per file
```rgd
// Each graph requires a header for separation
# graph
# nodes
Alice: birthday = 1990-10-10
Bob: birthday = 1992-07-04

# edges
Alice -- Bob: relation = "spouse"

# graph
# nodes
&Alice // Equivalent to copy/pasting original definition
Charlie: birthday = 2015-01-01

# edges
Alice -> Charlie: relation = "child"

# graph
# nodes
&*  // Include *all* previously defined nodes
# edges
Bob -> Charlie: relation = "child"
```


Multigraphs

```rgd
{A, B, C}: type = "person"
A: age = 25
B: age = 30
C: age = 28

# edges
A -> B &1: type = "relation", value = "sibling"
A -> B &2: type = "event", event = { date = 2024-08-01, tag = "visit" }
B -> C   : type = "relation", value = "friends"
C <> A &1: type = "relation", value = "spouse"
C <> A &2: type = "event", event = { date = 2025-11-01, tag = "marriage" }
```

Hypergraphs!

```rgd
{A, B, C, D, E, F}
#
{A, B, C}:  role = "worker"
{D, E}:     role = "manager"
{F}:        role = "leader"
```

Directed-Multi-Hypergraphs!

```rgd
{A,B,C,D,E,F}
#
{A,B} -> {D,E} &1: operation = "intersection"
{A,B} -> {D,E} &2: [10.0, 11.0, 13.0, 14.0, 16.0, 50.0]
{B,C,D,E}: group = "friends"
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
