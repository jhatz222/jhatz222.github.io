---
layout: page
title: Algorithms & Data Structures
permalink: /algorithms-data-structures
---

# Algorithms & Data Structures

**Original Artifact:** MongoDB Aggregation Pipeline Notebook from Spring 2024 that ran aggregation queries in Python
**Enhancement:** Added an in-memory Binary Search Tree with benchmarks; created MongoDB indexes to speed up queries

## Purpose of Notebook

I built a Python notebook that uses PyMongo’s aggregation framework to answer questions like:

- “How many animals are in each state?”  
- “How many facilities were founded each year?”

That worked, but it always:

1. Pulled back all documents  
2. Built a Python list of `(key, count)` pairs  
3. Ran `sorted()` over that list before printing results

In this enhancement, I decoupled sorting from Python’s built-in and:

1. Insert each `(key, count)` pair into a custom BST  
2. Walk the tree in-order to produce sorted output  
3. Time both approaches side by side so you can compare  
4. Add MongoDB indexes on `state` and `founded_year` fields so the database does as much narrowing down as possible

---

## Changes Made

- **Binary Search Tree**  
  Implemented a `BST` class in pure Python.  Instead of dumping into a list + `sorted()`, I insert into the tree and traverse it in-order.

- **Performance Benchmark**  
  Wrapped both approaches in `time.time()` calls so you can see which one is faster on the data set.

- **MongoDB Indexes**  
  Created single-field indexes on `state` and `founded_year` in the `animals` collection.  This pushes filtering/sorting work back to MongoDB and cuts down execution time dramatically.

---

## Important Code Snippets

### 1. Original Aggregation + Python Sort

```python
from pymongo import MongoClient
import time

client = MongoClient()
coll = client.shelter.animals

# count per state
start = time.time()
pipeline = [
    {"$group": {"_id": "$state", "count": {"$sum": 1}}},
    {"$sort": {"_id": 1}}
]
results = list(coll.aggregate(pipeline))
print("Python + MongoDB aggregation took:", time.time() - start)

for r in results:
    print(r["_id"], r["count"])

---

## BST Implementation

# bst.py

class Node:
    def __init__(self, key, count):
        self.key = key
        self.count = count
        self.left = None
        self.right = None

class BST:
    def __init__(self):
        self.root = None

    def _insert(self, node, key, count):
        if node is None:
            return Node(key, count)
        if key < node.key:
            node.left = self._insert(node.left, key, count)
        else:
            node.right = self._insert(node.right, key, count)
        return node

    def insert(self, key, count):
        self.root = self._insert(self.root, key, count)

    def _inorder(self, node, out):
        if not node:
            return
        self._inorder(node.left, out)
        out.append((node.key, node.count))
        self._inorder(node.right, out)

    def in_order(self):
        result = []
        self._inorder(self.root, result)
        return result

---

## Benchmarking Both Approaches

from bst import BST

# fetch unsorted pairs
pairs = [(r["_id"], r["count"]) for r in coll.aggregate([{"$group": {"_id": "$state", "count": {"$sum": 1}}}])]

# Python sort
t0 = time.time()
sorted_pairs = sorted(pairs, key=lambda x: x[0])
print("Python sorted() took:", time.time() - t0)

# BST
t1 = time.time()
tree = BST()
for k, c in pairs:
    tree.insert(k, c)
bst_pairs = tree.in_order()
print("BST in-order traversal took:", time.time() - t1)

---

## Creating MongoDB Indexes

// connect to mongo shell, then:

use shelter
db.animals.createIndex({ state:  1 })
db.animals.createIndex({ founded_year: 1 })

// re-run the aggregation pipeline and timing above to compare.

---

## Why I Made These Changes

Binary Search Tree is an O(log n) average-time insert structure; walking it in-order yields sorted output in O(n).

Python’s built-in sorted() is O(n log n) on a list, so for large data sets BST may win (and it’s instructive to compare).

Database Indexes push filtering work into MongoDB’s optimized engine, avoiding full collection scans.

---

## Reflection on Course Outcomes

Algorithmic Thinking (Outcome 2): Demonstrated custom data-structure design and complexity trade-offs.

Performance Awareness (Outcome 3): Measured and compared two strategies under real data.

Data Management (Outcome 4): Leveraged database indexing to optimize query execution.
