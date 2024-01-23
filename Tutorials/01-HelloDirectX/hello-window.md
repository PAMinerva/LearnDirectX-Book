# Hello Window

![Hello Window](images/A/HelloWindow.png)

## Introduction
DirectX provides a set of APIs that can be used to create games and graphics applications. Specifically, it includes support for high-performance 2-D and 3-D graphics, audio, arithmetic, and linear algebra operations. Below is a list of the main APIs included in DirectX. However, our primary focus is on Direct3D 12 and DirectXMath, the only ones we will be using for a while.

<br>

**Direct3D** provides functionality for performing 3-D graphics rendering tasks. It is used to draw primitives (i.e., points, lines, and triangles) within the rendering pipeline or to start parallel operations on the GPU. When we refer to Direct3D 12, we are specifically talking about the API that enables apps to leverage the graphics and computing capabilities of PCs equipped with a DirectX 12-compatible GPU.

**Direct2D** offers functionality for rendering 2-D geometries, bitmaps, and text. It is designed to interoperate with existing code that uses Direct3D to create 2D menus, user interface (UI) elements, and Heads-up Displays (HUDs).

**DirectWrite** provides support for high-quality text rendering, resolution-independent outline fonts, and full Unicode text and layouts. It is designed to interoperate with Direct2D to render text by taking advantage of hardware acceleration. Text can also be filled with an arbitrary Direct2D brush, such as radial gradients, linear gradients, and bitmaps.

**DirectXMath** provides types and helper functions for common linear algebra and graphics math operations that are frequently used in DirectX applications.

**XAudio2** allows the addition of sound effects and background music, or the development of high-performance audio engines.

**XInput** enables applications to receive input from the Xbox Controller when it is connected to a Windows PC.

<br>

