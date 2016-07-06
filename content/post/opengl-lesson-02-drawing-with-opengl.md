+++
title = "OpenGL Lesson 02 - Drawing with OpenGL"
date = "2012-12-12"
tags = ["opengl", "opengles", "c"]
+++
OpenGL is a primarily an C API, for drawing graphics. Implementations
and bindings exists for several languages such as Java, Python, Ruby and
Objective C. OpenGL became the standard drawing API supported by most
modern device with graphics, independent from vendor, operational
system, or if it is desktop or embedded. Of course the platform matters,
but we can split the platform dependent code from pure OpenGL.

OpenGL became a standard mainly due to its rendering pipeline, which is
trivially parallelized. This allowed the creation of specialized
hardware, the well known graphics cards. These cards became very small
and started to be practical shipping embedded devices with them. Now
high performance graphics in these devices are a reality.

On traditional desktop platforms, the usual layout of graphics card
stand as pictured below. In this scenario, moving data to and from the
card can mean a huge cost. On others platforms, such as the mobile, its
common that GPUs uses the same memory as the CPU. However, the
programmer still needs to handle this memory efficiently.

![](/img/arch.png)

For this new range of devices, the Khronos (group responsible for
standardizing OpenGL API) released an OpengGL specification focused on
embedded systems, the OpenGL ES.

In this post I would like to explain some key concepts about the OpenGL
API:

- What are the best practices for it.
- Differences between the "Desktop" version versus the ES version.

I want not to go deep in the API or its functionalities. You have other
sources that covers them better. I recommend the ones used as reference
for this post:

