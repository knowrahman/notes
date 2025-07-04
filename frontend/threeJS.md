
### **STAGE 1: Beginner -- Foundations**

#### 1.1 Introduction to WebGL and Three.js

-   **Concept**: What is WebGL? How does Three.js simplify it?

-   **Code Example**:

    ```
    import * as THREE from 'three';
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    ```

-   **Mini Challenge**: Set up a scene with a single cube and rotate it.

-   **Best Practice**: Always check WebGL compatibility. Use `requestAnimationFrame` for animations.

-   **Real-World Use**: 3D landing page intro or loading spinner in a product site.

-   **Quiz**:

    1.  What is the role of a `camera` in Three.js?

    2.  Why use `requestAnimationFrame()` over `setInterval()`?

    3.  What is a renderer and what does it output?

-   **Practical Tasks**:

    1.  Set up a basic scene with a cube and animate it.

    2.  Resize the canvas when the browser window changes.

* * * * *

#### 1.2 Scene, Camera, and Renderer

#### 1.3 Geometries and Materials

#### 1.4 Meshes and Adding Objects

#### 1.5 Lights and Shadows

#### 1.6 Animation Loop and Controls (OrbitControls)

#### 1.7 Basic Textures and Materials

> Each of these will follow the same format (explanation + code + exercise + best practices + real-world use + quiz + project guidance).

* * * * *

### **STAGE 2: Intermediate -- Interactive and Dynamic Scenes**

#### 2.1 Loading Models (GLTF, OBJ)

#### 2.2 Adding GUI Controls (dat.GUI, Leva)

#### 2.3 Texture Mapping (UVs, Bump, Normal, AO)

#### 2.4 Raycasting and Mouse Interactions

#### 2.5 Lights: Ambient, Spot, Directional, HDRI

#### 2.6 Responsive Design for 3D

#### 2.7 React Three Fiber (R3F) Basics

* * * * *

### **STAGE 3: Advanced -- Customization and Performance**

#### 3.1 Writing Custom Shaders (GLSL)

#### 3.2 Post-Processing Effects (Bloom, Depth of Field)

#### 3.3 Performance Optimization (Levels of Detail, Instancing)

#### 3.4 Physics Integration (Cannon.js, Rapier)

#### 3.5 VR and AR Setup (WebXR)

#### 3.6 Deploying Three.js Projects

#### 3.7 Full Project Integration (3D Portfolio or Product Viewer)

* * * * *



> **WebGL → Shaders → Vertex & Fragment Shaders → GLSL ES → WebGPU**

* * * * *

**1\. WebGL -- What It Really Is**
---------------------------------

### What is WebGL?

WebGL (**Web Graphics Library**) is a **low-level JavaScript API** that lets you draw 3D graphics directly on an HTML `<canvas>` using the GPU --- **no plugins** needed.

### What It Does:

-   Gives you direct access to your GPU.

-   Lets you draw **triangles**, which is how all 3D graphics are made.

-   You control how each **vertex** and **pixel** is drawn using **shaders**.

### Why It's Hard:

-   There's **no scene**, **no objects**, **no camera** --- everything must be manually programmed.

-   That's why libraries like **Three.js** exist: they simplify WebGL.

* * * * *

**2\. Shaders -- What Are They?**
--------------------------------

A **shader** is a program that runs on your **GPU**, written in **GLSL ES**.

-   GPU processes **vertices** and **fragments** (pixels).

-   You write two shaders for every object:

    1.  **Vertex Shader** → moves geometry around.

    2.  **Fragment Shader** → colors pixels on the screen.

These are **compiled and run per frame**, for every vertex and pixel.

* * * * *

**3\. Vertex Shader -- Think Geometry Transformer**
--------------------------------------------------

A **vertex shader** is a program that runs **once per vertex**.

### What It Does:

-   Takes your 3D coordinates and transforms them to 2D screen space.

-   Can also:

    -   Animate vertices

    -   Pass attributes (like color, UV, normals) to the fragment shader

### Sample Vertex Shader (GLSL ES):

```
attribute vec3 position;

uniform mat4 projectionMatrix;
uniform mat4 modelViewMatrix;

void main() {
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}

```

**Explanation:**

-   `attribute vec3 position`: position of each vertex (input).

-   `projectionMatrix * modelViewMatrix`: transforms 3D → screen space.

-   `gl_Position`: special built-in variable. This is where the vertex ends up.

* * * * *

**4\. Fragment Shader -- Think Pixel Painter**
---------------------------------------------

A **fragment shader** runs **once per pixel** on the object's surface.

### What It Does:

-   Outputs a `gl_FragColor` (the color of the pixel).

-   You can do:

    -   Simple colors

    -   Texture mapping

    -   Lighting effects

    -   Procedural effects

### Sample Fragment Shader:

```
void main() {
  gl_FragColor = vec4(1.0, 0.2, 0.5, 1.0); // RGBA Pink
}

```

**Output:** Every pixel of your shape will be colored pink.

* * * * *

**5\. GLSL ES -- The Shader Language**
-------------------------------------

**GLSL ES (OpenGL Shading Language for Embedded Systems)** is a strict C-style language with GPU-specific data types.

### Common Data Types:

| Type | Meaning |
| --- | --- |
| `vec2` | 2D vector (e.g., UV coords) |
| `vec3` | 3D vector (e.g., RGB, position) |
| `vec4` | 4D vector (e.g., RGBA or homogeneous coordinates) |
| `mat4` | 4x4 matrix (used for transforms) |

### GLSL Rules:

-   No dynamic memory.

-   No console logs (you can't debug shaders like JS).

-   Must define `main()` function.

* * * * *

**6\. WebGPU -- The Next Generation**
------------------------------------

### What is WebGPU?

**WebGPU** is a brand-new web API designed to **replace WebGL**. It's built to:

-   Provide more control over the GPU.

-   Match modern graphics APIs like **Vulkan**, **Metal**, and **DirectX 12**.

-   Be more efficient and flexible for complex 3D or compute workloads.

### Key Differences from WebGL:

| Feature | WebGL | WebGPU |
| --- | --- | --- |
| Abstraction | High (OpenGL style) | Low (closer to hardware) |
| Shader Language | GLSL ES | WGSL (WebGPU Shader Language) |
| Performance | Good | Better (multi-threaded, compute shaders) |
| Adoption | Widely supported | Gaining traction (Chromium, Safari Tech Preview) |

* * * * *

**In Practice: How It Connects in Three.js**
--------------------------------------------

In Three.js:

-   You don't write raw WebGL → you use a **scene graph**.

-   But Three.js still compiles to **WebGL shaders** behind the scenes.

-   You can inject your own shaders using:

    -   `ShaderMaterial`: when you want full control over shaders.

    -   `RawShaderMaterial`: even lower-level.

    -   `onBeforeCompile`: to tweak built-in materials.

* * * * *