# glfwInit()
### Function Specification
[Link to GLFW docs entry](https://www.glfw.org/docs/3.3/group__init.html#ga317aac130a235ab08c6db0834907d85e)

[Link to Function Source](https://github.com/glfw/glfw/blob/1fe98a0d5388435720d380439497630ac0477455/src/init.c#L410)

### int glfwInit(void)
>This function initializes the GLFW library. Before most GLFW functions can be used, GLFW must be initialized, and before an application terminates GLFW should be terminated in order to free any resources allocated during or after initialization.

>If this function fails, it calls **glfwTerminate** before returning. If it succeeds, you should call glfwTerminate before the application exits.

>Additional calls to this function after successful initialization but before termination will return GLFW_TRUE immediately.

### Actual Function Body
This function is pretty verbose but definitely not as big as others. At its core, it initializes the internal _glfw object, which holds ALL of the state that glfw tracks, such as context, windows, current platform, etc. 

It also goes ahead and figures out what platform we are using, creating the necessary platform struct and initializing it (In our case, primarily initializes x11 utils and dynamically grabs glx)

ADDITIONALLY, by default GLFW supports an operating system independent multi-threading layer. This is where the `glfwPlatformCreateMutex`, and `glfwPlatformCreateTls`come in. These two create a Mutex for managing shared resources between threads, and a "Thread Local Storage" system. If we intend to support multi-threading, it might be better to opt in to platform-specific (Linux) multithreading, but keeping something like this may also work great.

`glfwInitGamepadMappings` just takes from a humongous table and maps popular gamepad inputs to specific symbols/values to register callbacks for an all that

`glfwPlatformInitTimer()` initializes the platform-specific timing system. The API is then able to expose functions like `glfwSetTime(double time)`, and `glfwSleep(double time)` by leveraging the platform's timer mechanism.

Finally, `glfwDefaultWindowHints()` sets all window-related hints to their default values. 
>  Window Hints are a mechanism to specify preferences or requirements for the window and context creation process before the window is actually created. These hints can include a wide range of options such as the OpenGL version, whether to use a fullscreen mode, the window size, and many others.

The function returns true if everything went well, false if otherwise.

```C++
GLFWAPI int glfwInit(void)
{
    if (_glfw.initialized)
        return GLFW_TRUE;

    memset(&_glfw, 0, sizeof(_glfw));
    _glfw.hints.init = _glfwInitHints;

    _glfw.allocator = _glfwInitAllocator;
    if (!_glfw.allocator.allocate)
    {
        _glfw.allocator.allocate   = defaultAllocate;
        _glfw.allocator.reallocate = defaultReallocate;
        _glfw.allocator.deallocate = defaultDeallocate;
    }

    if (!_glfwSelectPlatform(_glfw.hints.init.platformID, &_glfw.platform))
        return GLFW_FALSE;

    if (!_glfw.platform.init())
    {
        terminate();
        return GLFW_FALSE;
    }

    if (!_glfwPlatformCreateMutex(&_glfw.errorLock) ||
        !_glfwPlatformCreateTls(&_glfw.errorSlot) ||
        !_glfwPlatformCreateTls(&_glfw.contextSlot))
    {
        terminate();
        return GLFW_FALSE;
    }

    _glfwPlatformSetTls(&_glfw.errorSlot, &_glfwMainThreadError);

    _glfwInitGamepadMappings();

    _glfwPlatformInitTimer();
    _glfw.timer.offset = _glfwPlatformGetTimerValue();

    _glfw.initialized = GLFW_TRUE;

    glfwDefaultWindowHints();
    return GLFW_TRUE;
}
```

We don't care too much about how the platform is selected, since we are specifically dealing with Linux/X11 Window System. However,
we do care about how the platform is initialized. The primary function we will be drilling down to is the platform.init() function

the `platform.init` function actually comes from the g`glfwSelectPlatform` function, that calls a function called `connect` on the supported platform (x11 in our case), which uses a series of IFNDEF's to link to `_glfwConnectX11`. Finally, this results in `platform.init` being equal to `_glfwInitX11`