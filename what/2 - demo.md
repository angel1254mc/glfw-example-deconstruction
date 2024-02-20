# The Demo itself

```C
#include "include/glad/glad.h"
#include <GLFW/glfw3.h>

int main() {
    if (!glfwInit()) {
        return -1;
    }

    GLFWwindow* window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);

    if (!window) {
        glfwTerminate();
        return -1;
    }

    glfwMakeContextCurrent(window);

    // glClearColor(0.25f, 0.5f, 0.75f, 1.0f);

    while (!glfwWindowShouldClose(window)) {
        // glClear(GL_COLOR_BUFFER_BIT);
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

Above we have code that renders a simple blank, resizeable window using GLFW. It is composed of 
- **glfwInit()**: an initialization function that sets up the library we're using (covered more when we dive into it)
- **glfwCreateWindow()**: A function that both creates an internal representation of a x11 window (including creating a context) that we can work with, and actually performs all the X11 functions to put a Window w/ Colormap on screen.
- **glfwMakeContextCurrent()**: a function that establishes our current OpenGL context (1)
- **glfwWindowShouldClose()**: A function that polls glfw's state for any events/signs that the window should close. The while loop we place it in serves as our **explicit** message processing loop (2)
- **glfwSwapBuffers()**: A function that swaps the front and back buffers of our context, rendering a new picture on screen (3)
- **glfwPollEvents()**: A function that "processes events that have already been received, and triggers window and input callbacks associated with those events" [See docs](https://www.glfw.org/docs/3.0/group__window.html#ga37bd57223967b4211d60ca1a0bf3c832:~:text=void%20glfwPollEvents,))


Extra Details
1) glMakeContextCurrent() - To understand what this does, we need to know about Contexts. According to the OpenGL Wiki:
>  A context stores all of the state associated with this instance of OpenGL. It represents the (potentially visible) default framebuffer that rendering commands will draw to when not drawing to a framebuffer object. Think of a context as an object that holds all of OpenGL; when a context is destroyed, OpenGL is destroyed.

OpenGL is essentially a state machine where each node is a context. When you make a context the "active" context, this means that all subsequent commands will refer to that context. For example, when you call something like `glBindTexture()`, internally it could be likened to `getCurrentContext()->glBindTexture()`. The way you change the current context is through this function. 

GLFW, like FreeGLUT, primarily wraps around the glx [MakeContextCurrent](https://github.com/anholt/mesa/blob/01e511233b24872b08bff862ff692dfb5b22c1f4/src/glx/glxcurrent.c#L174) function to achieve this. [GLX is an extension to the X Window System core protocol providing an interface between OpenGL and the X Window System](https://en.wikipedia.org/wiki/GLX)

2) glfwWindowShouldClose() - As mentioned, this function polls our internal glfw state to see whether or not the window should close. This is typically used to signal the end of the message processing loop and the beginning of tearing down the program.

One important detail is that the message processing loop here is explicit (we have control over what goes first, event processing, display, etc) rather than implicit (freeglut main loop lives inside the freeglut code, as we saw a couple of days back). This is only really meaningful for the low-level api team, since this would mean that we could leave the fine details of ordering the main program loop up to the high-level api team.

3) Buffers are the data representation of what is drawn/is to be drawn on screen. The front buffer is the one currently drawn, the back buffer is the one that we will draw and can modify before swapping it forward (as far as I understand it) [See this Link for more details](https://community.khronos.org/t/understanding-the-opengl-main-loop-swapbuffers/75593)

This comment from user GClements from the [Khronos Community Forum](https://community.khronos.org/t/swapbuffers-when-to-use/104682) helps cement this:
> You should call it (glutSwapBuffers) at the end of the display function (which is the only place rendering operations should be performed). With a double-buffered context, all rendering operations are performed on an off-screen buffer (the back buffer). The only way to make the rendering result appear on the screen is to use glutSwapBuffers to copy the back buffer to the front buffer. The contents of the back buffer will be undefined after this call.

These are the 6 main functions we'll be looking through to get a better understanding of how they work.



- glfwInit()
    - _glfwInitX11()
- [glfwCreateWindow()](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/window.c#L180)
    - [fbconfig, ctxconfig, wndconfig, window](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/window.c#L205-L207)
    - [_glfwCreateWindowX11()](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/x11_window.c#L1959)
        - [_glfwInitGLX()](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/glx_context.c#L258)
        - [DefaultVisual()](https://tronche.com/gui/x/xlib/display/display-macros.html#:~:text=a%20single%20screen.-,DefaultVisual,-DefaultVisual)
        - [DefaultDepth()](https://tronche.com/gui/x/xlib/display/display-macros.html#:~:text=of%20this%20colormap.-,DefaultDepth)
        - [_glfwCreateContextGLX()](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/glx_context.c#L452)
            - [chooseGLXFBConfig()](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/glx_context.c#L52)
            - [glxCreateWindow(display, native, handle, null)](https://github.com/Mesa3D/mesa/blob/31c9e17bf2a5e3ff73fce7a2cf14db121c5bbc90/src/gallium/frontends/glx/xlib/glx_api.c#L1882)
        - [CreateNativeWindow(window, wndconfig, visual, depth)](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/x11_window.c#L1959)
            - [XCreateColormap(display, root, visual, AllocNone)](https://tronche.com/gui/x/xlib/color/XCreateColormap.html)
            - glfwIsVisualTransparentX11()
            - [XCreateWindow(display, root, x, y, w, h, etc...)](https://tronche.com/gui/x/xlib/window/XCreateWindow.html)
            - XSaveContext(display, handle, context, window)
            - set atoms, wm details and hints, PID,
            - [glfwCreateInputContextX11(window)](https://github.com/glfw/glfw/blob/8f2f766f0d2ed476c03a2ae02e48ac41a9602b03/src/x11_window.c#L1922)
                - [XCreateIC()](https://www.x.org/releases/X11R7.5/doc/man/man3/XIMOfIC.3.html)
                - [XGetICValues()](https://linux.die.net/man/3/xgeticvalues)
                - [XSelectInput()](https://tronche.com/gui/x/xlib/event-handling/XSelectInput.html)
            - glfwSetWindowTitleX11()
                - [Xutf8SetWMProperties()](https://linux.die.net/man/3/xutf8setwmproperties)
                - [XChangeProperty(WM_NAME)](https://tronche.com/gui/x/xlib/window-information/XChangeProperty.html)
                - [XChangeProperty(WM_ICON_NAME)](https://tronche.com/gui/x/xlib/window-information/XChangeProperty.html)
                - [XFlush()](https://tronche.com/gui/x/xlib/event-handling/XFlush.html)
            - [glfwGetWindowPosX11()](https://github.com/glfw/glfw/blob/6f1ddf51a130f2dee6ade5fa4d8217e4071124e8/src/x11_window.c#L2153)
                - [XTranslateCoordinates()](https://github.com/glfw/glfw/blob/6f1ddf51a130f2dee6ade5fa4d8217e4071124e8/src/x11_window.c#L2153)
            - [glfwGetWindowSizeX11](https://github.com/glfw/glfw/blob/6f1ddf51a130f2dee6ade5fa4d8217e4071124e8/src/x11_window.c#L2191)
                - [XGetWindowAttributes()](https://tronche.com/gui/x/xlib/window-information/XGetWindowAttributes.html)
        - [glfwCreateContextGLX(window, ctxconfig, fbconfig)](https://github.com/glfw/glfw/blob/2b3f919b6055e6837ca0fad193c7f96b323f1256/src/glx_context.c#L452)
            - [chooseGLXFBConfig()](https://github.com/glfw/glfw/blob/2b3f919b6055e6837ca0fad193c7f96b323f1256/src/glx_context.c#L52)
                - [glxGetFBConfigs(display, screen, int& count)](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glXGetFBConfigs.xml)
                - [For above, see also Mesa3D](https://github.com/Mesa3D/mesa/blob/31c9e17bf2a5e3ff73fce7a2cf14db121c5bbc90/src/glx/glxcmds.c#L1464)

