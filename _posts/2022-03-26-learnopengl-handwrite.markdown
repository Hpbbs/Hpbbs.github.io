---
layout: post
title: "LearnOpengl 摘抄"
published: false
categories: [computer]
tags: [computer graphics]
---

# LearnOpenGL 摘抄

学习OpenGl的视角：

- 当作api集合，函数库来学习；函数库的视角
- 计算机图形学的数学视角

## 1. Introduction

### 1.1 Prerequisites

- Therefore a decent knowledge of the `C++` programming language is required for these chapters.
- we will be using some math (linear algebra, geometry, and trigonometry) along the way and I will try to explain all the required concepts of the math required.

## 2. Getting Started

### 1. OpenGL concept

#### 1.1 OpenGL Layer

**OpenGL** is mainly considered an API (an Application Programming Interface) that provides us with a large set of functions that we can use to manipulate graphics and images

OpenGL by itself is not an API, but merely a specification, developed and maintained by the [Khronos Group](http://www.khronos.org/).

OpenGL : API and Specification : actual developed versions of OpenGL are allowed to have different implementations

**actual OpenGL libraries**:are usually the graphics card manufacturers. Each graphics card that you buy supports specific versions of OpenGL which are the versions of OpenGL developed specifically for that card (series).

**this is usually solved by updating your video card drivers**



OpenGL(Specifications) --- OpenGLsupporLibray --- Video Card Drivers

#### 1.2 Core-profile vs Immediate mode

早期：**immdiate mode**： (often referred to as the fixed function pipeline) 就是固定的编程管线：不够自由（性能上不够efficient）但是简单易用，The immediate mode is really easy to use and understand, but it is also extremely inefficient.

For that reason the specification started to deprecate immediate mode functionality from version 3.2 onwards and started motivating developers to develop in OpenGL's **core-profile mode**, which is a division of OpenGL's specification that removed all old deprecated functionality.

**core-profile mode**：flexibility efficiency,most importantly: a much better understanding of graphics programming.

**About Version and compatible**

this book at core-profile OpenGL version 3.3: 3.3 是API固化的版本

All future versions of OpenGL starting from 3.3 add extra useful features to OpenGL without changing OpenGL's core mechanics; the newer versions just introduce slightly more efficient or more useful ways to accomplish the same tasks. The result is that all concepts and techniques remain the same over the modern OpenGL versions so it is perfectly valid to learn OpenGL 3.3.

When using functionality from the most recent version of OpenGL, only the most modern graphics cards will be able to run your application. This is often why most developers generally target lower versions of OpenGL and optionally enable higher version functionality.

#### 1.3 Extension ： 在固化的接口提供灵活性的措施

simply by checking if the extension is supported by the graphics card. Often, when an extension is popular or very useful it eventually becomes part of future OpenGL versions.

extension 需要先检查后使用

#### 1.4 StateMachine --- OpenGL context

**OpenGL is by itself a large state machine**: a collection of variables that define how OpenGL should currently operate. The state of OpenGL is commonly referred to as the **OpenGL context**. When using OpenGL, we often change its state by setting some options, manipulating some buffers and then render using the current context.

**As long as you keep in mind that OpenGL is basically one large state machine, most of its functionality will make more sense.**

#### 1.5 Objects --- C在OOP的映射

The OpenGL libraries are written in C and allows for many derivations in other languages, but in its core it remains a C-library. Since many of C's language-constructs do not translate that well to other higher-level languages, OpenGL was developed with several abstractions in mind. One of those abstractions are objects in OpenGL

An **object** in OpenGL is a collection of options that represents a subset of OpenGL's state

**The great thing about using these objects** is that we can define more than one object in our application, set their options and whenever we start an operation that uses OpenGL's state, we bind the object with our preferred settings. 

- [opengl.org](https://www.opengl.org/): official website of OpenGL.
- [OpenGL registry](https://www.opengl.org/registry/): hosts the OpenGL specifications and extensions for all OpenGL versions.

### 2. Creating a window

The first thing we need to do before we start creating stunning graphics is to create an OpenGL context and an application window to draw in

However, those operations are specific per operating system and **OpenGL purposefully tries to abstract itself** from these operations. This means we have to create a window, define a context, and handle user input all by ourselves.

### 3. Hello Window(GLFW: Context and Window)

Compiling the library from the source code guarantees that the resulting library is perfectly tailored for your CPU/OS, a luxury pre-compiled binaries don't always provide (sometimes, pre-compiled binaries are not available for your system)

- use glfw as framebuffer driver to compatibal for all os platform
- use glad for load OpenGL library
- do some widow initialize
- it done



### 4. Hello Triangle

**In OpenGL everything is in 3D space**, but the screen or window is a 2D array of pixels so a large part of OpenGL's work is about transforming all 3D coordinates to 2D pixels that fit on your screen

The graphics pipeline can be divided into **two large parts**: the first transforms your 3D coordinates into 2D coordinates and the second part transforms the 2D coordinates into actual colored pixels

All of these steps are highly specialized (they have one specific function) and can easily be **executed in parallel**. Because of their parallel nature, graphics cards of today have thousands of **small processing cores to quickly process your data within the graphics pipeline**. The processing cores run small programs on the GPU for each step of the pipeline. **These small programs are called shaders.**

自由替换Shader：

Some of these shaders are configurable by the developer which allows us to write our own shaders to replace the existing default shaders. 

Shaders are written in the **OpenGL Shading Language (GLSL) **



[! imge/pipeline ]

**vertex shader** that takes as input a single vertex.he main purpose of the vertex shader is to transform 3D coordinates into different 3D coordinates (more on that later) and the vertex shader **allows us to do some basic processing on the vertex attributes**

the **primitive assembly** stage takes as input all the vertices (or vertex if GL_POINTS is chosen) from the vertex shader that form a primitive and assembles all the point(s) in the primitive shape given; in this case a triangle.

**primitives** and are given to OpenGL while calling any of the drawing commands. Some of these hints are GL_POINTS, GL_TRIANGLES and GL_LINE_STRIP.

**The geometry shader** takes as input a collection of vertices that form a primitive and has the ability to generate other shapes by emitting new vertices to form new (or other) primitive(s). In this example case, it generates a second triangle out of the given shape.

the output of the geometry shader is then passed on to the **rasterization stage** where it maps the resulting primitive(s) to the **corresponding pixels** on the final screen, resulting in fragments for the fragment shader to use. Before the fragment shaders run, **clipping is performed. Clipping discards all fragments that are outside your view, increasing performance.**

A **fragment** in OpenGL is all the data required for OpenGL to render a single pixel.

The main purpose of the **fragment shader** is to calculate the final color of a pixel and this is usually the stage **where all the advanced OpenGL effects occur**,Usually the fragment shader contains data about the 3D scene that it can **use to calculate the final pixel color (like lights, shadows, color of the light and so on).**

After all the corresponding color values have been determined, the final object will then pass through one more stage that we call the **alpha test and blending stage.** This stage checks the corresponding **depth (and stencil) value** (we'll get to those later) of the fragment and uses those to check if the resulting fragment is in front or behind other objects and should be discarded accordingly. The stage also checks for **alpha values** (alpha values define the opacity of an object) and blends the objects accordingly. So even if a pixel output color is calculated in the fragment shader, the final pixel color could still be something entirely different when rendering multiple triangles.



In modern OpenGL we are **required** to define **at least a vertex and fragment shader** of our own (there are no default vertex/fragment shaders on the GPU).

##### Vertex Input

OpenGL doesn't simply transform **all** your 3D coordinates to 2D pixels on your screen; OpenGL only processes 3D coordinates when they're in a specific range between `-1.0` and `1.0` on all 3 axes (`x`, `y` and `z`). 

All coordinates within this so called **normalized device coordinates range** will end up visible on your screen (and all coordinates outside this region won't)

> **Normalized Device Coordinates (NDC)**
>
> -1 1 mapping to screen coordinate



- 定义顶点数据 传入 Graphic Pipline 的 first process：vertex shader：This is done by creating **memory on the GPU** where we store the vertex data，configure how OpenGL should interpret the memory and specify how to send the data to the graphics card，The vertex shader then processes as much vertices as we tell it to from its memory.

We manage this memory via so called **vertex buffer objects (VBO)** that can store a large number of vertices in the GPU's memory. 单次传入大量顶点数据，Sending data to the graphics card from the CPU is relatively slow，提高性能



Just like any object in OpenGL, this buffer has a unique ID corresponding to that buffer

OpenGL has many types of buffer objects and the buffer type of a vertex buffer object is GL_ARRAY_BUFFER. OpenGL allows us to bind to several buffers at once as long as they have a different buffer type. We can bind the newly created buffer to the GL_ARRAY_BUFFER target with the glBindBuffer function:

```c++
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
```





glBufferData is a function specifically targeted to copy user-defined data into the currently bound buffer.

glBufferData(GL_ARRAY_BUFFER, **sizeof**(vertices), vertices, GL_STATIC_DRAW);

- GL_STREAM_DRAW: the data is set only once and used by the GPU at most a few times.
- GL_STATIC_DRAW: the data is set only once and used many times.
- GL_DYNAMIC_DRAW: the data is changed a lot and used many times



Shader 需要用GLSL写：As you can see, GLSL looks similar to C

GLSL has a vector datatype that contains 1 to 4 floats based on its postfix digit

```c
// 版本 core mode
#version 330 core 
// We also specifically set the location of the input variable via layout (location = 0) and you'll later see that why we're going to need that location.
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

 Note that the `vec.w` component is not used as a position in space (we're dealing with 3D, not 4D) but is used for something called perspective division. 



#### Vertex Shader

- write shader using GLSL

- Compiling a shader

  In order for OpenGL to use the shader it has to dynamically compile it at run-time from its source code. The first thing we need to do is create a shader object, again referenced by an ID.

  create the shader with glCreateShader

```c
// create shader

**unsigned** **int** vertexShader;

 vertexShader = glCreateShader(GL_VERTEX_SHADER);

// set shader source code

**const** **char** *vertexShaderSource = "#version 330 core\n"    "layout (location = 0) in vec3 aPos;\n"    "void main()\n"    "{\n"    "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"    "}\0";

glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);

// compile shader runtime

 glCompileShader(vertexShader);

// check compile success
int  success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
if(!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```



#### Fragment shader

The fragment shader is all about calculating the color output of your pixels

When defining a color in OpenGL or GLSL we set the strength of each component to a value between `0.0` and `1.0`. 



We can declare output values with the `out` keyword

> \#version 330 core
>
>  **out** vec4 FragColor;
>
>  **void** main() {    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f); } 



```c

unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);
```



the only thing left to do is **link both shader objects into a shader program that we can use for rendering**. Make sure to check for compile errors here as well!

#### ShaderProgram

A shader program object is the final linked version of multiple shaders combined

The activated shader program's shaders will be used when we issue render calls.

```c
// creating program object
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
// attach to program
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
// link the shader
glLinkProgram(shaderProgram);

glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
if(!success) {
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
    ...
}

// active the program
glUseProgram(shaderProgram);
//Every shader and rendering call after glUseProgram will now use this program object (and thus the shaders).

// Oh yeah, and don't forget to delete the shader objects once we've linked them into the program object; we no longer need them anymore:
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);  


```

#### Binding Vertex Attribute

OpenGL does not yet know how it should interpret the vertex data in memory and how it should connect the vertex data to the vertex shader's attributes. We'll be nice and tell OpenGL how to do that.

With this knowledge we can tell OpenGL how it should interpret the vertex data (per vertex attribute) using `glVertexAttribPointer`

```c

glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
// layout (location = 0), vec3, type of data, normalized?,This is the offset of where the position data begins in the buffer
glEnableVertexAttribArray(0);  
```

- The first parameter specifies which vertex attribute we want to configure. Remember that we specified the location of the position vertex attribute in the vertex shader with `layout (location = 0)`. This sets the location of the vertex attribute to `0` and since we want to pass data to this vertex attribute, we pass in `0`.
- The next argument specifies the size of the vertex attribute. The vertex attribute is a `vec3` so it is composed of `3` values.
- The third argument specifies the type of the data which is GL_FLOAT (a `vec*` in GLSL consists of floating point values).
- The next argument specifies if we want the data to be normalized. If we're inputting integer data types (int, byte) and we've set this to GL_TRUE, the integer data is normalized to `0` (or `-1` for signed data) and `1` when converted to float. This is not relevant for us so we'll leave this at GL_FALSE.
- The fifth argument is known as the stride and tells us the space between consecutive vertex attributes. Since the next set of position data is located exactly 3 times the size of a `float` away we specify that value as the stride. Note that since we know that the array is tightly packed (there is no space between the next vertex attribute value) we could've also specified the stride as `0` to let OpenGL determine the stride (this only works when values are tightly packed). Whenever we have more vertex attributes we have to carefully define the spacing between each vertex attribute but we'll get to see more examples of that later on.
- The last parameter is of type `void*` and thus requires that weird cast. This is the offset of where the position data begins in the buffer. Since the position data is at the start of the data array this value is just `0`. We will explore this parameter in more detail later on



```c
// 0. copy our vertices array in a buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 1. then set the vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);  
// 2. use our shader program when we want to render an object
glUseProgram(shaderProgram);
// 3. now draw the object 
someOpenGLFunctionThatDrawsOurTriangle();   
```

We have to repeat this process every time we want to draw an object --- s0 VAO

#### VertexArrayObject

A vertex array object (also known as VAO) can be bound just like a vertex buffer object and any subsequent vertex attribute calls from that point on will be stored inside the VAO

advantage that when configuring vertex attribute pointers you only have to make those calls once and whenever we want to draw the object, we can just bind the corresponding VAO. This makes **switching between different vertex data and attribute configurations as easy as binding a different VAO**. All the state we just set is stored inside the VAO.



Core OpenGL **requires** that we use a VAO so it knows what to do with our vertex inputs. If we fail to bind a VAO, OpenGL will most likely refuse to draw anything.



```c
unsigned int VAO;
glGenVertexArrays(1, &VAO);

// ..:: Initialization code (done once (unless your object frequently changes)) :: ..
// 1. bind Vertex Array Object
glBindVertexArray(VAO);
// 2. copy our vertices array in a buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 3. then set our vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);  

  
[...]

// ..:: Drawing code (in render loop) :: ..
// 4. draw the object
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
someOpenGLFunctionThatDrawsOurTriangle();  

glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawArrays(GL_TRIANGLES, 0, 3);
```

Usually when you have multiple objects you want to draw, you first generate/configure all the VAOs (and thus the required VBO and attribute pointers) and store those for later use. The moment we want to draw one of our objects, we take the corresponding VAO, bind it, then draw the object and unbind the VAO again.



#### Element Buffer Objects

here is one last thing we'd like to discuss when rendering vertices and that is element buffer objects abbreviated to EBO

An EBO is a buffer, just like a vertex buffer object, that stores indices that OpenGL uses to decide what vertices to draw. This so called indexed drawing is exactly the solution to our problem

**Wireframe mode**
To draw your triangles in wireframe mode, you can configure how OpenGL draws its primitives via `glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)`. The first argument says we want to apply it to the front and back of all triangles and the second line tells us to draw them as lines. Any subsequent drawing calls will render the triangles in wireframe mode until we set it back to its default using `glPolygonMode(GL_FRONT_AND_BACK, GL_FILL)`.

### 5.Shaders

 shaders are little programs that rest on the GPU. These programs are run for each specific section of the graphics pipeline. In a basic sense, **shaders are nothing more than programs transforming inputs to outputs**

Shaders are also **very isolated programs** in that they're not allowed to communicate with each other; the only communication they have is via their inputs and outputs

#### 5.1 GLSL

GLSL is tailored for use with graphics and contains useful features specifically targeted at vector and matrix manipulation --- 向量和矩阵操作友好

Each shader's entry point is at its main function where we process any input variables and output the results in its output variables --- 一个main 处理输入和输出

```c
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;
  
uniform type uniform_name;
  
void main()
{
  // process input(s) and do some weird graphics stuff
  ...
  // output processed stuff to output variable
  out_variable_name = weird_stuff_we_processed;
}
```

Vertex shader : input is vertex attribute

There is a maximum number of vertex attributes we're allowed to declare limited by the hardware, minimum is 14 vec4, querying GL_MAX_VERTEX_ATTRIBS

```c++
int nrAttributes;
glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &nrAttributes);
```

##### 5.1.1 Types

GLSL has most of the default basic types we know from languages like C: `int`, `float`, `double`, `uint` and `bool`. GLSL also features two container types that we'll be using a lot, namely `vectors` and `matrices`. 

**vectors**

- `vecn`: the default vector of `n` floats.
- `bvecn`: a vector of `n` booleans.
- `ivecn`: a vector of `n` integers.
- `uvecn`: a vector of `n` unsigned integers.
- `dvecn`: a vector of `n` double components.

GLSL also allows you to use `rgba` for colors or `stpq` for texture coordinates, accessing the same components.

**swizzling**

```c
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

##### 5.1.2 Ins and outs

Shaders are nice little programs on their own, but they are part of a whole and for that reason we want to have inputs and outputs on the individual shaders so that we can move stuff around. GLSL defined the `in` and `out` keywords specifically for that purpose

The **vertex shader** thus requires an **extra layout specification** for its inputs so we can link it with the vertex data.

The other exception is that the **fragment shader requires a `vec4` color output variable**

##### 5.1.3 Uniforms

Uniforms are another way to **pass data from our application on the CPU to the shaders on the GPU**

Uniforms are however slightly different compared to vertex attributes.

First of all, **uniforms are global**. Global, meaning that a uniform variable is unique per shader program object, and can be accessed from any shader at any stage in the shader program. Second, whatever you set the uniform value to, uniforms will **keep their values until they're either reset or updated**.

Note that finding the uniform location does not require you to use the shader program first, but updating a uniform **does** require you to first use the program (by calling glUseProgram), because it sets the uniform on the currently active shader program

> Because OpenGL is in its core a C library it does not have native support for function overloading, so wherever a function can be called with different types OpenGL defines new functions for each type required



#####  5.1.4 More Attr

add color data to the vertex data in VBO

### 6. Textures

What artists and programmers generally prefer is to use a texture. A texture is a 2D image (even 1D and 3D textures exist) used to add detail to an object

textures can also be used to store a large collection of arbitrary data to send to the shaders

Texture coordinates range from `0` to `1` in the `x` and `y` axis (remember that we use 2D texture images). Retrieving the texture color using texture coordinates is called **sampling**. 

Texture coordinates start at `(0,0)` for the lower left corner of a texture image to `(1,1)` for the upper right corner of a texture image.

Texture sampling has a loose interpretation and can be done in many different ways. **It is thus our job to tell OpenGL how it should *sample* its textures.**



#### 6.1 Texture Wrapping

Texture coordinates usually range from `(0,0)` to `(1,1)` but what happens if we specify coordinates outside this range? 

- GL_REPEAT: The default behavior for textures. Repeats the texture image.
- GL_MIRRORED_REPEAT: Same as GL_REPEAT but mirrors the image with each repeat.
- GL_CLAMP_TO_EDGE: Clamps the coordinates between `0` and `1`. The result is that higher coordinates become clamped to the edge, resulting in a stretched edge pattern.
- GL_CLAMP_TO_BORDER: Coordinates outside the range are now given a user-specified border color.

Each of the aforementioned options can be set per coordinate axis (`s`, `t` (and `r` if you're using 3D textures) equivalent to `x`,`y`,`z`) with the glTexParameter* function:

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
```

If we choose the GL_CLAMP_TO_BORDER option we should also specify a border color

This is done using the `fv` equivalent of the glTexParameter function with GL_TEXTURE_BORDER_COLOR as its option where we pass in a float array of the border's color value:

```c++
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);  
```

##### 6.2 Texture Filtering -- Sample Algorithm with low resolution on a large object

Texture coordinates do not depend on resolution but can be any floating point value, thus OpenGL has to figure out which texture pixel (also known as a texel ) to map the texture coordinate to

There are several options available but for now we'll discuss the most important options: GL_NEAREST and GL_LINEAR.

GL_NEAREST (also known as nearest neighbor or point filtering) is the default texture filtering method of OpenGL

GL_LINEAR (also known as (bi)linear filtering) takes an interpolated value from the texture coordinate's neighboring texels, approximating a color between the texels

Texture filtering can be set for magnifying and minifying operations (when scaling up or downwards) so you could for example use nearest neighbor filtering when textures are scaled downwards and linear filtering for upscaled textures. We thus have to specify the filtering method for both options via glTexParameter*. The code should look similar to setting the wrapping method:

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```



##### 6.3 mipmaps -- Sample Algorithm with high resolution on a small object

To solve this issue OpenGL uses a concept called mipmaps that is basically a collection of texture images where each subsequent texture is twice as small compared to the previous one. The idea behind mipmaps should be easy to understand: after a certain distance threshold from the viewer, OpenGL will use a different mipmap texture that best suits the distance to the object. Because the object is far away, the smaller resolution will not be noticeable to the user. OpenGL is then able to sample the correct texels, and there's less cache memory involved when sampling that part of the mipmaps

OpenGL is able to do all the work for us with a single call to glGenerateMipmaps after we've created a texture.

When switching between mipmaps levels during rendering OpenGL may show some artifacts like sharp edges visible between the two mipmap layers. Just like normal texture filtering, it is also possible to filter between mipmap levels using NEAREST and LINEAR filtering for switching between mipmap levels.

- GL_NEAREST_MIPMAP_NEAREST: takes the nearest mipmap to match the pixel size and uses nearest neighbor interpolation for texture sampling.
- GL_LINEAR_MIPMAP_NEAREST: takes the nearest mipmap level and samples that level using linear interpolation.
- GL_NEAREST_MIPMAP_LINEAR: linearly interpolates between the two mipmaps that most closely match the size of a pixel and samples the interpolated level via nearest neighbor interpolation.
- GL_LINEAR_MIPMAP_LINEAR: linearly interpolates between the two closest mipmaps and samples the interpolated level via linear interpolation.

Just like texture filtering we can set the filtering method to one of the 4 aforementioned methods using glTexParameteri:

```
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

A common mistake is to set one of the mipmap filtering options as the magnification filter. This doesn't have any effect since **mipmaps are primarily used for when textures get downscaled**: texture magnification doesn't use mipmaps and giving it a mipmap filtering option will generate an OpenGL GL_INVALID_ENUM error code.

##### 6.4 Loading and creating textures - use stbi image loader

##### 6.5 Generating a texture

```c++
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
// set the texture wrapping/filtering options (on the currently bound texture object)
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);	
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
// load and generate the texture
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
if (data)
{
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
}
else
{
    std::cout << "Failed to load texture" << std::endl;
}
stbi_image_free(data);
```



##### 6.6 Applying textures

We need to inform OpenGL how to sample the texture so we'll have to update the vertex data with the texture coordinates:

```c++
float vertices[] = {
    // positions          // colors           // texture coords
     0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // top right
     0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // bottom right
    -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // bottom left
    -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // top left 
};
```



##### 6.7 Texture Units

Using glUniform1i we can actually assign a *location* value to the texture sampler so we can set multiple textures at once in a fragment shader. This location of a texture is more commonly known as a texture unit.

The main purpose of texture units is to allow us to use more than 1 texture in our shaders. By assigning texture units to the samplers, we can bind to multiple textures at once as long as we activate the corresponding texture unit first.

> OpenGL should have a at least a minimum of 16 texture units for you to use which you can activate using GL_TEXTURE0 to GL_TEXTURE15. They are defined in order so we could also get GL_TEXTURE8 via GL_TEXTURE0 + 8 for example, which is useful when we'd have to loop over several texture units



### 7. Transformations

We could try and make them move by changing their vertices and re-configuring their buffers each frame, but that's cumbersome and **costs quite some processing power**. There are much **better ways to transform** an object and that's by using (multiple) matrix objects

Identity Matrix

The identity matrix is an `NxN` matrix with only 0s except on its diagonal. : 做点乘的时候，对向量是完全没有影响的

The identity matrix is usually a starting point for generating other transformation matrices and if we dig even deeper into linear algebra, a very useful matrix for proving theorems and solving linear equations.

Scaling

（S1, S2, S3）分布到Identity Matrix 的每个对角的值，便是非均匀scale

Translation

(T1, T2, T3) 分布到单位矩阵的 w所在的列，能对x y z的 vector值做到加法的效果，便是平移的效果了

> **Homogeneous coordinates**
> The `w` component of a vector is also known as a homogeneous coordinate. To get the 3D vector from a homogeneous vector we divide the `x`, `y` and `z` coordinate by its `w` coordinate. We usually do not notice this since the `w` component is `1.0` most of the time. Using homogeneous coordinates has several advantages: it allows us to do matrix translations on 3D vectors **(without a `w` component we can't translate vectors)** and in the next chapter we'll use the `w` value to create 3D perspective.
>
> Also, whenever the homogeneous coordinate is equal to `0`, the vector is specifically known as a direction vector since a vector with a `w` coordinate of `0` cannot be translated.

Rotation

otations are a bit trickier

The angle specified will rotate the object along the rotation axis given.

分别旋转一个 axis：
例如：旋转x-axis ：对 y z 进行2D旋转

Mx mul My mul Mz 最终得到最终选装效果

However, this quickly introduces a problem called Gimbal lock.

**OpenGL does not have any form of matrix or vector knowledge built in, so we have to define our own mathematics classes and functions.**

#### GLM： gl math

GLM stands for Open**GL** **M**athematics and is a *header-only* library, which means that we only have to include the proper header files and we're done; no linking and compiling necessary. GLM can be downloaded from their [website](https://glm.g-truc.net/0.9.8/index.html). Copy the root directory of the header files into your *includes* folder and let's get rolling.

1. compute transform Matrix
2. get the transformation matrix to the Shader

OpenGL developers often use an internal matrix layout called **column-major ordering** which is the default matrix layout in GLM so there is no need to transpose the matrices

### 8. Coordinate System

There are a total of 5 different coordinate systems that are of importance to us:

- Local space (or Object space)
- World space
- View space (or Eye space)
- Clip space
- Screen space

1. Local coordinates are the coordinates of your object relative to its local origin; they're the coordinates your object begins in.
2. The next step is to transform the local coordinates to world-space coordinates which are coordinates in respect of a larger world. These coordinates are relative to some global origin of the world, together with many other objects also placed relative to this world's origin.
3. Next we transform the world coordinates to view-space coordinates in such a way that each coordinate is as seen from the camera or viewer's point of view.
4. After the coordinates are in view space we want to project them to clip coordinates. **Clip coordinates are processed to the `-1.0` and `1.0` range and determine which vertices will end up on the screen**. Projection to clip-space coordinates can add **perspective** if using perspective projection.
5. And lastly we transform the clip coordinates to screen coordinates in a process we call **viewport transform** that transforms the coordinates from `-1.0` and `1.0` to the coordinate range defined by glViewport. The resulting coordinates are then sent to the rasterizer to turn them into fragments.



The coordinates of your object are transformed from local to world space; this is accomplished with the model matrix.

This *viewing box* a projection matrix creates is called a **frustum** and each coordinate that ends up inside this frustum will end up on the user's screen. 

Once all the vertices are transformed to clip space a final operation called **perspective division** is performed where we divide the `x`, `y` and `z` components of the position vectors by the vector's homogeneous `w` component; perspective division is what transforms the 4D clip space coordinates to 3D normalized device coordinates. This step is performed automatically at the end of the vertex shader step.

**orthographic projection**

An orthographic projection matrix defines a cube-like frustum box that defines the clipping space where each vertex outside this box is clipped.The frustum looks a bit like a container

**Perspective projection**

The projection matrix maps a given frustum range to clip space, but also manipulates the `w` value of each vertex coordinate in such a way that the further away a vertex coordinate is from the viewer, the higher this `w` component becomes

OpenGL requires that the visible coordinates fall between the range `-1.0` and `1.0` as the final vertex shader output, thus once the coordinates are in clip space, perspective division is applied to the clip space coordinates: x y z 每个分量都要除以 w，

Each component of the vertex coordinate is divided by its `w` component giving smaller vertex coordinates the further away a vertex is from the viewer. This is another reason why the `w` component is important, since it helps us with perspective projection.

这个 w component 其实是：当距离，透视投影需要根据距离做缩放，来得到透视的效果

http://www.songho.ca/opengl/gl_projectionmatrix.html

>  glm::mat4 proj = glm::perspective(glm::radians(45.0f), (**float**)width/(**float**)height, 0.1f, 100.0f);

#### Practice 

OpenGL is a right-handed system we have to move in the positive z-axis.

 OpenGL draws your cube triangle-by-triangle, fragment by fragment, it will overwrite any pixel color that may have already been drawn there before. Since OpenGL gives no guarantee on the order of triangles rendered (within the same draw call), some triangles are drawn on top of each other even though one should clearly be in front of the other.

Luckily, OpenGL stores depth information in a buffer called the z-buffer that allows OpenGL to decide when to draw over a pixel and when not to. Using the z-buffer we can configure OpenGL to do depth-testing.

### 9. Camera

OpenGL by itself is not familiar with the concept of a *camera*, but we can try to simulate one by moving all objects in the scene in the reverse direction, giving the illusion that **we** are moving

### 10. Review



## 3. Lighting