- [GL Programming](http://www.glprogramming.com/red/index.html)
- [Song Ho Ahn's Tutorials and Notes](http://www.songho.ca/opengl/)
- OpenGL Programming Guide, 7th edition. (a.k.a. The Red Book) or a newer edition

Hello Triangle!
---------------

![](/img/hello.png)

Enough talking, show me the code! I wrote the following code using GLUT
and OpenGL 1. GLUT is a simple toolkit to create simple OpenGL
applications. It basically opens an window with a GL context, and
handles primitive mouse and keyboard events.

```c

#include <GL/gl.h>
#include <GL/glut.h>

void display()
{
    glClear(GL_COLOR_BUFFER_BIT); // Clean up the canvas

    glBegin(GL_TRIANGLES);
    glVertex2f(-1.0f, -1.0f);
    glVertex2f( 0.0f,  1.0f);
    glVertex2f( 1.0f, -1.0f);
    glEnd();

    glFlush(); // Forces previous GL commands to be send to the GPU
}

int main(int argc, char **argv)
{
    glutInit(&argc, argv);
    glutInitWindowSize(480, 480);
    glutCreateWindow("Hello World");

    glutDisplayFunc(display);

    glutMainLoop();

    return 0;
}
```

Drawing the triagle
-------------------

In OpenGL 1 and 2, the easiest way for you to draw a triangle, is using
some form of `glVertex*`. These call must be enclosed between `glBegin`
and `glEnd`.

OpengGL uses a coordinate system where the origin is the center of the
viewport, the X axis has left to right orientation and Y axis is bottom
to top, as pictured below. By default everything between (-1, -1) and
(1, 1) is what youll be shown in the viewport. Check this
[tutorial](http://www.songho.ca/opengl/gl_transform.html) for
understanding OpengGL coordinate system and camera deeper.

![](/img/gles-cam.png)

You also need to assert what kind of primitive you are passing to
OpenGL. It accepts the primitives illustrated below with their
correspondent constants. OpenGL ES does not support polygon or quads,
you will need to assemble them yourself.

![](/img/primitives.gif)

Interleaved with the vertex position, you can add other information such
as colors, texture coordinates, normal direction. You can define other
vertex attributes for richer shaders. Shader is a piece of code that
defines how your primitives will be rendered. With them is possible to
make a lot of effects such as normal mapping, adding shadows, particles
and many more. When we choose a shading model we are using OpengGL
default shaders. Standard OpengGL defines a large set of inputs and
outputs a shader must have. OpenGL ES 2 and above does not defines what
you must enter as input for the shaders. Is up to the programmer what
will be the inputs vertex shader will have. The contract assumes that
the vertex shader will return at least a position (search for
gl\_Position) and the fragment shader a color (gl\_FragColor). Do not
worry. This will be further detailed in a following post.

Vertex Arrays
-------------

Drawing with `glVertex*` was deprecated from OpenGL 3 and beyond. In
OpenGL ES we do not have them either. This drawing method overheads of
one function call for each information entered in the pipeline. The
OpengGL comittee also wanted to disencourage the usage of this kind of
input mode. The interfere of this overhead is small for small objects,
but is not true for large ones. Another reason, for removing them
(specially on ES version), was the make OpenGL implementation lighter,
by reducing the number of internal states.

Prefere to draw using vertex arrays. Vertex arrays are arrays that each
element contains all the vertex information. The command to draw them is
` glDrawArrays `. Indexes can be specified to reuse the vertex
definition by using `glDrawElements`. A good reference for this subject
is [this one](http://www.songho.ca/opengl/gl_vertexarray.html). As this
is the standard way when using OpenGL ES 2, I will give an example.

Example of Vertex Arrays
------------------------

To draw a square you must first define the schema of each vertex. Here,
each vertex has a position (3 floats) and a color (3 floats for R, G and
B color channels). In C I like to define a struct to improve
readability:

```c
struct vertex_t {
    GLfloat position[3];
    GLfloat color[3];
};

void display()
{
    struct vertex_t vertex_data[] = {
         {{-1.0f, -1.0f, 0.0f}, {1.0f, 0.0f, 0.0f}}, // red bottom left
         {{-1.0f,  1.0f, 0.0f}, {0.0f, 1.0f, 0.0f}}, // green top left 
         {{ 1.0f, -1.0f, 0.0f}, {0.0f, 0.0f, 1.0f}}, // blue bottom right
         {{ 1.0f,  1.0f, 0.0f}, {1.0f, 1.0f, 1.0f}}, // white top right
    };
    
    // ...
}
```

It is possible to have different arrays for color and position, but for
speeding up the shader execution is recommended to keep information
about the same vertex contiguous to use memory locality.

```c
glVertexAttribPointer(
    position_attribute_location, // attribute description (depends on the shader)
    3, // size of the information (3 coordinates in this case)
    GL_FLOAT, // type of the information
    GL_FALSE, // if the value will be normalized (for vectors)
    sizeof(struct vertex_t), // stride of the memory vector
    &vertex_data[0].position // initial address
);
// asserting that position will be used by the shader
glEnableVertexAttribArray(position_attribute_location); 

glVertexAttribPointer(
    color_attribute_location,
    3,
    GL_FLOAT,
    GL_FALSE,
    sizeof(struct vertex_t),
    &vertex_data[0].color
);
// asserting that color will be used by the shader
glEnableVertexAttribArray(color_attribute_location);

// Draw a triangle strip vertex from the current attribute pointers
// starting on index 0 and using 4 elements
glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
```

This gives us the following result:

![](/img/hello2.png)

Notice that the colors for each vertex stands as the ones we gave, but
all the region in between has the color calculated as the interpolation
of the vertex colors. The rasterizer is the responsible for doing this.

You could also use indexes to access the vertex. For bigger elements it
may be a better solution, specially for 3D objects, because you can
share vertex that appears in multiple polygons of an object. Using index
would speed up the memory transfer and usage for these objects.

```c
GLubyte indices[] = {
    0, 1, 2, // first triangle
    1, 2, 3  // second triangle
};

glDrawElements(
    GL_TRIANGLES, // not a strip in this case
    6, // number of indexes
    GL_UNSIGNED_BYTE, // type of the indexes
    indices // a pointer to indexes themselves
);
```

The previous code does it for a trivial shader set (called a program). I
will not explain how to use it here because is out of the scope for this
lesson. On regular OpenGL versions you could use `glVertexPointer` and
`glColorPointer`. I will left it as an exercise for you.

Vertex Buffer Objects
---------------------

Buffers are objects that stores vertex information in GPU memory. It is
a must for improving the performance when drawing large objects. In
heavy application such as games or CADs, is good to remove the overhead
of from sending vertex data from regular memory to graphics card memory
by pre loading the vertex data in a buffer. The code below show how can
you upload the vertex data to a buffer.

```c
GLuint bufferId;

// here you get handlers for GPU buffers, in this case, only one
glGenBuffers(1, &bufferId);

// asserts that you are using the buffer represented by bufferId
// as the current ARRAY_BUFFER
glBindBuffer(GL_ARRAY_BUFFER, bufferId);

glBufferData(
    GL_ARRAY_BUFFER, // the data is uploaded to current array buffer
    sizeof(vertex_data), // number of bytes of the total array
    vertex_data, // the pointer to the data
    GL_STATIC_DRAW, // hint of how the buffer will be used, in this case, data will not change
)
```

To draw the buffer content, you must use `glVertexAttribPointer` passing
the buffer offset instead of the ` vertex_data ` address. OpenGL will
notice that a buffer is bound and will use it.

```c
glBindBuffer(GL_ARRAY_BUFFER, bufferId); // Bind whenever you will use it

glVertexAttribPointer(
    position_attribute_location, // attribute description (depends on the shader)
    3, // size of the information (3 coordinates in this case)
    GL_FLOAT, // type of the information
    GL_FALSE, // if the value will be normalized (for vectors)
    sizeof(struct vertex_t), // stride of the vertex buffer data
    0 // offset at buffer
);
// asserting that position will be used by the shader
glEnableVertexAttribArray(position_attribute_location); 

glVertexAttribPointer(
    color_attribute_location,
    3,
    GL_FLOAT,
    GL_FALSE,
    sizeof(struct vertex_t),
    3 // offset at buffer
);
// asserting that color will be used by the shader
glEnableVertexAttribArray(color_attribute_location);

// Draw a triangle strip vertex from the current attribute pointers
// starting on index 0 and using 4 elements
glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);

// Unbinds the buffer
glBindBuffer(GL_ARRAY_BUFFER, 0);
```

Remarks about OpengGL ES 2
--------------------------

OpenGL ES 2 does not have the model-view matrix, responsible for setting
the camera view, nor the matrix stack. All you have is the X axis [-1.0,
-1.0] to Y axis [1.0, 1.0] region which will be mapped to the viewport.
If you need those features (you will for most 3d applications) you will
have to handle them inside your application code. You will have to do it
yourself by exporting a model-view in the vertex shader as an uniform
variable. For a theorical background on it, check my [previous
lesson](http://codecereal.blogspot.com.br/2012/11/opengl-lesson-01-pre-opengl.html).
A great tutorial on how to play with the camera, transformations and
model coordinates is
[this](http://www.songho.ca/opengl/gl_transform.html). It explains a bit
how it works under the hood for regular OpenGL. Its worthy to take a
look.

While hacking some examples I have come to some weird behaviour in
OpenGL ES 2.0 for the Raspberry Pi. It might be bugs or mine
misinterpretation.

- `glUniformMatrix4fv` translate parameter *must* be false. It simply
    does not work otherwise.
- ` glDrawElements ` did not worked with unsigned int.
