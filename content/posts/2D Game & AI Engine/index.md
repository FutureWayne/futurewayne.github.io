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

> ![kinematic](https://github.com/UtahGameAIClass/assignment-1-movement-ai-FutureWayne/assets/39150337/ba5827c4-2492-45bd-861b-7de5d7fc43ae)

---

### Seek Behavior
The `Seek` function drives AI (boids) directly towards a target using maximum force. Without proper drag handling, this can cause overshooting and spiraling effects.

> _**Image Placeholder:** Seek behavior demo_

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

#### 2. **Physics-Based Approach**
- Focuses on acceleration and deceleration using physics principles.
- Calculates braking distance dynamically.

**Pros:**
- Adaptive to different rigidbody settings.
- Maximizes efficiency with constant max acceleration/deceleration.

**Cons:**
- Requires careful drag handling.
- More edge cases to manage.

> _**Image Placeholder:** Comparison of Arrival Approaches_

---

### Wandering Behavior
Implemented using a "wander circle" and random binomial targeting, with both velocity matching and physics-based orientation control for smooth rotation.

> _**Image Placeholder:** Wander behavior visualization_

---

### Flocking Behavior
Classic flocking combines:
1. Cohesion
2. Alignment
3. Separation

I optimized this by delegating flocking to a `Seek` behavior towards a calculated position. Additionally, I introduced a **leader-following system** to maintain group coherence.

> _**Image Placeholder:** Flocking with leader-following_

---

### Tuning with OpenFrameworks GUI
To streamline testing, I integrated sliders and buttons via OpenFrameworks GUI for real-time adjustments of parameters like max speed, acceleration, and drag.

> _**Image Placeholder:** Screenshot of OpenFrameworks GUI controls_

---

## 🧭 Pathfinding Algorithms: Dijkstra & A* in Action

Beyond movement behaviors, efficient navigation is key. I implemented **Dijkstra** and **A\*** algorithms across various graph types, optimizing data structures and heuristics for performance.

---

### Graph Designs

#### 1. **Valorant-Inspired Map**
A hand-crafted graph replicating tactical routes from a Valorant map, with weights reflecting both distance and encounter probability.

> _**Image Placeholder:** Graph visualization of Valorant map_

---

#### 2. **Very Big™ Graph**
A procedurally generated graph with 2000 nodes to stress-test algorithm scalability. Optimized using spatial partitioning to reduce runtime complexity during node generation.

> _**Image Placeholder:** Visualization of large random graph_

---

#### 3. **Indoor Obstacle Graph**
Simulates an indoor environment using a tile-based system, parsing walkable areas and obstacles from a text file.

> _**Image Placeholder:** Grid graph with obstacles representation_

---

### Optimizing the Open List: Custom Data Structure
Standard structures like `std::priority_queue` had limitations, so I designed a hybrid:
- **Priority Queue** for ordered retrieval.
- **Hash Map** for fast membership checks and cost updates.

This approach eliminated costly operations like direct element adjustments while keeping the algorithm efficient.

> _**Image Placeholder:** Diagram of custom Open List structure_

---

### Heuristic Strategies in A\*

- **Euclidean Heuristic:** Accurate and admissible for realistic maps.
- **Squared Distance Heuristic:** Faster but inadmissible due to overestimation.

Balancing speed and accuracy was critical, especially when choosing heuristics based on the use case.

> _**Image Placeholder:** Heuristic impact comparison_

---

### Algorithm Performance Insights

Running both algorithms on the Very Big™ Graph revealed key differences:

- **Dijkstra:** Uniform, radial exploration.
- **A\*:** Targeted, efficient pathfinding thanks to heuristics.

> _**Image Placeholder:** Side-by-side comparison of Dijkstra vs A\* node expansions_

---

### Heuristic Impact Recap
- **Overestimating Heuristics:** Faster but risks suboptimal paths.
- **Underestimating Heuristics:** Guarantees optimal paths but at a higher computational cost.

Choosing the right heuristic depends on whether speed or path accuracy is prioritized.

---

## 🚀 Conclusion

This project bridged physics-based AI movement with algorithmic navigation, offering a robust foundation for dynamic and intelligent agent behaviors in games or simulations. From fine-tuning steering mechanics to crafting scalable pathfinding systems, I gained hands-on experience balancing realism, efficiency, and computational constraints.

Stay tuned for future updates where I’ll integrate these systems into more complex AI behaviors!

> _**Image Placeholder:** Final showcase of AI navigating with both steering and pathfinding_

---

_Thanks for reading! If you have any questions or suggestions, feel free to reach out or check out the repo [here](#)._ 