To create graphics applications, you first need a window to draw on. Therefore, the aim of this tutorial is to create and display a simple window on your screen. For this purpose, we will examine the [D3D12HelloWindow](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld) sample, which is part of the [DirectX-Graphics-Samples](https://github.com/microsoft/DirectX-Graphics-Samples) repository maintained by Microsoft. The only significant graphics operation performed by this sample is the setting of the window background color. You might be surprised to discover that you need to write a substantial amount of code to execute this simple operation. The good news is that the code we will review in this first tutorial primarily consists of boilerplate code. This means that, by the end of this tutorial, you will have a basic understanding of the common framework used by almost all samples we will examine in the upcoming tutorials, so we can solely focus on new additions.

Before starting to review the source code of the [D3D12HelloWindow](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld) sample, you need a basic understanding of the Component Object Model (COM), the DXGI API, as well as how Windows applications work. You can skip the following three sections if you are comfortable with these topics. Alternatively, you can also refer to {cite}`AboutWindows,ProgrammingDirectXWithCOM,DXGIOverview` for further information.

<br>

## Windows applications
This section is heavily inspired by the first chapter of the book “Programming Microsoft Visual C++, Fifth Edition” by David J. Kruglinski, George Shepherd and Scott Wingo.

Windows applications use an event-driven programming model, as illustrated in the following below. In this model, programs respond to events by processing messages sent by the operating system. An event could be a keystroke, a mouse click, or a command for a window to repaint itself. The entry point of a Windows application is a function called **WinMain**, but most of the action occurs in a function known as the window procedure. The window procedure processes messages sent by the OS to the application that a window belongs to. **WinMain** creates that window and then enters a message loop, retrieving messages and dispatching them to the window procedure. Messages wait in a message queue until they are retrieved. The primary task of a Windows application is to respond to the messages it receives. In between messages, it does little except wait for the next message to arrive. An application can exit the message loop when a **WM_QUIT** message is retrieved from the message queue, signaling that the application is about to end. This message is sent by the OS when the user closes the window. When the message loop ends, **WinMain** returns, and the application terminates.

<br>

![Image](images/A/rect31572.png)

<br>

```{note}
Window messages can also be sent directly to a window procedure, bypassing the message queue. If the sending thread is dispatching a message to a window created by the same thread, the window procedure of the specified window is invoked. However, if a thread is dispatching a message to a window created by a different thread, the process becomes more complex. Fortunately, we don't need to delve into the low-level details in this tutorial series.
```

### Window Procedure

As previously mentioned, a window procedure is a function that receives and processes messages sent by the operating system to the application that a window belongs to. A window class defines key characteristics of a window, such as its window procedure address, its default background color, and its icon. Every window created with a specific class will use the same window procedure to respond to messages.

When the application dispatches a message to a window procedure, it also passes additional information about the message as arguments in its input parameters. This allows the window procedure to perform an appropriate action for a message by consuming the related message data. If a window procedure does not process a message, it must return the message to the system for default processing by calling the **DefWindowProc** function, which performs a default action and returns a message result. The window procedure must then return this value as its own message result.

Since a window procedure is shared by all windows belonging to the same class, it can process messages for different windows. To identify the specific window a message is addressed to, a window procedure can examine the window handle passed as an input parameter. The code provided in the window procedure to process a particular message is known as a message handler.


### Messages

Windows defines many different message types. Usually, messages have names that begin with the letters "WM_", as in **WM_CREATE** and **WM_PAINT**. The following table shows ten of the most common messages.

<br>

| Message        | Sent when                                                                      |
| -------------- | ------------------------------------------------------------------------------ |
| WM_CHAR        | A character is input from the keyboard.                                        |
| WM_COMMAND     | The user selects a menu item, or a control sends a notification to its parent. |
| WM_CREATE      | A window is created.                                                           |
| WM_DESTROY     | A window is destroyed.                                                         |
| WM_LBUTTONDOWN | The left mouse button is pressed.                                              |
| WM_LBUTTONUP   | The left mouse button is released.                                             |
| WM_MOUSEMOVE   | The mouse pointer is moved.                                                    |
| WM_PAINT       | A window needs repainting.                                                     |
| WM_QUIT        | The application is about to terminate.                                         |
| WM_SIZE        | A window is resized.                                                           |

<br>

For example, a window receives a **WM_PAINT** message when its interior needs repainting. You can think of a Windows program as a collection of message handlers.

When the message loop dispatches a message, the window procedure is called, and you can retrieve the information on the message from its four input parameters:

* The handle of the window to which the message is directed,

* A message ID, and

* Two 32-bit parameters known as **wParam** and **lParam**.

<br>

The window handle is a 32-bit value that uniquely identifies a window. Internally, the value references a data structure in which the OS stores relevant information about the window such as its size, style, and location on the screen.

The message ID is a numeric value that identifies the message type: **WM_CREATE**, **WM_PAINT**, and so on.

**wParam** and **lParam** contain information specific to the message type. For example, when a **WM_LBUTTONDOWN** message arrives, **wParam** holds a series of bit flags identifying the state of the <kbd>Ctrl</kbd> and <kbd>Shift</kbd> keys and of the mouse buttons. **lParam** holds two 16-bit values identifying the location of the mouse pointer (in screen coordinates) when the click occurred. At that point, you have all you need to know to process the **WM_LBUTTONDOWN** message in the window procedure. Conventionally, **WinMain** should return the value stored in the **wParam** of the **WM_QUIT** message.

The only criticism to the above explanation is that, unlike typical window applications, graphics applications perform the majority of their processing in between messages. However, the [D3D12HelloWindow](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld) sample is an exception as its sole purpose is to display a window on the screen (i.e., no significant graphics operations are involved).

<br>

## Component Object Model (COM)

Microsoft used the Component Object Model (COM) to design the internals of DirectX. Therefore, whenever you program with DirectX, you are also implicitly using COM. This is an object-oriented programming model created by Microsoft to break dependencies of the code at the binary level. This implies that if an API, a framework, or a generic technology is built upon COM, then it will be language-independent and backward-compatible, to a certain extent.

```{note}
Unfortunately, this doesn't automatically mean you can write DirectX applications using any programming language you want, or that you can run a DX12 application with older libraries and runtime (DX11 or earlier).
```

COM is a complex programming model, but fortunately, you don't need to master it to write DirectX applications. Indeed, we will only use COM as end-users rather than for developing our API or framework. That is, DirectX will hide the complexity of COM from us. However, to effectively program with DirectX, we still need to know some basic concepts about COM. First, it can be useful to understand what it means to break dependencies of the code at the binary level and what type of problems this break can solve.

If you've ever developed a Windows library, you're likely familiar with the process of exporting functionality from DLLs written in the C language for use by applications written in other languages (such as C++, C#, Java, Python, etc.). Microsoft didn't use C to write DirectX, though. They preferred an object-oriented language like C++. Now, consider the scenario of writing a DLL that exports a C++ class. The functionality provided by this class can't be easily used by other languages because C++ only specifies what happens at the source code level. The standard doesn't say anything about what happens at the binary level. For example, we know that object-oriented languages use virtual tables to implement polymorphism. However, this is an implementation concept, just like the stack and the heap: the C++ standard doesn't say anything about how to implement polymorphism. The following image shows a common layout for a class in memory. However, nothing prevents a new language from placing the virtual table pointer at the end, or defining a whole new system to implement polymorphism.

<br>

![Image](images/A/class-layout2.png)

<br>

In other words, using a C++ class exported from a DLL is feasible as long as you are operating on the same OS and with the same compiler. Conversely, other languages or different implementations of C++ might struggle to communicate with the DLL if they lack knowledge about the binary layout of the exported class in memory. Specifically, when a compiler attempts to resolve a call to a virtual function, it requires knowledge of the class's memory layout to access the function in the virtual table.

Even if you were able to use a C++ class exported from a DLL, one problem still remains. Typically, the DLL developer provides an include file with the class declaration. Now, consider using this include file to compile an application that creates an instance of the exported class as a local variable on the stack. Additionally, assume you place the DLL in the executable directory of your application. Also, suppose that you put the DLL in the executable directory of your application. After a while, the developer releases an updated version of their DLL, and you choose to overwrite the old one in the executable directory without recompiling your app as the notes of the developer indicate they only added a private member in the exported class. Indeed, C++ rules state that everything should be fine as the public part has not changed. The problem is that this statement is only valid at the source level, not at the binary level. If you now execute your app, the new DLL is loaded in memory and the new constructor of the exported class is invoked to initialize the new private member. However, you haven't recompiled your app, so the space reserved for the local variable on the stack is the one specified by the definition of the class in the old include file you used to compile the first time. You can easily imagine how this can lead to incorrect results, or even worse, crashes.

These problems arise from the fact that the binary representation of the DLL is exposed to the app. COM try to resolve this inconvenience with few fundamental principles:

* Clients (apps) communicate with servers (DLLs) using abstract interfaces instead of concrete classes. If a server exports a class (called COM class) that implements an interface, a client can reference an instance of the COM class (called COM object) through an interface pointer and use it to call the member methods exported by the COM class.

* Clients create COM objects using methods implemented in the servers. That way, the implementation of the COM class is hidden from the clients entirely. Only the server knows how to create a COM object, so if the private part of the COM class changes, the client is not affected because the interface is still the same — interfaces don't contain data members.

* COM classes and interfaces have unique IDs. That way it's possible for more than a server to implement the same interface, and for a client to load the correct server (DLL) in memory. So, if there are more versions of the DLL available, the client can choose what server to load. Then, if your application wants to use new functionalities, you are forced to recompile.

<br>

However, even though COM involves the use of abstract interfaces, client and server still need to agree on the binary representation of these interfaces for effective communication. To address this, the COM specification outlines a binary object layout that can be implemented and comprehended by nearly any language and platform. Notably, Microsoft opted to employ a virtual table mechanism similar to the one used in their C++ implementation. Essentially, a COM interface in memory is nothing more than a virtual table containing function pointers and additional data. Consequently, an interface pointer to a COM object is merely a pointer to a virtual table.

In essence, for a language or compiler to support COM, it must adhere to the layout of COM interfaces as specified by the COM specification. This requirement highlights one of the reasons why you may not always be able to use DirectX with your preferred programming language.

As mentioned earlier, clients cannot directly create COM objects. Typically, we use a method like **CoCreateInstance**, specifying the COM class ID for which the client wants to create a COM object, and the interface ID implemented by the COM class that the client is interested in obtaining a pointer to. The client doesn't need to know where the server is located. The Windows Registry is used for this purpose, and **CoCreateInstance** (with the help of a system service) can locate the server based on the arguments passed as parameters. At this point, the server can create the COM object, and an interface pointer to that object is returned to the client, which can use it to communicate with the server (i.e., call its member functions).

However, you would rarely use **CoCreateInstance** to directly create DirectX COM objects. Typically, we will create COM objects indirectly by using specific DirectX methods that return pointers (as output parameters) to whatever interface is implemented by the related COM classes. Although this mechanism is less centralized (i.e., it doesn't rely on the Windows Registry to locate servers), it functions similarly. Generally, the functions to create DirectX COM objects return an **HRESULT**, an encoded value indicating the success or failure of the operation.

At this point we can better define the meaning of backward compatibility mentioned at the beginning of this section. In short, a DirectX application can run on a system provided that the servers with the COM classes used by the client can be loaded. So, as long as new versions of a DLL don't change the COM classes to include new disruptive functionalities that modify the related COM interfaces, the application can still load the new DLL and use the new COM classes without problems.

before creating any COM object, you should initialize the COM library by calling **CoInitializeEx**. However, when you create COM objects indirectly, the creation methods will handle this task for you. We will see many examples of such methods in the upcoming tutorials.

COM defines a base interface that all other interfaces must extend: **IUnknown**. This interface defines some basic operations:

* **AddRef** increments the reference count for an interface pointer to a COM object. You should call this method whenever you make a copy of an interface pointer.

* **Release** decrements the reference count for an interface on a COM object. When the reference count on an object reaches zero, this method must cause the interface pointer to free itself.

* **QueryInterface** queries a COM object for a pointer to one of its interfaces; identifying the interface by a reference to its interface identifier (IID). If the COM object implements the interface, then it returns a pointer to that interface after calling **AddRef** on it.

<br>

Directly managing interface pointers to COM objects can be a challenging task, as you need to explicitly call **Release** and **AddRef** to maintain the reference count. A more convenient solution with C++ is to use smart pointers. **Microsoft::WRL::ComPtr** is a smart pointer provided by the Windows Runtime C++ Template Library (WRL). This library is "pure" C++, making it suitable for classic Win32 desktop applications. It automatically calls **AddRef** and **Release** on the underlying interface pointer, meaning it maintains a reference count for the underlying interface pointer and releases the interface pointer when the reference count drops to zero. Moreover, it defines various other methods, including:

* **Get** returns the underlying interface pointer to a COM object. It's especially useful when you have functions that accept a raw interface pointer instead of a **ComPtr**.

* **GetAddressOf** returns a pointer to the underlying interface pointer to a COM object. It's especially useful because, whenever you create a COM object indirectly, the default is to return the interface pointer as an output parameter (i.e., as a pointer to an interface pointer).

* **Reset** calls **Release** on the underlying interface pointer to a COM object and then set it to **nullptr**. It can be useful when you don't need the underlying interface pointer anymore, but the **ComPtr** hasn't gone out of scope yet.

* **ReleaseAndGetAddressOf** is similar to **Reset** but it returns the underlying pointer (that will be a **nullptr**) to the caller. It can be useful if you want a new initialization for an underlying interface pointer you don't need anymore (e.g., when you want to pass it as an argument to the output parameter of a creation method).

* **Detach** returns the underlying interface pointer to a COM object and then set it to **nullptr**. It can be useful if you need to return the underlying interface pointer to a caller and, at the same time, you don't need the local **ComPtr** object anymore.

* **As** is just a wrapper around **QueryInterface**. It takes another **ComPtr** as input parameter to get a pointer to one of the interfaces implemented by a COM object.

<br>

Also, the dereference operator -> is overloaded and returns the underlying interface pointer to a COM object, so that you don't need to call **Get** if you only want to invoke a function through the interface pointer.

<br>

## DirectX Graphics Infrastructure (DXGI)

Microsoft DirectX Graphics Infrastructure (DXGI) is an API that collects functionality and tasks that don't change regardless of the version of graphics API you are actually using (Direct3D 10, 11, or 12). Specifically, DXGI manages low-level tasks such as enumerating hardware graphics devices (GPUs) and outputs (monitors), creating rendering buffers, presenting rendered frames to an output, controlling gamma, and managing full-screen transitions. This allows a graphics API to focus on drawing 3D content into buffers without worrying about the origin of these buffers or how they will be displayed. <br>
DXGI's purpose is to communicate with the kernel mode driver and the system hardware, as shown in the following diagram.

<br>

![Image](images/A/rect92360.png)

<br>

A graphics application can either access DXGI directly or use the Direct3D API, which manages communications with DXGI. You might prefer to interact with DXGI directly if your application needs to enumerate devices or control how data is presented to an output.

An adapter is an abstraction of a hardware or software device. Typically, there are multiple adapters on a machine. Some devices are implemented in hardware, such as a video card, while others are implemented in software, like the Direct3D rasterizer provided by Microsoft. The following diagram illustrates a system with a single computer, two adapters (video cards), and three output monitors.

<br>

![Image](images/A/dxgi-adapter-output.png)

<br>

The primary task of your graphics applications is to draw on buffers and ask DXGI to present those buffers as frames to the output. If the application has two buffers available, it can render on one buffer (the render target) while presenting another one. Depending on the time it takes to render a frame, or the desired frame rate for presentation, the application may need more than two buffers. The collection of buffers created is referred to as a swap chain, as depicted in the following illustration.

<br>

![Image](images/A/dxgi-swap-chain.png)

<br>

A swap chain consists of one front (or present) buffer and one or more back buffers, which are used as render targets. Each application creates its own swap chain. To maximize the speed of data presentation to an output, a swap chain is almost always created in GPU memory. DXGI, with the assistance of the kernel driver, is responsible for scanning rendered content in the front buffer from video memory and presenting it on outputs.

A swap chain can be configured for drawing in either full-screen or windowed mode, eliminating the need to determine whether an output is windowed or full screen. A full-screen mode swap chain can optimize performance by switching the display resolution. An output can support one or more display modes, which include resolution, refresh rate, format, etc. DXGI might change the display mode of an output when making a full-screen transition. However, resizing swap chain buffers will not trigger a display mode switch. The swap chain makes an implicit promise that if you choose a back buffer that exactly matches a display mode supported by the target output, then it will switch to that display mode when entering full-screen mode on that output. Consequently, you actually select a display mode by choosing your back buffer size and format.

<br>

## Framework overview

As mentioned earlier in this tutorial, the framework used for building the [D3D12HelloWindow](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld) sample is common to nearly all the other samples we'll explore in the upcoming tutorials. This means that, by the end of this tutorial, you will know how to write a generic DirectX application, or at least the backbone of a complete graphics application.

As you can see in {numref}`project-props` below, the Direct3D 12 and DXGI import libraries (LIB files) are listed in the additional dependencies of the project. The information stored in these files will help the linker resolve references to functions exported by the corresponding DLLs. Also, observe that the Direct3D 12 DLL will not be loaded when the application starts, but only the first time we call an exported function. This approach improves performance by only loading DLLs when they're needed, which can reduce startup time and improve memory usage.

<br>

```{figure} images/A/project-properties.PNG
---
name: project-props
---
Project property pages in Visual Studio
```
<br>

DirectX applications are normal Windows programs, so the entry point is **WinMain** as usual.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/Main.cpp
:name: winmain-code
#include "stdafx.h"
#include "D3D12HelloWindow.h"
 
_Use_decl_annotations_
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE, LPSTR, int nCmdShow)
{
    D3D12HelloWindow sample(1280, 720, L"D3D12 Hello Window");
    return Win32Application::Run(&sample, hInstance, nCmdShow);
}
```
<br>

{numref}`stdafx-code` show the *stdafx.h* header, which includes other header files associated with various DirectX libraries, such as Direct3D 12, DirectXMath, and DXGI. We need to include *wrl.h* to use the smart pointers provided by the Windows Template Library. *d3dx12.h* defines helpful structures that act as C++ wrappers around Direct3D 12 native structures, simplifying their initialization. Additionally, this header provides helper functions that make handling subresources more straightforward. *D3DCompiler.h* is the header associated with a library we will use to compile shader code. The concepts of subresources and shader code will be covered in upcoming tutorials.

```{seealso}
**_Use_decl_annotations_** is a macro that simplifies SAL annotations. We won't go into detail about this concept, as it's not relevant for this tutorial or the rest of the series. If you want to learn more, check out the official Microsoft documentation: see {cite}`UsingSALAnnotations`.
```
<br>

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/stdafx.h
:name: stdafx-code
#include <windows.h>
 
#include <d3d12.h>
#include <dxgi1_6.h>
#include <D3Dcompiler.h>
#include <DirectXMath.h>
#include "d3dx12.h"
 
#include <string>
#include <wrl.h>
#include <shellapi.h>
```
<br>

**WinMain** is called by the C/C++ runtime startup and takes four parameters. However, we are only interested in the named ones in {numref}`winmain-code`. <br>
**hInstance** represents the base virtual address of the executable file loaded into memory. This information is primarily used by the operating system to identify the application and manage some of its resources. <br>
**nCmdShow** is a flag that indicates whether the main application window is minimized, maximized, or shown normally. We'll pass this flag to a Windows function that's responsible for showing windows (we'll discuss this in more detail in the next section).

<br>

The **D3D12HelloWindow** class is the application class, which defines data and methods specific to the sample.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/D3D12HelloWindow.h
:name: D3D12HelloWindow-code
class D3D12HelloWindow : public DXSample
{
public:
    D3D12HelloWindow(UINT width, UINT height, std::wstring name);
 
    virtual void OnInit();
    virtual void OnUpdate();
    virtual void OnRender();
    virtual void OnDestroy();
 
private:
    static const UINT FrameCount = 2;
 
    // Pipeline objects.
    ComPtr<IDXGISwapChain3> m_swapChain;
    ComPtr<ID3D12Device> m_device;
    ComPtr<ID3D12Resource> m_renderTargets[FrameCount];
    ComPtr<ID3D12CommandAllocator> m_commandAllocator;
    ComPtr<ID3D12CommandQueue> m_commandQueue;
    ComPtr<ID3D12DescriptorHeap> m_rtvHeap;
    ComPtr<ID3D12PipelineState> m_pipelineState;
    ComPtr<ID3D12GraphicsCommandList> m_commandList;
    UINT m_rtvDescriptorSize;
 
    // Synchronization objects.
    UINT m_frameIndex;
    HANDLE m_fenceEvent;
    ComPtr<ID3D12Fence> m_fence;
    UINT64 m_fenceValue;
 
    void LoadPipeline();
    void LoadAssets();
    void PopulateCommandList();
    void WaitForPreviousFrame();
};
```
<br>

The **DXSample** base class defines data and methods used by all graphics samples.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/DXSample.h
:name: DXSample-code
class DXSample
{
public:
    DXSample(UINT width, UINT height, std::wstring name);
    virtual ~DXSample();
 
    virtual void OnInit() = 0;
    virtual void OnUpdate() = 0;
    virtual void OnRender() = 0;
    virtual void OnDestroy() = 0;
 
    // Samples override the event handlers to handle specific messages.
    virtual void OnKeyDown(UINT8 /*key*/)   {}
    virtual void OnKeyUp(UINT8 /*key*/)     {}
 
    // Accessors.
    UINT GetWidth() const           { return m_width; }
    UINT GetHeight() const          { return m_height; }
    const WCHAR* GetTitle() const   { return m_title.c_str(); }
 
    void ParseCommandLineArgs(_In_reads_(argc) WCHAR* argv[], int argc);
 
protected:
    std::wstring GetAssetFullPath(LPCWSTR assetName);
 
    void GetHardwareAdapter(
        _In_ IDXGIFactory1* pFactory,
        _Outptr_result_maybenull_ IDXGIAdapter1** ppAdapter,
        bool requestHighPerformanceAdapter = false);
 
    void SetCustomWindowText(LPCWSTR text);
 
    // Viewport dimensions.
    UINT m_width;
    UINT m_height;
    float m_aspectRatio;
 
    // Adapter info.
    bool m_useWarpDevice;
 
private:
    // Root assets path.
    std::wstring m_assetsPath;
 
    // Window title.
    std::wstring m_title;
};
```
<br>

The **Win32Application** class defines data and methods used by all Windows applications.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/Win32Application.h
:name: Win32Application-code
class Win32Application
{
public:
    static int Run(DXSample* pSample, HINSTANCE hInstance, int nCmdShow);
    static HWND GetHwnd() { return m_hwnd; }
 
protected:
    static LRESULT CALLBACK WindowProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam);
 
private:
    static HWND m_hwnd;
};
```
<br>

```{note}
It's perfectly fine if you don't understand the meaning of every single class member at this point. The important thing is to focus on comprehending the overall structure and purpose of the classes. In the next section and in later tutorials, I will provide more detailed explanations for each of the class members.
```
<br>

As you might have noticed in {numref}`winmain-code`, the entry point (**WinMain**) creates an instance of the **D3D12HelloWindow** class by calling its constructor.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/D3D12HelloWindow.cpp
:name: D3D12HelloWindow-ctor-code
D3D12HelloWindow::D3D12HelloWindow(UINT width, UINT height, std::wstring name) :
    DXSample(width, height, name),
    m_frameIndex(0),
    m_rtvDescriptorSize(0)
{
}
```
<br>

The constructor of the **D3D12HelloWindow** class initializes some of the class's data members to default values and invokes the constructor of its base class (**DXSample**).

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/DXSample.cpp
:name: DXSample-ctor-code
DXSample::DXSample(UINT width, UINT height, std::wstring name) :
    m_width(width),
    m_height(height),
    m_title(name),
    m_useWarpDevice(false)
{
    WCHAR assetsPath[512];
    GetAssetsPath(assetsPath, _countof(assetsPath));
    m_assetsPath = assetsPath;
 
    m_aspectRatio = static_cast<float>(width) / static_cast<float>(height);
}
```
<br>

The constructor of the DXSample cass initializes the sample's name and the dimensions of the window's client area, where the rendering will take place.

The **GetAssetsPath** function returns the absolute path of the executable. This is where the application will search for resource files (shaders, textures, etc.) required to run the sample. However, for this initial sample, we don't need such resources, so you won't find anything in the executable directory, except for the executable itself, of course.

The aspect ratio refers to the proportional relationship between the width and height of the window's client area.

<br>

![Image](images/A/win-client-area.png)

<br>

The client area is the region of a window where drawing is allowed. Technically speaking, it's the area where the render target is mapped (once the GPU finishes drawing a frame on it). Conceptually, you can consider a render target as a texture that the GPU utilizes for rendering\drawing operations.

<br>

In {numref}`winmain-code`, **Win32Application::Run** is called passing the instance of the **D3D12HelloWindow** class, along with the named parameters of **WinMain**.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/Win32Application.cpp
:name: run-code
int Win32Application::Run(DXSample* pSample, HINSTANCE hInstance, int nCmdShow)
{
    // Parse the command line parameters
    int argc;
    LPWSTR* argv = CommandLineToArgvW(GetCommandLineW(), &argc);
    pSample->ParseCommandLineArgs(argv, argc);
    LocalFree(argv);
 
    // Initialize the window class.
    WNDCLASSEX windowClass = { 0 };
    windowClass.cbSize = sizeof(WNDCLASSEX);
    windowClass.style = CS_HREDRAW | CS_VREDRAW;
    windowClass.lpfnWndProc = WindowProc;
    windowClass.hInstance = hInstance;
    windowClass.hCursor = LoadCursor(NULL, IDC_ARROW);
    windowClass.lpszClassName = L"DXSampleClass";
    RegisterClassEx(&windowClass);
 
    RECT windowRect = { 0, 0, static_cast<LONG>(pSample->GetWidth()), static_cast<LONG>(pSample->GetHeight()) };
    AdjustWindowRect(&windowRect, WS_OVERLAPPEDWINDOW, FALSE);
 
    // Create the window and store a handle to it.
    m_hwnd = CreateWindow(
        windowClass.lpszClassName,
        pSample->GetTitle(),
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT,
        CW_USEDEFAULT,
        windowRect.right - windowRect.left,
        windowRect.bottom - windowRect.top,
        nullptr,        // We have no parent window.
        nullptr,        // We aren't using menus.
        hInstance,
        pSample);
 
    // Initialize the sample. OnInit is defined in each child-implementation of DXSample.
    pSample->OnInit();
 
    ShowWindow(m_hwnd, nCmdShow);
 
    // Main sample loop.
    MSG msg = {};
    while (msg.message != WM_QUIT)
    {
        // Process any messages in the queue.
        if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
    }
 
    pSample->OnDestroy();
 
    // Return this part of the WM_QUIT message to Windows.
    return static_cast<char>(msg.wParam);
}
```
<br>

Creating a window requires an instance of a window class (represented by the **WNDCLASSEX** structure), which defines essential attributes for all windows derived from that class. Here's a summary of the most important **WNDCLASSEX** fields.

**style** specifies additional informations about the window's appearance and behavior. For example, `CS_HREDRAW | CS_VREDRAW` instructs the OS to redraw the entire window if a resize operation affects the client area's dimensions.

**hCursor** indicates the cursor displayed when the mouse hovers over the window's client area.

**hInstance** specifies the application to which the window belongs. This information is passed to **WinMain** as the first argument.

**lpszClassName** specifies the name we want to give to the window class.

**lpfnWndProc** specifies the address of the window procedure.

<br>

**RegisterClassEx** registers a window class, allowing us to create multiple windows with the same style, window procedure, and other attributes.

**CreateWindow** creates a window and returns its handle, which is used to identify and manipulate the window. It takes the name of the window class and additional parameters that define the window's characteristics. For more details, refer to the Microsoft documentation for **CreateWindow**

```{note}
The application class instance, passed as an argument to **Run**, holds the client area dimensions. However, **CreateWindow** needs the entire window's size, so we must derive it. **AdjustWindowRect** calculates the window's dimensions based on the client area size and the window style. **WS_OVERLAPPEDWINDOW** defines a window with a title bar and no menu.
```

```{note}
The last parameter of **CreateWindow** allows us to provide a pointer that the operating system (OS) will return in response to a **WM_CREATE** message. This message is sent by the OS to an application immediately after creating a window, which is when **CreateWindow** returns. This allows us to store the instance of **DXSample** for later use (more on this shortly).
```

Observe that the **OnInit** method is invoked through the **pSample** pointer before showing the window on the screen to the user.

<br>

## D3D12HelloWindow: code review

The **DXSample::OnInit** method is a virtual function that must be overridden in derived classes. This has been done this in the **D3D12HelloWindow** class. The implementation is shown in {numref}`oninit-code`.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/D3D12HelloWindow.cpp
:name: oninit-code
void D3D12HelloWindow::OnInit()
{
    LoadPipeline();
    LoadAssets();
}
```
<br>

The **OnInit** method simply calls **LoadPipeline** and **LoadAssets**. Let's start examining **LoadPipeline**.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/D3D12HelloWindow.cpp
:name: loadpipeline-code
// Load the rendering pipeline dependencies.
void D3D12HelloWindow::LoadPipeline()
{
    UINT dxgiFactoryFlags = 0;
 
#if defined(_DEBUG)
    // Enable the debug layer (requires the Graphics Tools "optional feature").
    // NOTE: Enabling the debug layer after device creation will invalidate the active device.
    {
        ComPtr<ID3D12Debug> debugController;
        if (SUCCEEDED(D3D12GetDebugInterface(IID_PPV_ARGS(&debugController))))
        {
            debugController->EnableDebugLayer();
 
            // Enable additional debug layers.
            dxgiFactoryFlags |= DXGI_CREATE_FACTORY_DEBUG;
        }
    }
#endif
 
    ComPtr<IDXGIFactory4> factory;
    ThrowIfFailed(CreateDXGIFactory2(dxgiFactoryFlags, IID_PPV_ARGS(&factory)));
 
    if (m_useWarpDevice)
    {
        ComPtr<IDXGIAdapter> warpAdapter;
        ThrowIfFailed(factory->EnumWarpAdapter(IID_PPV_ARGS(&warpAdapter)));
 
        ThrowIfFailed(D3D12CreateDevice(
            warpAdapter.Get(),
            D3D_FEATURE_LEVEL_11_0,
            IID_PPV_ARGS(&m_device)
            ));
    }
    else
    {
        ComPtr<IDXGIAdapter1> hardwareAdapter;
        GetHardwareAdapter(factory.Get(), &hardwareAdapter);
 
        ThrowIfFailed(D3D12CreateDevice(
            hardwareAdapter.Get(),
            D3D_FEATURE_LEVEL_11_0,
            IID_PPV_ARGS(&m_device)
            ));
    }
 
    // Describe and create the command queue.
    D3D12_COMMAND_QUEUE_DESC queueDesc = {};
    queueDesc.Flags = D3D12_COMMAND_QUEUE_FLAG_NONE;
    queueDesc.Type = D3D12_COMMAND_LIST_TYPE_DIRECT;
 
    ThrowIfFailed(m_device->CreateCommandQueue(&queueDesc, IID_PPV_ARGS(&m_commandQueue)));
 
    // Describe and create the swap chain.
    DXGI_SWAP_CHAIN_DESC1 swapChainDesc = {};
    swapChainDesc.BufferCount = FrameCount;
    swapChainDesc.Width = m_width;
    swapChainDesc.Height = m_height;
    swapChainDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
    swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
    swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;
    swapChainDesc.SampleDesc.Count = 1;
 
    ComPtr<IDXGISwapChain1> swapChain;
    ThrowIfFailed(factory->CreateSwapChainForHwnd(
        m_commandQueue.Get(),        // Swap chain needs the queue so that it can force a flush on it.
        Win32Application::GetHwnd(),
        &swapChainDesc,
        nullptr,
        nullptr,
        &swapChain
        ));
 
    // This sample does not support fullscreen transitions.
    ThrowIfFailed(factory->MakeWindowAssociation(Win32Application::GetHwnd(), DXGI_MWA_NO_ALT_ENTER));
 
    ThrowIfFailed(swapChain.As(&m_swapChain));
    m_frameIndex = m_swapChain->GetCurrentBackBufferIndex();
 
    // Create descriptor heaps.
    {
        // Describe and create a render target view (RTV) descriptor heap.
        D3D12_DESCRIPTOR_HEAP_DESC rtvHeapDesc = {};
        rtvHeapDesc.NumDescriptors = FrameCount;
        rtvHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
        rtvHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;
        ThrowIfFailed(m_device->CreateDescriptorHeap(&rtvHeapDesc, IID_PPV_ARGS(&m_rtvHeap)));
 
        m_rtvDescriptorSize = m_device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);
    }
 
    // Create frame resources.
    {
        CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart());
 
        // Create a RTV for each frame.
        for (UINT n = 0; n < FrameCount; n++)
        {
            ThrowIfFailed(m_swapChain->GetBuffer(n, IID_PPV_ARGS(&m_renderTargets[n])));
            m_device->CreateRenderTargetView(m_renderTargets[n].Get(), nullptr, rtvHandle);
            rtvHandle.Offset(1, m_rtvDescriptorSize);
        }
    }
 
    ThrowIfFailed(m_device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&m_commandAllocator)));
}
```
<br>

**LoadPipeline** is responsible for creating some of the objects we need to render (draw) the frames of our sample. Here, for the first time, we meet a method to create a DirectX COM object indirectly: **D3D12GetDebugInterface**. The prototype of this method is as follows:

```{code-block} cpp
:caption: d3d12.h
:name: D3D12GetDebugInterface-code

HRESULT WINAPI D3D12GetDebugInterface( _In_ REFIID riid, _COM_Outptr_opt_ void** ppvDebug );
```
<br>

The first parameter is the ID of the interface to which we want to get a pointer, while the second parameter is where the function will return the interface pointer to the caller (that is, the address where it will store the address of the COM interface; that's why the parameter is a pointer to a pointer). In this case, the first parameter is the ID of the **ID3D12Debug** interface. So, **D3D12GetDebugInterface** creates an instance of the COM object that implements this interface and returns the interface pointer in the second parameter. Then, the caller can use it to call functionality implemented by the related COM class. Often, the macro **IID_PPV_ARGS** is used to reduce typos. This macro is defined as:

```{code-block} cpp
:caption: combaseapi.h
:name: iid-ppv-args-code

#define IID_PPV_ARGS(ppType) __uuidof(**(ppType)), IID_PPV_ARGS_Helper(ppType)
```
<br>

The **__uuidof** operator allows to get IDs of COM interfaces and classes. Usually, we pass the address of a **ComPtr** as an argument to **IID_PPV_ARGS**, so that it must be dereference twice to get the underlying interface pointer, which is used by **__uuidof** to determine the type of the requested object and get the corresponding ID. The helper macro **IID_PPV_ARGS_Helper** simply converts the address of a **ComPtr** to a normal **void**** (take a look at the source code in *combaseapi.h* if you want to see how it is implemented). 

After getting a pointer to the **ID3D12Debug** interface, we use it to enable the Direct3D debug layer. This layer helps during debugging by showing error and warning messages in the output window of the debugger if any obscure rendering error, validation control or memory leak occurs.

```{attention}
**D3D12GetDebugInterface** must be called before the D3D12 device is created. Calling it after creating the D3D12 device will cause the D3D12 runtime to remove the device. The D3D12 runtime refers to a set of functionalities provided by the Direct3D 12 library on which all Direct3D 12 applications depend at run time in order to run as intended. For example, during some critical calls to the DirectX API, the runtime is assumed to handle parameter validation before handing control to user-mode driver, which can assume the parameters are correct.
```

The **IDXGIFactory4** interface allows to create some important DXGI objects (for example, adapters and swap chains), as well as to enumerate adapters and outputs. It also allows to manage full-screen transitions. We use **CreateDXGIFactory2** to get a pointer to **IDXGIFactory4** because this method allows to pass a flag indicating we are also interested in enabling an additional layer to be informed of errors and warnings about DXGI (in addition to those related to Direct3D).

If you have a video card installed on your system, an interface pointer to the related adapter can be obtained by calling the helper function **DXSample::GetHardwareAdapter**. At that point, we can pass it as an argument to **D3D12CreateDevice** to create a device object and get an interface pointer to it. In the context of Direct3D, by device we simply mean a video card, and we use the device object created with **D3D12CreateDevice** to communicate with a specific video card. **D3D_FEATURE_LEVEL_11_0** specifies we are interested in creating a device object for a GPU that supports the basic functionality provided by Direct3D 12. 

```{note}
A feature level is a well-defined set of GPU functionalities. For instance, the 9_1 feature level implements the functionality that was implemented in Microsoft Direct3D 9, while the 11_0 feature level implements the functionality that was implemented in Direct3D 11. Of course, Direct3D 12 also implement the functionality of the earlier versions, adding new ones.
```

```{note}
A software adapter (called WARP adapter) could also be installed on your system by default. To get an interface pointer to a WARP adapter you need to call **EnumWarpAdapter**. However, we won't make use of software adapters in this tutorial series.
```

Before proceeding with the explanation of the **LoadPipeline** function, let's see how we can obtain an interface pointer to a hardware adapter by examining the code of the **DXSample::GetHardwareAdapter** method.

```{code-block} cpp
:caption: D3D12HelloWorld/src/HelloWindow/DXSample.cpp
:name: GetHardwareAdapter-code
// Helper function for acquiring the first available hardware adapter that supports Direct3D 12.
// If no such adapter can be found, *ppAdapter will be set to nullptr.
_Use_decl_annotations_
void DXSample::GetHardwareAdapter(
    IDXGIFactory1* pFactory,
    IDXGIAdapter1** ppAdapter,
    bool requestHighPerformanceAdapter)
{
    *ppAdapter = nullptr;
 
    ComPtr<IDXGIAdapter1> adapter;
 
    ComPtr<IDXGIFactory6> factory6;
    if (SUCCEEDED(pFactory->QueryInterface(IID_PPV_ARGS(&factory6))))
    {
        for (
            UINT adapterIndex = 0;
            SUCCEEDED(factory6->EnumAdapterByGpuPreference(
                adapterIndex,
                requestHighPerformanceAdapter == true ? DXGI_GPU_PREFERENCE_HIGH_PERFORMANCE : DXGI_GPU_PREFERENCE_UNSPECIFIED,
                IID_PPV_ARGS(&adapter)));
            ++adapterIndex)
        {
            DXGI_ADAPTER_DESC1 desc;
            adapter->GetDesc1(&desc);
 
            if (desc.Flags & DXGI_ADAPTER_FLAG_SOFTWARE)
            {
                // Don't select the Basic Render Driver adapter.
                // If you want a software adapter, pass in "/warp" on the command line.
                continue;
            }
 
            // Check to see whether the adapter supports Direct3D 12, but don't create the
            // actual device yet.
            if (SUCCEEDED(D3D12CreateDevice(adapter.Get(), D3D_FEATURE_LEVEL_11_0, _uuidof(ID3D12Device), nullptr)))
            {
                break;
            }
        }
    }
 
    if(adapter.Get() == nullptr)
    {
        for (UINT adapterIndex = 0; SUCCEEDED(pFactory->EnumAdapters1(adapterIndex, &adapter)); ++adapterIndex)
        {
            DXGI_ADAPTER_DESC1 desc;
            adapter->GetDesc1(&desc);
 
            if (desc.Flags & DXGI_ADAPTER_FLAG_SOFTWARE)
            {
                // Don't select the Basic Render Driver adapter.
                // If you want a software adapter, pass in "/warp" on the command line.
                continue;
            }
 
            // Check to see whether the adapter supports Direct3D 12, but don't create the
            // actual device yet.
            if (SUCCEEDED(D3D12CreateDevice(adapter.Get(), D3D_FEATURE_LEVEL_11_0, _uuidof(ID3D12Device), nullptr)))
            {
                break;
            }
        }
    }
    
    *ppAdapter = adapter.Detach();
}
```
<br>

We use **QueryInterface** to check if we can obtain a pointer to **IDXGIFactory6** from the pointer to the **IDXGIFactory4** interface we passed as an argument. Observe that this will work only if the related COM object implements both interfaces (**IDXGIFactory4** and **IDXGIFactory6**). In that case, we can loop to get a pointer to **IDXGIAdapter1** by calling **EnumAdapterByGpuPreference**. Otherwise, we must use **EnumAdapters1**. However, regardless of how you obtain it, we aim to skip software adapters. Such information (and much more) can be obtained from the **DXGI_ADAPTER_DESC1** structure returned by **IDXGIAdapter1::GetDesc1** as an output parameter.

```{note}
Typically, interface names ending with a number extend an earlier, well known interface by inheriting from it and adding new functionalities. For example, in the code above we passed **IDXGIFactory4** as an argument to a **IDXGIFactory1** parameter since we are confident that both interfaces implement **QueryInterface**, which is the function we call through the corresponding interface pointer.
```

Now, we can get back examining **LoadPipeline**.<br>
We need to create a command queue where to submit command lists which, in turn, will hold the commands we want the GPU execute. Indeed, part of the work of a GPU is to execute commands in command lists consumed from a command queue. In this first sample we have very few commands to send to the GPU because it simply shows a window with a blueish client area. Despite this, we still need a command queue as we need to associate it with the swap chain (behind the scenes the DXGI API records commands to be executed by the GPU; more on this shortly). As shown in the image below, there are multiple types of command queues, each of which can hold command lists of a specific type. Then, CPU threads can create command lists of whatever type and insert them in the related command queue.

<br>

![Image](images/A/rect3158.png)

<br>

Many GPUs have one or more dedicated copy engines, a compute engine, and a 3D engine, each capable of executing specific commands in parallel with the other engines. For this reason, there can be no simple guarantee of the order of execution, hence the need for synchronization mechanisms that allow establishing an execution order, if needed.

For this sample, we will use a graphics command queue (referred to as the 3D queue in the illustration above) as it is capable of holding direct command lists that, in turn, can include all types of commands. Indeed, a graphics queue can drive all GPU engines; the compute queue can drive the compute and copy engines, and the copy queue can only drive the copy engine.

To create a swap chain, we need to specify the number of buffers, their size, format, and usage. We're going to use two buffers of the same size as the window's client area. That way, the buffers will be mapped to the client area without stretching the image. <br>
**DXGI_FORMAT_R8G8B8A8_UNORM** indicates the format of the buffers. You can imagine the buffers in the swap chain as grids of (width * height) elements, whose common type is specified by a DXGI_FORMAT value. In this case, we indicate that each element is a 32-bit value composed of four 8-bit unsigned-normalized-integer channels, each in the range $[0/255, 255/255]=[0, 1]$ (that is, each channel can have 256 different values). The four channels are called R, G, B and A to mimic the RGB color model, where a color is defined by the amount of red, green and blue it includes. The channel A (called alpha) is used to control the transparency or the opacity of the color. <br>
**DXGI_USAGE_RENDER_TARGET_OUTPUT** specifies that the buffers will be used as render targets (the targets of drawing operations executed by the GPU). <br>
**DXGI_SWAP_EFFECT_FLIP_DISCARD** indicates that we want to use the flip presentation model (developed by Microsoft to provide a faster way to present frames on the screen) and that DXGI can discard the contents of a back buffer after it is presented to the user — this can enable some optimizations. <br>
**SampleDesc.Count** specifies the number of samples per pixel. It should be always 1 as buffers presented using the flip presentation model don't support multisampling (you need to explicitly create MSAA render targets and resolve yourself the results from multi samples to a single sample as part of the presentation of the frame; we will see how to implement MSAA in a later tutorial). <br>
The call to **CreateSwapChainForHwnd** creates the swap chain. It takes the command queue, the window handle that describes where the buffers will be presented and the description of the buffers in the swap chain. 

```{note}
The comment to the first argument of **CreateSwapChainForHwnd** in {numref}`loadpipeline-code` states that the swap chain needs a queue to flush. That's because, behind the scenes, the DXGI API creates a command list with the commands needed to create the buffers in the swap chain (in GPU memory).
```

Now, we have a pointer to **IDXGISwapChain1**. The corresponding COM object almost certainly also implements **IDXGISwapChain3**, so we get a pointer to this interface by using the member function **ComPtr::As**. **IDXGISwapChain3** allows to invoke **GetCurrentBackBufferIndex** to get the index of the current back buffer in the swap chain (the buffer we are going to draw on; the render target).

The call to **IDXGIFactory::MakeWindowAssociation** prevents switching to full-screen mode using the <kbd>Alt</kbd>+<kbd>Enter</kbd> shortcut. We don't need to provide support for full-screen mode in this sample.

The next step is to create a descriptor heap, a memory space we can consider as an array of descriptors. A descriptor, as its name implies, it's a block of data that describes a resource to the GPU (type, format, address and other hardware-specific information) for binding purposes. That is, whenever we need to bind a resource to the rendering pipeline, we pass a descriptor to the GPU to let it know where to find the resource and how to access it.

<br>

![Image](images/A/rect853.png)

<br>

![Image](images/A/rect853b.png)

<br>

But why do we need to create a descriptor heap? Well, many GPUs require that binding information resides in a small size region of memory, which allows the GPU to use less bits to address them (for example, by using byte offsets from a base address). So, the primary purpose of a descriptor heap is to encompass the bulk of memory allocation required for storing descriptors.

Currently, we have a couple of buffers in the swap chain (allocated in GPU memory) that can be used as render targets. However, whenever we want to create a new frame, we need to bind one of them as render target so that the GPU exactly know where to draw. For this purpose, we need to create a descriptor for each buffer in the swap chain and record a binding command in the command list to specify the current render target to the GPU.

We will create descriptors and store them in a descriptor heap from our C++ applications. This means the descriptor heap needs to be CPU visible. That is, the related space must be allocated in CPU-visible video memory (a small part of dedicated VRAM) or in system memory (RAM). That's exactly what **CreateDescriptorHeap** does. 

```{note}
If you are wondering how GPUs can access descriptors in system memory, the answer is that it is possible through the PCI-e bus. However, as for the descriptor of a render target, the GPU doesn't even need to do that since the driver implicitly copies this descriptor in the command that binds the render target to the pipeline, so that the GPU can directly read from that command (in the correpsponding command list) the details about the buffer to use as render target.
```

```{note}
Differently from render targets, descriptors for texture and other type of resources need to be accessed by the GPU wherever they are in memory. That is, the driver doesn't copy the descriptors in the binding command (more on this in the next tutorial).
```

Then, we set the fields of a **D3D12_DESCRIPTOR_HEAP_DESC** structure to specify we want a descriptor heap that will hold two descriptors of type RTV (render target view; descriptor and view are pretty much the same thing in DirectX). That way, **ID3D12Device::CreateDescriptorHeap** creates a descriptor heap that has enough space to contain two RTVs, and returns an interface pointer to such descriptor heap in the last parameter (or more precisely, an interface pointer to the COM object that implements the interface we will use to reference the descriptor heap; I won't stress on this point anymore). <br>
Observe that a descriptor heap can only hold descriptors of a specific type. We will see other types of descriptors, and the related heaps, starting from the next tutorial.

**ID3D12Device::GetDescriptorHandleIncrementSize** returns the size of a descriptor, based on the type of descriptor heap passed as argument. In this case, we want to know the size of RTVs, so we pass a type of descriptor heap capable of containing them. We store this information for later use.

Once we have the descriptor heap, we need to create the views (RTVs) to the two buffers in the swap chain. **ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart** returns a CPU handle to the first descriptor in the heap, where we are going to store the first RTV.

```{note}
As mentioned earlier, a descriptor heap must be CPU visible, so we need a CPU descriptor handle that points to a descriptor in a descriptor heap in order to store a view. <br>
Handles are opaque pointers, meaning that a handle uniquely identifies a resource, but you shouldn't dereference it as its value only makes sense within a specific context. However, as we will see later in this tutorial, in the context we are working in, CPU descriptor handles returned by **GetCPUDescriptorHandleForHeapStart** are simple CPU virtual addresses, allowing us to use pointer arithmetic to calculate the addresses of other descriptors (see the implementation of **CD3DX12_CPU_DESCRIPTOR_HANDLE::Offset** in *d3dx12.h*). This confirms that descriptor heaps are allocated in CPU-visible memory. <br>
GPU descriptor handles also exist. However, we will only use them to reference descriptors in a special descriptor heap (more on this in the next tutorial). Observe that, unlike CPU handles, usually GPU handles are just byte offsets from the start of a descriptor heap, not virtual addresses.
```

```{note}
**CD3DX12_CPU_DESCRIPTOR_HANDLE** is a wrapper for the Direct3D 12 structure **CPU_DESCRIPTOR_HANDLE**. In general, types with a name that begin with `CD3DX12` are defined in *d3dx12.h* and behave like C++ classes around Direct3D 12 structures to simplify their initialization and to provide useful helper functions.
```

**IDXGISwapChain:: GetBuffer** returns (as an output parameter) an interface pointer to the buffer of the swap chain associated with the index passed in the first parameter. In particular, we get a pointer to a **ID3D12Resource** interface, used to reference a wide variety of resources from our C++ application. That is, **ID3D12Resource** provides useful information about the related resource in GPU memory (type, format, dimension, GPU virtual address, etc.), but we cannot directly use it to access a GPU resource from CPU code.

```{note}
**ID3D12Resource** is only an interface implemented by a COM object in system memory: it doesn't directly reference resources in GPU memory. Even if you have the GPU virtual address of a resource, you can't access it from your C++ application because it only "understands" addresses of its CPU virtual address space. However, in some cases, we can use **ID3D12Resource** to map the GPU memory space of a resource to the virtual address space of our application in order to make it CPU visible\accessible. More on this in the next tutorial.
```

With the pointers to the **ID3D12Resource** interfaces that describe the buffers in the swap chain, we can create the related views by calling **ID3D12Device::CreateRenderTargetView**, indicating the descriptor\position in the descriptor heap where we want to store them. To get the handle of the second descriptor in the heap we use **CD3DX12_CPU_DESCRIPTOR_HANDLE::Offset**, which allows to offset a handle to point a different descriptor from the first one. For this purpose, we need to pass the number of descriptors to offset and their size. That's the reason we stored the size of RTVs.

**ID3D12Device:: CreateCommandAllocator** creates a command allocator. This is essentially a memory manager for a command list, meaning it manages the memory space where a command list stores its commands. You must specify the type of command list the allocator is going to manage. A direct command list can hold all types of commands.

```{note}
We'll be recording commands in command lists from our C++ application, so the memory space managed by the allocators must be CPU visible. And indeed, the allocation occurs in memory accessible to both CPU and GPU. This implies that either of them can access the command list, although one of the two must do it through the PCI-e bus (unless you have an integrated GPU, of course; further details will be covered in the next tutorial).
```

```{attention}
Be careful when reusing a command list because the CPU memory space holding the related commands might still be in use, accessed by the GPU. That is, we don't know when the GPU finishes executing the commands of a command list, at least without a synchronization mechanism between CPU and GPU. We will see how to implement such a mechanism shortly.
```

Now, it's time to look at the code of the **LoadAssets** function.

[WIP]

<br>

````{admonition} Support this project
If you found the content of this tutorial somewhat useful or interesting, please consider supporting this project by clicking on the Sponsor button below. Whether a small tip, a one-time donation, or a recurring payment, all contributions are welcome! Thank you!

```{figure} ../../sponsor.png
:align: center
:target: https://github.com/sponsors/PAMinerva

```
````

<br>

## References
```{bibliography}
:filter: docname in docnames
```