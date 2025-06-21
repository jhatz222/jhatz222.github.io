---
layout: page
title: "Algorithms & Data Structures"
permalink: /algorithms-data-structures
---

## Artifact: MongoDB Aggregation Pipeline Notebook

**Original:** Spring 2024 CS-340 notebook running basic aggregation queries in Python.  
**Enhancement:** Added algorithmic code and performance benchmarks.

I wrapped each MongoDB aggregation in Python `time.time()` calls to record how long the database took to return results. Then I implemented an in-memory Binary Search Tree class in Python so that, instead of dumping `(state, count)` pairs into a list and calling `sorted()`, the data is inserted into the tree and then retrieved via an in-order traversal. Finally, I printed both timings side by side to show which method is faster on this data set.

```python
# Cell 10: Binary Search Tree Implementation
class Node:
    def __init__(self, state, count):
        self.state, self.count = state, count
        self.left = self.right = None

class BST:
    def __init__(self):
        self.root = None

    def insert(self, state, count):
        def _ins(node, s, c):
            if node is None:
                return Node(s, c)
            if s < node.state:
                node.left = _ins(node.left, s, c)
            else:
                node.right = _ins(node.right, s, c)
            return node
        self.root = _ins(self.root, state, count)

    def in_order(self):
        out = []
        def _traverse(n):
            if n:
                _traverse(n.left)
                out.append((n.state, n.count))
                _traverse(n.right)
        _traverse(self.root)
        return out

# Build and traverse BST
bst = BST()
for r in results:   # results from the aggregation pipeline
    bst.insert(r['_id'], r['count'])
print("BST order:", bst.in_order())

import time
data = [(r['_id'], r['count']) for r in results]

t0 = time.time()
py_sorted = sorted(data, key=lambda x: x[0])
print("Python sort time:", time.time() - t0)

t1 = time.time()
bst2 = BST()
for st, ct in data:
    bst2.insert(st, ct)
bst_trav = bst2.in_order()
print("BST time:", time.time() - t1)
