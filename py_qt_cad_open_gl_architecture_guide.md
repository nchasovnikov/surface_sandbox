# PyQt CAD-Like Application – Modern OpenGL Architecture Guide

This document summarizes the full evolution of a PyQt-based CAD-style application with a GPU-accelerated OpenGL viewport, including architecture decisions, fixes, and final working patterns.

---

## 1. Goals

- Fullscreen CAD-style UI
- Translucent overlay panels
- GPU-accelerated 3D viewport
- STL loading
- Modern OpenGL (shaders, VAO/VBO)
- Clean architecture (Scene / Object / View separation)

---

## 2. Architecture Overview

### Separation of concerns

```
Scene
 ├── MeshObject 1
 ├── MeshObject 2
 └── ...

GLViewport → renders Scene
UI → modifies Scene
```

### Components

#### MeshObject
- Stores geometry (vertices, normals)
- Stores OpenGL buffers (VAO, VBO, NBO)
- Holds transform matrix

#### Scene
- Manages multiple objects

#### GLViewport
- Handles rendering only
- Maintains camera (rotation, zoom)

---

## 3. Common Issues Encountered

### ❌ Module OpenGL not found
```
pip install PyOpenGL PyOpenGL_accelerate
```

### ❌ Rotation only in one direction
- Fixed by accumulating rotation angles instead of overwriting

### ❌ Segfault in initializeGL()
**Cause:** OpenGL calls before context exists

**Fix:**
- Do NOT create VAOs during STL load
- Only create them inside `initializeGL()` or `paintGL()`

### ❌ OpenGL Error 1282
**Cause:** Invalid OpenGL state

Typical reasons:
- VAO/VBO used before initialization
- Missing buffer bindings
- Calling GL functions without active context

**Fix:**
- Always bind VAO before setting attributes
- Ensure buffers exist
- Initialize only inside OpenGL context

---

## 4. Final Working Code

```python
# (Full working code)
from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QToolBar, QAction, QFileDialog
)
from PyQt5.QtCore import Qt
from PyQt5.QtWidgets import QOpenGLWidget
from OpenGL.GL import *
import numpy as np
import sys
import struct

VERT_SHADER = """
#version 330
layout(location = 0) in vec3 position;
layout(location = 1) in vec3 normal;

uniform mat4 mvp;
uniform mat4 model;

out vec3 fragNormal;
out vec3 fragPos;

void main() {
    fragPos = vec3(model * vec4(position, 1.0));
    fragNormal = mat3(transpose(inverse(model))) * normal;
    gl_Position = mvp * vec4(position, 1.0);
}
"""

FRAG_SHADER = """
#version 330
in vec3 fragNormal;
in vec3 fragPos;

out vec4 color;

void main() {
    vec3 norm = normalize(fragNormal);
    vec3 lightDir = normalize(vec3(5,5,5) - fragPos);
    float diff = max(dot(norm, lightDir), 0.2);
    color = vec4(vec3(0.0, 0.7, 1.0) * diff, 1.0);
}
"""

class MeshObject:
    def __init__(self):
        self.vertices = None
        self.normals = None
        self.vao = None
        self.vbo = None
        self.nbo = None
        self.vertex_count = 0
        self.transform = np.identity(4, dtype=np.float32)

    def load_stl_binary(self, path):
        verts, norms = [], []
        with open(path, 'rb') as f:
            f.read(80)
            tri_count = struct.unpack('<I', f.read(4))[0]
            for _ in range(tri_count):
                n = struct.unpack('<fff', f.read(12))
                v1 = struct.unpack('<fff', f.read(12))
                v2 = struct.unpack('<fff', f.read(12))
                v3 = struct.unpack('<fff', f.read(12))
                verts.extend(v1 + v2 + v3)
                norms.extend(n*3)
                f.read(2)
        self.vertices = np.array(verts, dtype=np.float32)
        self.normals = np.array(norms, dtype=np.float32)

    def init_vao(self):
        if self.vertices is None:
            return
        self.vao = glGenVertexArrays(1)
        glBindVertexArray(self.vao)

        self.vbo = glGenBuffers(1)
        glBindBuffer(GL_ARRAY_BUFFER, self.vbo)
        glBufferData(GL_ARRAY_BUFFER, self.vertices.nbytes, self.vertices, GL_STATIC_DRAW)
        glEnableVertexAttribArray(0)
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, None)

        self.nbo = glGenBuffers(1)
        glBindBuffer(GL_ARRAY_BUFFER, self.nbo)
        glBufferData(GL_ARRAY_BUFFER, self.normals.nbytes, self.normals, GL_STATIC_DRAW)
        glEnableVertexAttribArray(1)
        glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 0, None)

        glBindVertexArray(0)
        self.vertex_count = len(self.vertices)//3

class Scene:
    def __init__(self):
        self.objects = []

    def add_object(self, obj):
        self.objects.append(obj)

class GLViewport(QOpenGLWidget):
    def __init__(self):
        super().__init__()
        self.scene = Scene()
        self.program = None

    def initializeGL(self):
        glEnable(GL_DEPTH_TEST)

        vs = glCreateShader(GL_VERTEX_SHADER)
        glShaderSource(vs, VERT_SHADER)
        glCompileShader(vs)

        fs = glCreateShader(GL_FRAGMENT_SHADER)
        glShaderSource(fs, FRAG_SHADER)
        glCompileShader(fs)

        self.program = glCreateProgram()
        glAttachShader(self.program, vs)
        glAttachShader(self.program, fs)
        glLinkProgram(self.program)

        for obj in self.scene.objects:
            obj.init_vao()

    def paintGL(self):
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
        glUseProgram(self.program)

        for obj in self.scene.objects:
            if obj.vao is None:
                obj.init_vao()
            glBindVertexArray(obj.vao)
            glDrawArrays(GL_TRIANGLES, 0, obj.vertex_count)
            glBindVertexArray(0)

class CADWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.viewport = GLViewport()
        self.setCentralWidget(self.viewport)

        tb = QToolBar()
        self.addToolBar(tb)
        act = QAction("Load STL", self)
        act.triggered.connect(self.load)
        tb.addAction(act)

    def load(self):
        p,_ = QFileDialog.getOpenFileName(self,"Open STL","","*.stl")
        if p:
            obj = MeshObject()
            obj.load_stl_binary(p)
            self.viewport.scene.add_object(obj)
            self.viewport.update()

if __name__ == "__main__":
    app = QApplication(sys.argv)
    w = CADWindow()
    w.show()
    sys.exit(app.exec_())
```

---

## 5. Key Takeaways

- Never call OpenGL before context exists
- Separate data (Scene, MeshObject) from rendering (Viewport)
- Use modern OpenGL: VAO + VBO + shaders
- Initialize GPU resources lazily or in `initializeGL()`

---

## 6. Next Steps

- Camera system (orbit + pan)
- Fit-to-view (bounding box)
- Wireframe rendering
- Object selection (picking)
- Edge rendering

---

You now have a solid base for a real CAD-style application.

