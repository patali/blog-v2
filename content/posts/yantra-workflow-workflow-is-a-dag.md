+++
date = 2025-11-02T07:30:00Z
disable_comments = false
tags = ["yantra", "workflow", "golang", "automation", "graph", "river queue", "rube goldberg machine"]
title = "Understanding Yantra's DAGs: Why Workflows in Yantra Are Graphs"
type = ""
draft = false
+++

```
NOTE: I asked Claude to help me distill this topic from my notes, and it did such a fine job that I hardly made any changes. I left this explanation in place because it's a succinct overview of Yantra's node execution process.
```

At its core, Yantra represents workflows as Directed Acyclic Graphs (DAGs). Let's explore exactly what that means and why this design choice is crucial .

### What Is a DAG?

- **Directed**: Data flows in one direction (node A → node B)
- **Acyclic**: No loops back to previous nodes (no infinite cycles)
- **Graph**: Nodes (tasks) connected by edges (data flow)

### Why Not Just a List?

Because real workflows aren't linear. Consider a typical DevOps automation:

```
Start → Fetch API Data → Transform
                           ├─ Send to Slack (in parallel)
                           ├─ Save to Database (in parallel)
                           └─ Generate Report
                                 └─ Conditional Check
                                       ├─ (if critical) → Page On-Call
                                       └─ (if normal) → Email Summary
```

A linear list can't represent this. You need a graph to express:
- **Parallel execution**: Slack and Database saves happen simultaneously
- **Conditional branching**: Different paths based on data
- **Fan-out/fan-in**: One node triggers multiple children, multiple nodes merge into one

### The Loop Paradox: How Can a DAG Have Loops?

Here's a brain-teaser: Yantra has a **Loop Node** that iterates over arrays. But DAGs are acyclic (no loops)! How?

The answer: **Structural DAG vs. Execution Graph**

The *workflow structure* is a DAG - the visual graph you see in the editor. But when a Loop Node executes, it **internally creates an execution subgraph** where it runs child nodes multiple times.

```
Visual DAG:
  Loop Node → HTTP Request → Accumulator

Execution Graph (3 items):
  Loop[0] → HTTP Request[0] ↘
  Loop[1] → HTTP Request[1] → Accumulator
  Loop[2] → HTTP Request[2] ↗
```

The loop is unrolled into sequential execution (parallel in future maybe?). The DAG structure is preserved - you never have an edge going backwards to create a cycle in the workflow definition itself.

This is important because:
1. **No infinite loops**: You can't accidentally create a workflow that never terminates
2. **Deterministic execution order**: Topological sort always works on a DAG
3. **Safe checkpointing**: We can always determine which nodes have completed

### How Yantra Traverses the DAG

We use **Breadth-First Search (BFS)** with dependency resolution:

```go
// Start from the "start" node
queue := []string{startNodeID}
executed := make(map[string]bool)
nodeOutputs := make(map[string]interface{})

for len(queue) > 0 {
    currentNode := queue[0]
    queue = queue[1:]
    
    // Check if all parent nodes completed
    parents := getParentNodes(currentNode)
    allParentsComplete := true
    for _, parent := range parents {
        if !executed[parent] {
            allParentsComplete = false
            break
        }
    }
    
    if !allParentsComplete {
        // Wait for parents - add back to end of queue
        queue = append(queue, currentNode)
        continue
    }
    
    // Execute the node
    input := collectInputFromParents(currentNode, nodeOutputs)
    output := executeNode(currentNode, input)
    
    nodeOutputs[currentNode] = output
    executed[currentNode] = true
    
    // Add child nodes to queue
    for _, childNode := range getChildren(currentNode) {
        if !executed[childNode] && !inQueue(childNode, queue) {
            queue = append(queue, childNode)
        }
    }
}
```

Each node is completely isolated - it receives input, processes it, produces output. Just like each component in a GStreamer pipeline.