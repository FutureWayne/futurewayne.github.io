---
title: "Developing Smarter AI: Steering Behaviors and Pathfinding Algorithms"
summary: "In this blog, I’ll walk through my journey of developing AI behaviors, focusing on dynamic steering systems and efficient pathfinding algorithms. From crafting lifelike boid movements to optimizing search strategies in complex environments, this project deepened my understanding of both kinematic motion and algorithmic problem-solving.."
categories: ["Post","Blog",]
tags: ["post","AI","Pathfinding"]
#externalUrl: ""
#showSummary: true
date: 2024-01-04
draft: false
---



---

## 🏎️ Dynamic Steering: Kinematic Motion for AI

Creating believable AI movement requires more than just moving from point A to B. I implemented several steering behaviors inspired by boid algorithms, focusing on flexibility and realism.





### Kinematic Behavior

The AI (boid) changes its velocity and orientation instantly.

> ![img](https://github-production-user-asset-6210df.s3.amazonaws.com/39150337/306135101-ba5827c4-2492-45bd-861b-7de5d7fc43ae.gif?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20250428%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250428T102254Z&X-Amz-Expires=300&X-Amz-Signature=fbf85dd0c5e1bdcb296f81788a810d877b9fdf0e6a87f21439bf6c93003286ea&X-Amz-SignedHeaders=host)

---

### Seek Behavior
The `Seek` function drives AI (boids) directly towards a target using maximum force. Without proper drag handling, this can cause overshooting and spiraling effects.

> ![Kinematic Behavior](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\Kinematic Behavior.gif)

---

### Arrive Behavior: Two Approaches

I explored two distinct methods to refine how AI arrives smoothly at a target:

#### 1. **Velocity Matching Approach**
- Defines max speed, slow radius, and arrival radius.
- Smooth deceleration as AI nears the target by interpolating velocity.
  

**Pros:**
- Smooth transitions.
- Drag-independent.

**Cons:**
- Fixed slow radius limits flexibility.
- Potential inefficiency in certain scenarios.

> ![Velocity Matching Approach](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\Velocity Matching Approach.gif)

---

#### 2. **Physics-Based Approach**
- Focuses on acceleration and deceleration using physics principles.
- Calculates braking distance dynamically.

**Pros:**
- Adaptive to different rigidbody settings.
- Maximizes efficiency with constant max acceleration/deceleration.

**Cons:**
- Requires careful drag handling.
- More edge cases to manage.

> _![Physics-Based Approach](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\Physics-Based Approach.gif)_

---

### Wandering Behavior
Implemented using a "wander circle" and random binomial targeting, with both velocity matching and physics-based orientation control for smooth rotation.

> ![Wandering Behavior](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\Wandering Behavior.gif)

---

### Flocking Behavior
Classic flocking combines:
1. Cohesion
2. Alignment
3. Separation

I optimized this by delegating flocking to a `Seek` behavior towards a calculated position. Additionally, I introduced a **leader-following system** to maintain group coherence.

> ![Flocking Behavior](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\Flocking Behavior.gif)

---

### Tuning with OpenFrameworks GUI
To streamline testing, I integrated sliders and buttons via OpenFrameworks GUI for real-time adjustments of parameters like max speed, acceleration, and drag.



---

## 🧭 Pathfinding Algorithms: Dijkstra & A* in Action

Beyond movement behaviors, efficient navigation is key. I implemented **Dijkstra** and **A\*** algorithms across various graph types, optimizing data structures and heuristics for performance.

---

### Graph Designs

### 1️⃣ Meaningful Map
The first graph I developed represents a familiar environment—specifically, one of my favorite maps from the shooter game **Valorant**. This graph schematically outlines all possible routes from the defense spawn to the attack spawn, comprising **38 nodes** and **53 edges**.

> ![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428042651591.png)

To manage this data efficiently, I used a **JSON parser library**, allowing nodes and edges to be read from a single JSON file. Each edge is weighted by: 

`weight = distance × coefficient`

The coefficient reflects the likelihood of encountering an opponent along the path—higher values indicate higher risk.


> ![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428042859474.png)
>
> ![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428042902970.png)

---

### 2️⃣ Very Big™ Graph
The second graph is a **large-scale**, randomly generated structure aimed at testing algorithm performance under heavy load. It contains **2000 nodes**, adhering to these generation rules:
- Nodes are randomly placed with a **minimum spacing**.
- Each node connects to nearby neighbors, max **5 outgoing connections**.

> _**Image Placeholder:** Visualization of Very Big™ graph_

#### ⚙️ Generation Algorithm:
1. Start at a central point.
2. Generate new nodes based on random angles/distances.
3. Ensure minimum spacing—retry if necessary.
4. Connect nodes to nearest neighbors (up to 5).

#### 🚨 Performance Optimization:
- **Spatial Partitioning:** The space is divided into cells based on minimum spacing, reducing distance check overhead.
- **Queue System:** Nodes expand outward; if a node fails to generate new nodes after several attempts, it's dequeued.

This approach ensures coherent structure while keeping runtime generation efficient.

---

> _![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428043218487.png)_

---

### 3️⃣ Indoor Environments with Obstacles
This graph simulates indoor navigation using a **tiled system**, where nodes represent walkable areas and obstacles are defined in a text file using symbols:

- `*` = Walkable Node
- Other symbols = Obstacles

> _![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428043228014.png)_

Nodes are connected to immediate neighbors, with weights based on Euclidean distance. Nodes adjacent to obstacles are marked inaccessible to prevent collisions.

> _![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428043303326.png)_

---

## ⚡ Pathfinding Algorithm

### Open List Data Structure
The **Open List** is critical for both Dijkstra and A\*, supporting:
- Retrieval & removal of the best node
- Membership checks
- Insertion
- Cost adjustments (Decrease Key)

These operations are listed in order of their expected frequency of occurrence during the algorithm's execution. Below is an analysis of various data structures' suitability for the **Open List**, focusing on their time complexity for each operation:

1. **Unsorted Array:**

-  **Retrieval and removal of the best element, Membership check, and Adjustment of element values:** Requires iterating through the entire array, resulting in a time complexity of O(N).

-  **Insertion of new elements:** Can be done in constant time, O(1), as it involves adding the element at the end of the array.

---

2. **Sorted Array (By Key):**

- **Membership check and Adjustment of element values:** Can leverage binary search, leading to a time complexity of O(logN).

- **Retrieval and removal of the best element and Insertion of new elements:** Due to the need to maintain the sorted order, these operations incur a time complexity of O(N) as they may require shifting elements.

---

3. **Sorted Array (By Priority):**

- **Retrieval of the best element:** Is straightforward as the best element is always at the beginning, resulting in O(1) time complexity.

- **Membership check, Insertion of new elements, and Adjustment of element values:** Require scanning the entire array, leading to a time complexity of O(N).

---

4. **Hash Table:**

- **Membership check, Insertion of new elements, and Adjustment of element values:** Typically operate in constant time, O(1), due to the direct access nature of hash tables.

- **Retrieval and removal of the best element:** Requires iterating over the entire table to identify the best element, resulting in O(N) time complexity.

---

5. **Binary Heap:**

- **Adding new elements, Retrieval and removal of the best element:** Benefit from the heap's structure, allowing these operations to be performed in O(logN) time. Though Retrieval of the best element will only cost O(1), but after remove the element the whole heap must reconstruct itself.

- **Membership check:** Is less efficient, requiring a full traversal of the heap, thus having a time complexity of O(N).

---

#### ⏱️ Data Structure Comparison

| **Data Structure**     | **Retrieval & Removal** | **Membership** | **Insertion** | **Adjustment (Decrease Key)** |
|------------------------|-------------------------|----------------|---------------|-------------------------------|
| Unsorted Array         | O(N)                    | O(N)           | O(1)          | O(N)                          |
| Sorted Array (By Key)  | O(N)                    | O(logN)        | O(N)          | O(logN)                       |
| Sorted Array (Priority)| O(1)                    | O(N)           | O(N)          | O(N)                          |
| Hash Table             | O(N)                    | O(1)           | O(1)          | O(1)                          |
| Binary Heap            | O(logN)                 | O(N)           | O(logN)       | O(logN)                       |

Although **binary heap** (e.g., `std::priority_queue`) is efficient for retrieval, it lacks fast membership checks and element updates.

---

### 🛠️ Custom Hybrid Data Structure
To overcome these limitations, I designed a structure combining:
- A **Priority Queue** for ordering nodes.
- A **Hash Map** for quick membership and cost tracking.

#### How It Works:
- On insertion, check the hash map for existing nodes with higher costs.
- When popping, validate the node against the hash map.
- Outdated nodes are lazily discarded.

This ensures efficient retrieval, insertion, and cost management without needing direct priority adjustments.

---

## 🎯 Heuristics

### 📏 Euclidean Heuristic
- Uses straight-line distance.
- **Admissible** and **consistent** in both weighted graphs and grid-based layouts.
- Perfect for scenarios where edge weights correlate with distance.

---

### 📐 Squared Distance Heuristic
- Uses squared Euclidean distance: heuristic = dx² + dy²


- **Not admissible**—it overestimates costs.
- Leads to faster computation but risks invalid paths due to inconsistency.

---

## 📊 Algorithm Analysis

### Performance on Very Big™ Graph
Starting from the center:

- **Dijkstra:** Explores uniformly outward, forming a circular search pattern.
- **A\*:** Focused search guided by heuristics, reducing unnecessary node expansions.

> ![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428044302505.png)
>
> ![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428044306985.png)

---

### Heuristic Impact on A\*

#### Overestimating Heuristic:
- Reduces nodes explored.
- Faster execution.
- May skip optimal paths.

#### Underestimating Heuristic:
- Explores more nodes.
- Guarantees shortest path.
- Higher computational cost.

> ![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428044318166.png)
>
> ![](C:\Users\futur\Code\futurewayne.github.io\content\posts\2D Game & AI Engine\image-20250428044325600.png)

---

## 📝 Conclusion
In this project, I explored both classic and advanced aspects of pathfinding:

- Implemented **Dijkstra** and **A\*** across diverse graph types.
- Designed a **custom data structure** to optimize Open List operations.
- Analyzed heuristic strategies to balance speed vs accuracy.

This comprehensive study sharpened my understanding of algorithm efficiency, data structure design, and practical AI navigation challenges in both game-like and abstract environments.

---
