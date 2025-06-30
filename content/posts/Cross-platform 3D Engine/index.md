---
title: "Cross-platform Game Engine"
summary: "A 3D game engine supporting Direct 3D and OpenGL across multiple architectures."
categories: ["Post","Blog"]
tags: ["post","C++"]
showSummary: true
date: 2025-01-01
draft: false
---



# 🛠️ Custom 3D Engine

> **A lightweight, cross‑platform rendering engine with a Lua‑based asset pipeline and Maya export tools.**

  

## ✨ Features

| Category            | Highlights                                                   |
| ------------------- | ------------------------------------------------------------ |
| **Rendering**       | • Direct3D 11 & OpenGL 4.x back‑ends• x86 & x64 builds• Unified `cRenderer` interface |
| **Asset Pipeline**  | • Human‑readable **Lua** meshes for debugging & VCS• Compact **binary** meshes for runtime‑speed• 1‑click conversion via `MeshBuilder` |
| **Authoring Tools** | • **MayaMeshExporter** plugin (C++)• Vertex‑animation export (mesh + per‑frame positions)• Automatic Lua → binary serialization |
| **Runtime Systems** | • Vertex‑animation player• Bounding‑Volume Hierarchy (**BVH**) collision framework |
| **Sample Game**     | • Flappy Bird remake with animated bird & BVH pipes          |

## Cross‑Platform Rendering

The engine isolates API‑specific code behind a **single interface** declared in cRenderer.h.

```
// cRenderer.h
struct cRenderer
{
    virtual void Clear( float r, float g, float b, float a ) = 0;
    virtual void SwapBuffers() = 0;
    /* ... */
};
```

| File                | Purpose                    |
| ------------------- | -------------------------- |
| `cRenderer.d3d.cpp` | Direct3D 11 implementation |
| `cRenderer.gl.cpp`  | OpenGL 4.x implementation  |

> **Pointer size:** 4 bytes on x86, 8 bytes on x64. The engine pads/aligns data structures accordingly to avoid UB across builds.

## Mesh Representation

### Human‑Readable (Lua)

Pros: drag‑and‑drop into Git, easy diffs, quick hot‑fixes.

```
return
{
    vertexCount = 4,
    vertices =
    {
        { x = -1.5, y = -1.0, z = -1.5 },
        { x =  1.5, y = -1.0, z = -1.5 },
        { x = -1.5, y = -1.0, z =  1.5 },
        { x =  1.5, y = -1.0, z =  1.5 },
    },
    indices = { 0,1,2, 2,1,3 }
}
```

### Binary Format

For shipping builds the Lua is converted to a tight binary blob by **MeshBuilder**.

```
04 00                      -- Vertex Count
00 00 C0 BF 00 00 80 BF ... -- Vertex 0 (float‑packed)
06 00                      -- Index Count
00 00 01 00 02 00 ...       -- Indices
```

*Same bytes on both APIs; the renderer fixes vertex winding at runtime.*

| Format | Strength         | Typical Use            |
| ------ | ---------------- | ---------------------- |
| Lua    | Readability, VCS | Prototyping, debugging |
| Binary | Load speed, size | Production, consoles   |

## Maya Export Workflow

> **Goal:** one‑click export → play‑in‑engine vertex animation.

1. **Author** animation in Maya.
2. **Export** with **MayaMeshExporter** ➜ *.lua (mesh + per‑frame positions).
3. **Build** assets: `MeshBuilder` converts Lua → binary.
4. **Run** the engine – the `Graphics` module deserializes and streams frames to the GPU.

```
graph TD;
  Maya-->|MayaMeshExporter|Lua;
  Lua-->|MeshBuilder|Binary;
  Binary-->|Loader|Engine;
```

| Issue                          | Fix                                              |
| ------------------------------ | ------------------------------------------------ |
| Huge Maya API surface          | Focused on core nodes, referenced sample plugins |
| Balancing readability vs. perf | Dual‑format pipeline (Lua + binary)              |

## Sample Project – Flappy Bird

- **Animated Bird:** exported via the Maya pipeline.
- **Collision:** pipes + bird use a 2‑level **BVH** for fast AABB tests.
- **Controls:** high‑frequency input loop (fixed Δt) for deterministic behaviour.

*Takeaway: the modular engine allowed gameplay code with zero changes to core systems.*



### 🎬 Video Showcase

[![Watch the video](https://img.youtube.com/vi/rkJCvjlSofk/hqdefault.jpg)](https://www.youtube.com/shorts/rkJCvjlSofk)



## Building & Running

```
# Windows (MSVC)
cmake -B build -A x64 -DENGINE_D3D=ON
cmake --build build --config Release

# Linux / macOS (Clang + OpenGL)
cmake -B build -DENGINE_GL=ON
cmake --build build --config Release
```

Run the **AssetBuilder** target once to convert Lua → binary before launching the demo.

---

