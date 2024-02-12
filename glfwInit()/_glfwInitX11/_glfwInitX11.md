# _glfwInitX11
### Function Specification
[Link to Function Source](https://github.com/glfw/glfw/blob/1fe98a0d5388435720d380439497630ac0477455/src/x11_init.c#L1320)

### int _glfwInitX11(void)
Unfortunately, no documentation for this function.

### Actual Function Body
All this function does essentially is dynamically load the function definitions for all necessary XLib functions to interface with the window using GLFW. NOTE: this does not replace the fact that OpenGL interfaces with GLX (GLX sits between X11 and OpenGL). Instead, XLib functions are used SPECIFICALLY to mess around with the window stuff, while OpenGL and GLFW still use GLX as an intermediary for managing and BINDING CONTEXTS TO WINDOWS. Proof of this can be seen in `x11_window.c` (contains all glfw code for managing winddows), where upon window creation [_glfwCreateWindowX11](https://github.com/glfw/glfw/blob/master/src/x11_window.c#L1969), we initialize (or verify that initialization has been done) GLX before creating a window and a context.

Then, we manually initialize all necessary structs (screen, root, and x11 context)
```C++
_glfw.x11.screen = DefaultScreen(_glfw.x11.display);
_glfw.x11.root = RootWindow(_glfw.x11.display, _glfw.x11.screen);
_glfw.x11.context = XUniqueContext();
```
and set content scale, create an event pipeline, initialize extension, and extra things such as polling for the number of moitors and representing those in memory. 

We will likely not have to go as low-level as it relates to monitor/window management. Our implementation will likely leverage the slightly higher Xlib set of functions which can be used to create windows in a significantly shorter time.

[X Window System Documentation](https://www.x.org/releases/current/doc/man/man3/)
[Gist showing window creation using XLib](https://gist.github.com/m1nuz/8f8f10a7f8715b62fe79)

```C++
int _glfwInitX11(void)
{
    _glfw.x11.xlib.AllocClassHint = (PFN_XAllocClassHint)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XAllocClassHint");
    _glfw.x11.xlib.AllocSizeHints = (PFN_XAllocSizeHints)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XAllocSizeHints");
    _glfw.x11.xlib.AllocWMHints = (PFN_XAllocWMHints)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XAllocWMHints");
    _glfw.x11.xlib.ChangeProperty = (PFN_XChangeProperty)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XChangeProperty");
    _glfw.x11.xlib.ChangeWindowAttributes = (PFN_XChangeWindowAttributes)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XChangeWindowAttributes");
    _glfw.x11.xlib.CheckIfEvent = (PFN_XCheckIfEvent)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCheckIfEvent");
    _glfw.x11.xlib.CheckTypedWindowEvent = (PFN_XCheckTypedWindowEvent)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCheckTypedWindowEvent");
    _glfw.x11.xlib.CloseDisplay = (PFN_XCloseDisplay)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCloseDisplay");
    _glfw.x11.xlib.CloseIM = (PFN_XCloseIM)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCloseIM");
    _glfw.x11.xlib.ConvertSelection = (PFN_XConvertSelection)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XConvertSelection");
    _glfw.x11.xlib.CreateColormap = (PFN_XCreateColormap)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCreateColormap");
    _glfw.x11.xlib.CreateFontCursor = (PFN_XCreateFontCursor)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCreateFontCursor");
    _glfw.x11.xlib.CreateIC = (PFN_XCreateIC)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCreateIC");
    _glfw.x11.xlib.CreateRegion = (PFN_XCreateRegion)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCreateRegion");
    _glfw.x11.xlib.CreateWindow = (PFN_XCreateWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XCreateWindow");
    _glfw.x11.xlib.DefineCursor = (PFN_XDefineCursor)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XDefineCursor");
    _glfw.x11.xlib.DeleteContext = (PFN_XDeleteContext)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XDeleteContext");
    _glfw.x11.xlib.DeleteProperty = (PFN_XDeleteProperty)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XDeleteProperty");
    _glfw.x11.xlib.DestroyIC = (PFN_XDestroyIC)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XDestroyIC");
    _glfw.x11.xlib.DestroyRegion = (PFN_XDestroyRegion)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XDestroyRegion");
    _glfw.x11.xlib.DestroyWindow = (PFN_XDestroyWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XDestroyWindow");
    _glfw.x11.xlib.DisplayKeycodes = (PFN_XDisplayKeycodes)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XDisplayKeycodes");
    _glfw.x11.xlib.EventsQueued = (PFN_XEventsQueued)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XEventsQueued");
    _glfw.x11.xlib.FilterEvent = (PFN_XFilterEvent)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XFilterEvent");
    _glfw.x11.xlib.FindContext = (PFN_XFindContext)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XFindContext");
    _glfw.x11.xlib.Flush = (PFN_XFlush)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XFlush");
    _glfw.x11.xlib.Free = (PFN_XFree)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XFree");
    _glfw.x11.xlib.FreeColormap = (PFN_XFreeColormap)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XFreeColormap");
    _glfw.x11.xlib.FreeCursor = (PFN_XFreeCursor)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XFreeCursor");
    _glfw.x11.xlib.FreeEventData = (PFN_XFreeEventData)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XFreeEventData");
    _glfw.x11.xlib.GetErrorText = (PFN_XGetErrorText)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetErrorText");
    _glfw.x11.xlib.GetEventData = (PFN_XGetEventData)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetEventData");
    _glfw.x11.xlib.GetICValues = (PFN_XGetICValues)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetICValues");
    _glfw.x11.xlib.GetIMValues = (PFN_XGetIMValues)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetIMValues");
    _glfw.x11.xlib.GetInputFocus = (PFN_XGetInputFocus)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetInputFocus");
    _glfw.x11.xlib.GetKeyboardMapping = (PFN_XGetKeyboardMapping)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetKeyboardMapping");
    _glfw.x11.xlib.GetScreenSaver = (PFN_XGetScreenSaver)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetScreenSaver");
    _glfw.x11.xlib.GetSelectionOwner = (PFN_XGetSelectionOwner)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetSelectionOwner");
    _glfw.x11.xlib.GetVisualInfo = (PFN_XGetVisualInfo)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetVisualInfo");
    _glfw.x11.xlib.GetWMNormalHints = (PFN_XGetWMNormalHints)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetWMNormalHints");
    _glfw.x11.xlib.GetWindowAttributes = (PFN_XGetWindowAttributes)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetWindowAttributes");
    _glfw.x11.xlib.GetWindowProperty = (PFN_XGetWindowProperty)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGetWindowProperty");
    _glfw.x11.xlib.GrabPointer = (PFN_XGrabPointer)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XGrabPointer");
    _glfw.x11.xlib.IconifyWindow = (PFN_XIconifyWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XIconifyWindow");
    _glfw.x11.xlib.InternAtom = (PFN_XInternAtom)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XInternAtom");
    _glfw.x11.xlib.LookupString = (PFN_XLookupString)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XLookupString");
    _glfw.x11.xlib.MapRaised = (PFN_XMapRaised)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XMapRaised");
    _glfw.x11.xlib.MapWindow = (PFN_XMapWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XMapWindow");
    _glfw.x11.xlib.MoveResizeWindow = (PFN_XMoveResizeWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XMoveResizeWindow");
    _glfw.x11.xlib.MoveWindow = (PFN_XMoveWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XMoveWindow");
    _glfw.x11.xlib.NextEvent = (PFN_XNextEvent)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XNextEvent");
    _glfw.x11.xlib.OpenIM = (PFN_XOpenIM)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XOpenIM");
    _glfw.x11.xlib.PeekEvent = (PFN_XPeekEvent)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XPeekEvent");
    _glfw.x11.xlib.Pending = (PFN_XPending)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XPending");
    _glfw.x11.xlib.QueryExtension = (PFN_XQueryExtension)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XQueryExtension");
    _glfw.x11.xlib.QueryPointer = (PFN_XQueryPointer)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XQueryPointer");
    _glfw.x11.xlib.RaiseWindow = (PFN_XRaiseWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XRaiseWindow");
    _glfw.x11.xlib.RegisterIMInstantiateCallback = (PFN_XRegisterIMInstantiateCallback)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XRegisterIMInstantiateCallback");
    _glfw.x11.xlib.ResizeWindow = (PFN_XResizeWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XResizeWindow");
    _glfw.x11.xlib.ResourceManagerString = (PFN_XResourceManagerString)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XResourceManagerString");
    _glfw.x11.xlib.SaveContext = (PFN_XSaveContext)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSaveContext");
    _glfw.x11.xlib.SelectInput = (PFN_XSelectInput)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSelectInput");
    _glfw.x11.xlib.SendEvent = (PFN_XSendEvent)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSendEvent");
    _glfw.x11.xlib.SetClassHint = (PFN_XSetClassHint)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetClassHint");
    _glfw.x11.xlib.SetErrorHandler = (PFN_XSetErrorHandler)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetErrorHandler");
    _glfw.x11.xlib.SetICFocus = (PFN_XSetICFocus)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetICFocus");
    _glfw.x11.xlib.SetIMValues = (PFN_XSetIMValues)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetIMValues");
    _glfw.x11.xlib.SetInputFocus = (PFN_XSetInputFocus)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetInputFocus");
    _glfw.x11.xlib.SetLocaleModifiers = (PFN_XSetLocaleModifiers)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetLocaleModifiers");
    _glfw.x11.xlib.SetScreenSaver = (PFN_XSetScreenSaver)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetScreenSaver");
    _glfw.x11.xlib.SetSelectionOwner = (PFN_XSetSelectionOwner)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetSelectionOwner");
    _glfw.x11.xlib.SetWMHints = (PFN_XSetWMHints)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetWMHints");
    _glfw.x11.xlib.SetWMNormalHints = (PFN_XSetWMNormalHints)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetWMNormalHints");
    _glfw.x11.xlib.SetWMProtocols = (PFN_XSetWMProtocols)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSetWMProtocols");
    _glfw.x11.xlib.SupportsLocale = (PFN_XSupportsLocale)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSupportsLocale");
    _glfw.x11.xlib.Sync = (PFN_XSync)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XSync");
    _glfw.x11.xlib.TranslateCoordinates = (PFN_XTranslateCoordinates)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XTranslateCoordinates");
    _glfw.x11.xlib.UndefineCursor = (PFN_XUndefineCursor)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XUndefineCursor");
    _glfw.x11.xlib.UngrabPointer = (PFN_XUngrabPointer)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XUngrabPointer");
    _glfw.x11.xlib.UnmapWindow = (PFN_XUnmapWindow)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XUnmapWindow");
    _glfw.x11.xlib.UnsetICFocus = (PFN_XUnsetICFocus)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XUnsetICFocus");
    _glfw.x11.xlib.VisualIDFromVisual = (PFN_XVisualIDFromVisual)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XVisualIDFromVisual");
    _glfw.x11.xlib.WarpPointer = (PFN_XWarpPointer)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XWarpPointer");
    _glfw.x11.xkb.FreeKeyboard = (PFN_XkbFreeKeyboard)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbFreeKeyboard");
    _glfw.x11.xkb.FreeNames = (PFN_XkbFreeNames)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbFreeNames");
    _glfw.x11.xkb.GetMap = (PFN_XkbGetMap)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbGetMap");
    _glfw.x11.xkb.GetNames = (PFN_XkbGetNames)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbGetNames");
    _glfw.x11.xkb.GetState = (PFN_XkbGetState)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbGetState");
    _glfw.x11.xkb.KeycodeToKeysym = (PFN_XkbKeycodeToKeysym)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbKeycodeToKeysym");
    _glfw.x11.xkb.QueryExtension = (PFN_XkbQueryExtension)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbQueryExtension");
    _glfw.x11.xkb.SelectEventDetails = (PFN_XkbSelectEventDetails)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbSelectEventDetails");
    _glfw.x11.xkb.SetDetectableAutoRepeat = (PFN_XkbSetDetectableAutoRepeat)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XkbSetDetectableAutoRepeat");
    _glfw.x11.xrm.DestroyDatabase = (PFN_XrmDestroyDatabase)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XrmDestroyDatabase");
    _glfw.x11.xrm.GetResource = (PFN_XrmGetResource)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XrmGetResource");
    _glfw.x11.xrm.GetStringDatabase = (PFN_XrmGetStringDatabase)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XrmGetStringDatabase");
    _glfw.x11.xrm.UniqueQuark = (PFN_XrmUniqueQuark)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XrmUniqueQuark");
    _glfw.x11.xlib.UnregisterIMInstantiateCallback = (PFN_XUnregisterIMInstantiateCallback)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "XUnregisterIMInstantiateCallback");
    _glfw.x11.xlib.utf8LookupString = (PFN_Xutf8LookupString)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "Xutf8LookupString");
    _glfw.x11.xlib.utf8SetWMProperties = (PFN_Xutf8SetWMProperties)
        _glfwPlatformGetModuleSymbol(_glfw.x11.xlib.handle, "Xutf8SetWMProperties");

    if (_glfw.x11.xlib.utf8LookupString && _glfw.x11.xlib.utf8SetWMProperties)
        _glfw.x11.xlib.utf8 = GLFW_TRUE;

    _glfw.x11.screen = DefaultScreen(_glfw.x11.display);
    _glfw.x11.root = RootWindow(_glfw.x11.display, _glfw.x11.screen);
    _glfw.x11.context = XUniqueContext();

    getSystemContentScale(&_glfw.x11.contentScaleX, &_glfw.x11.contentScaleY);

    if (!createEmptyEventPipe())
        return GLFW_FALSE;

    if (!initExtensions())
        return GLFW_FALSE;

    _glfw.x11.helperWindowHandle = createHelperWindow();
    _glfw.x11.hiddenCursorHandle = createHiddenCursor();

    if (XSupportsLocale() && _glfw.x11.xlib.utf8)
    {
        XSetLocaleModifiers("");

        // If an IM is already present our callback will be called right away
        XRegisterIMInstantiateCallback(_glfw.x11.display,
                                       NULL, NULL, NULL,
                                       inputMethodInstantiateCallback,
                                       NULL);
    }

    _glfwPollMonitorsX11();
    return GLFW_TRUE;
}
```