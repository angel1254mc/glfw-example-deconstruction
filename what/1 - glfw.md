# GLFW
GLFW is a free, Open Source, multi-platform library for OpenGL, OpenGL ES, and Vulkan app development.
It works as a platform-independent API for handling Windows, Context, Surfaces, Inputs, and Events! Typically you'd use GLFW as a platform-independent way of interfacing with and modifying windows, contexts, etc., while you'd use raw OpenGL (usually by leveraging tools that automatically load OpenGL function pointers, such as GLAD or GLEW) for actually putting stuff on screen.

## GLFW and FreeGLUT (and Senior Project)
The whole reason why we're looking at GLFW is because of some difficulties we've faced with FreeGlut, another free, open-source, multi-platform library for OpenGL that basically does the same thing. Some of these difficulties are outlined below:

- (Kathy and Darren pwease list out any more difficulties I can only remember 1)
- Difficult setup on current systems and lack of tutorials/resources for use of FreeGlut on modern systems

The High-Level API team brought up GLFW as a suitable alternative that alleviates some/most of these problems. Luckily, GLFW is an open-source library, which means that we can actually see how each of the high-level components in the API work!

These Libraries (and OpenGL itself) are pretty huge, but we're able to take some liberties as part of our project.
- We are only working with one platform, no need for fancy dispatch tables or multi-platform abstractions. Just straight calls to our Windowing System (X11, I'll talk more about it later)
- We may have all of our underlying rendering logic run on the CPU, no need to go crazy coding up drivers (especially since each of us have a different GPU 😭)
- There is a pretty large existing body of code and literature on how the rendering pipeline works and how individual API functions relate to each step in the pipeline.

Since during the research phase, we primarily investigated GLUT by going down the stack for each relevant API function used in a super simple demo, we figured it would be best to do the same for GLFW and see how they compare.

## What is GLAD
If you look at the example code on the README, you'll see that we import both GLAD and GLFW

```C
#include "include/glad/glad.h"
#include <GLFW/glfw3.h>
```

The whole point of GLAD is to load openGL functions themselves onto our program. This is relevant because OpenGL is essentially just a [specification](https://publish.reddit.com/embed?url=https://www.reddit.com/r/cpp_questions/comments/ryr3fk/comment/hrqyder?snippet=2_0_390), an API for programmers to use to render stuff on screen. The responsibility of actually implementing this API is on the graphics drivers (software that runs on GPUs).

Without GLAD, the programmer would have to set up the function pointers themselves (and find where exactly the definitions for those functions live on the system 😭). Thankfully, GLAD handles all this and makes it super easy to use OpenGL.

### Does Glad Matter to us
Not really, since most of our code should be #1 running on the CPU, and #2 readily available on the RUDA repo, we won't need to do any fancy function loading. GLFW depends on GLAD, but if we build out own GLFW-like API for Windows, Contexts, and Event/Input handling we can just directly link it to our RUDA implementation.

This would also be the case if we stuck with FreeGLUT, our own developer toolkit would reference our own rudimentary renderer, so no difference there.