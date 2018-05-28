Lets you add edges to a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) and be told whether this edge
introduces a [cycle](https://en.wikipedia.org/wiki/Cycle_(graph_theory)). If it would, it is not added. Useful when trying to build
an acyclic graph.

Based on [the paper](https://dl.acm.org/citation.cfm?id=1210590):

```text
A Dynamic Topological Sort Algorithm for Directed Acyclic Graphs
   DAVID J. PEARCE / PAUL H. J. KELLY
   Journal of Experimental Algorithmics (JEA)
   Volume 11, 2006, Article No. 1.7
   ACM New York, NY, USA
```

# Documentation

[See here for documentation](https://blutorange.github.io/js-incremental-cycle-detect/)

# Install

The [drill](https://docs.npmjs.com/getting-started/installing-npm-packages-locally):

```sh
npm install --save incremental-cycle-detect
```

Typings for [Typescript](https://www.typescriptlang.org/) are available (this is written in typescript!).

Use the `dist.js` or `dist.min.js` for [browser usage](http://browserify.org/) if you must.

Exposes a [global object](https://softwareengineering.stackexchange.com/questions/277279/why-are-globals-bad-in-javascript) `window.IncrementalCycleDetect` with the same methods you can when importing this lib:

```javascript
import * as IncrementalCycleDetect from "incremental-cycle-detect";
```

# Usage

The main purpose of this library is to add edges to a directed acyclic graph and be told when
that make the graph cyclic.

```javascript
const { GenericGraphAdapter } = require("incremental-cycle-detect");
const graph = new GenericGraphAdapter();
graph.addEdge(0, 1) // => true
graph.addEdge(1, 2) // => true
graph.addEdge(2, 3) // => true
graph.addEdge(3, 0) // => false because this edge introduces a cycle
// The edge (3,0) was not added.

graph.deleteEdge(2, 3);
graph.addEdge(3, 0) // => true, no cycle because we deleted edge (2,3)
```

The main algorithm is implemented by `CycleDetectorImpl`. To allow for this lib to work with different graph
data structures, it is subclassed. The subclass is called `...Adapter` responsible for storing the vertex and edge data.
For convenience, the following adapters are provided and all implement [CommonAdapter](https://blutorange.github.io/js-incremental-cycle-detect/interfaces/commonadapter.html)

- [GenericGraphAdapter](https://blutorange.github.io/js-incremental-cycle-detect/interfaces/genericgraphadapter.html): Uses `Map`s to associate data with a vertex, allowing any type of vertex. In the above example, you could use strings, booleans, objects etc. instead of numbers. Seems to perform pretty well.
- [GraphlibAdapter](https://blutorange.github.io/js-incremental-cycle-detect/classes/graphlibadapter.html): For the npm module [graphlib](https://www.npmjs.com/package/graphlib). Vertices are strings.

```javascript
const { Graph } = require("graphlib");
const graph = new GraphlibAdapter({graphlib: Graph});
graph.addEdge(0, 1) // => true
```

You can add vertices explicitly, but it is not required. They are added if they do not exist.

See the documentation linked above for all methods available.

# Performance

Incremental cycle detection performs better than checking for cycles from scratch every time you add an edge.
Tests done with [benchmark](https://www.npmjs.com/package/benchmark). Compared with running a full topological
sort with `graphlib` (via `alg.isAcyclic(graph)`) each time a vertex is added. Measured time is the time that
was needed for creating a new graph and adding `n` vertices, checking for a cycle after each edge insertion.

```
// 200 vertices, 15000 random (same for each algorithm) edges added
incremental-cycle-detection(insert 15000, RandomSource) x 36.51 ops/sec ±4.53% (59 runs sampled)
graphlib(insert15000, RandomSource) x 0.18 ops/sec ±2.78% (5 runs sampled)
```

# JavaScript environment

Some parts need `Map`. You can either

- use this lib in an enviroment that [supports these](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [polyfill](https://en.wikipedia.org/wiki/Polyfill_%28programming%29) [Map](https://www.npmjs.com/package/core-js)
- pass an implementation of Mapt to the constructor of the graph adapter. This way you don't have to [monkey patch](https://stackoverflow.com/questions/5741877/is-monkey-patching-really-that-bad):

```javascript
import * as Map from "core-js/es6/map";
const graph = new GenericGraphAdapter({mapConstructor: Map}):
```

# Use your own graph data structure

You can also use the CycleDetector (implemented by `PearceKellyDetector`) directly and
roll your own graph data structure. See the [docs](https://blutorange.github.io/js-incremental-cycle-detect/classes/pearcekellydetector.html).
Basically, you need to call the `CycleDetector` every time you add an edge or delete a vertex. Then it tells you
whether adding an edge is allowed. You can also use an existing `GraphAdapter` as the starting point.

# Build

May not to work on [Windows](https://xkcd.com/196/).

```sh
git clone https://github.com/blutorange/js-incremental-cycle-detect
cd js-incremental-cycle-detection
npm install
npm run build
```

# Change log

I use the following keywords:

- `Added` A new feature that is backwards-compatible.
- `Changed` A change that is not backwards-compatible.
- `Fixed` A bug or error that was fixed.

From newest to oldest:

- 0.1.0 Initial version.