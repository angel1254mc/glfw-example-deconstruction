# GLFW-EXAMPLE-DECONSTRUCTION

This public repo is where I'll be writing down my notes as I "follow" the stack trace of a super simple glfw program that opens up a basic, resizeable window in X11.

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

## Repo structure
I'm going to be separating notes into folders corresponding to functions, with the exception of one folder that will cover glfw, glad, opengl and any other fundamental dependencies/libraries used in any of these functions.

- what
- glfwInit
- glfwCreateWindow
- glfwMakeContextCurrent
- glfwWindowShouldClose(window)
- glfwSwapBuffers(window)
- glfwPollEvents()