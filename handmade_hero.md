Notes on Day 1
===================================

Notes on Day 2
===================================

Notes on Day 3
===================================

Notes on Day 4
===================================

Notes on Day 5
===================================

Notes on Day 6
===================================
Input from gamepad (XInput compatible device) and keyboard. XInput Game Controller API evolved from the Direct Input API. Direct Input API is for legacy controllers so with XInput support we are good to go. Every game using a controller implements XInput API. Other options are Joystickapi and the Raw Input API.

You can use XInput API in 3 steps:
1. Loop over the controllers the system has found
2. Get the state of each controller (pass a structure that is filler out by the function)
3. You can use this state however you want

XInput is a polling based API. Start polling right after the message processing loop. Because keyboard messages are coming through the message loop and we want to capture those.

Here is a code snippet

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
#include <xinput.h>

MSG Message;
while(PeekMessageA(...))
{
    // ...
}
for(DWORD ControllerIndex = 0; ControllerIndex < XUSER_MAX_COUNT; ++ControllerIndex)
{
    XINPUT_STATE ControllerState;
    if (XInputGetState(ControllerIndex, &ControllerState) == ERROR_SUCCESS)
    {
        // NOTE(casey): This controller is plugged in
    }
    else
    {
        // NOTE(casey): This controller is not available.
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now the compilation will give unresolved external symbol error, "unresolved external symbol XInputGetState referenced in function WinMain" to be more specific. The problem looks simple enough, we simply need to link our XInputGetState with a library. 

Usually you would simply look at the bottom of the MSDN page of the function to find the library you need to link against. The problem here is that there're two libraries, and the Platform Requirements line (which up until now was a simple Windows 2000) is a bit sketchy:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Platform Requirements

Windows 8 (XInput 1.4), DirectX SDK (XInput 1.3), Windows Vista (XInput 9.1.0) 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We need one of the following:
* Windows 8 or later to use XInput version 1.4
    * Even in 2020, this is still not the case for everyone. Many people still comfortably use Windows 7.
* DirectX SDK to use XInput version 1.3
    * We don't really know what that means and what version do they mean, or if the user even has it installed.
    * We didn't use DirectX SDK yet, and if we ever are going to use one we're going to use a very early version of it.
* Windows Vista or later to use XInput version 9.1.0
    * Technically, by now Windows XP or earlier shouldn't be of any concern for us... But you never know.

All this said, we cannot guarantee that the user's machine will have the necessary .dlls installed when they will run the game. We cannot simply link with Xinput.lib and hope for the best, because if the program can't find one of these .dlls on the system, our game just won't load.

And that's kind of annoying because you don't need a gamepad to play this game, we're going to allow at least keyboard to play the game.

So what do we do? Let's talk about direct loading of functions.

We're going to copy-paste the function signatures we're interested in to our win32_handmade.cpp file, maybe clean them up first:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
DWORD WINAPI XInputGetState(DWORD dwUserIndex, XINPUT_STATE *pState);
DWORD WINAPI XInputSetState(DWORD dwUserIndex, XINPUT_VIBRATION *pVibration);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Usually, leaving them at that would simply tell the compiler: “there's an external function called this and that that you can bind against during the linker stage”. This doesn't really help us: that's exactly what we had in xinput.h to begin with.

However, if we change these into a typedef, we suddenly say: “there is a function of this type, and I want to start declaring variables that are pointers to it”. Think of our Win32MainWindowCallback: when we registered our WindowClass, we passed to it the pointer to this function as a variable (so that Windows could call us at a later stage).

Because we don't want conflicts with the existing naming, we'll rename this type to something else, let's say x_input_get_state and x_input_set_state, respectively.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
// Define a type of a function
typedef DWORD WINAPI x_input_get_state(DWORD dwUserIndex, XINPUT_STATE *pState);
typedef DWORD WINAPI x_input_set_state(DWORD dwUserIndex, XINPUT_VIBRATION *pVibration);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can declare variables of type x_input_get_state almost in the same manner you can declare an u32. But instead of uint32_t, the type of this thing is a function with that specific signature. OK, maybe in C you can't simply declare a variable x_input_get_state GetState;. That's illegal. But a pointer? No problem. (x_input_get_state *GetState;). And that's totally legal and is exactly what we want.

So we can go ahead and declare a couple of these pointers, one per type:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
// Define a type of a function
typedef DWORD WINAPI x_input_get_state(DWORD dwUserIndex, XINPUT_STATE *pState);
typedef DWORD WINAPI x_input_set_state(DWORD dwUserIndex, XINPUT_VIBRATION *pVibration);
// Create a pointer to function of this type
global_variable x_input_get_state *XInputGetState; 
global_variable x_input_set_state *XInputSetState; 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Unfortunately we run straight into the same problem as when we were typedefing the functions: If we name something with the same name as in xinput.h header, the compiler is going to complain. We change variable names a bit to avoid this compile error.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
// Define a type of a function
typedef DWORD WINAPI x_input_get_state(DWORD dwUserIndex, XINPUT_STATE *pState);
typedef DWORD WINAPI x_input_set_state(DWORD dwUserIndex, XINPUT_VIBRATION *pVibration);
// Create a pointer to function of this type
global_variable x_input_get_state *XInputGetState_; 
global_variable x_input_set_state *XInputSetState_; 
// Create an "alias" to be able to call it with its old name
#define XInputGetState XInputGetState_ 
#define XInputSetState XInputSetState_ 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now you should be able to compile your program. Action we've taken will resolve the linker error, even if the XInputGetState call in WinMain never went anywhere. That function is now actually #defined to be XInputGetState_, and it will call a function pointer.

However, if we try to run, we will get an Access Execution Violation while trying to run XInputGetState(_).
We defined XInputGetState and SetState as global variable function pointers. And being a static global variable, they are cleared to 0, so in fact by calling XInputGetState we try to access and execute a function located at the address 0. And that's very much illegal.

Now let's think for a moment. We started this whole process to make sure that our program doesn't crash if it fails to load XInput .dll. And, as we're right now, we will crash anyway unless we load the correct function. Therefore, we need to define a dummy, a stub function to call if we can't load the correct one. For this to work, our stub has to have the same function signature as the real thing.

We can simply copy and paste the same DWORD WINAPI blablabla but it's ugly and prone to error. Also, if we ever want to update that signature, we need to update it in all locations. Alternatively, we can step up our compiler preprocessor game even further, and #define a function of this exact signature. If you use the form #define NAME(param) you can use one or more params you #define inside your macro, it's almost like a function, but not.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
// Define a function macro
#define X_INPUT_GET_STATE(name) DWORD WINAPI name(DWORD dwUserIndex, XINPUT_STATE *pState)
#define X_INPUT_SET_STATE(name) DWORD WINAPI name(DWORD dwUserIndex, XINPUT_VIBRATION *pVibration)
// Define a type of a function
typedef X_INPUT_GET_STATE(x_input_get_state);
typedef X_INPUT_SET_STATE(x_input_set_state);
global_variable ...
#define ...
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now we can also create our stub function! It does nothing and simply returns ERROR_DEVICE_NOT_CONNECTED. We will name our functions XInputGetStateStub and XInputSetStateStub, respectively. We will also assign these stubs as the default value of the global variables.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
#define X_INPUT_SET_STATE(name) ...
typedef X_INPUT_SET_STATE... 
X_INPUT_GET_STATE(XInputGetStateStub) 
{ 
    return (ERROR_DEVICE_NOT_CONNECTED); 
}
X_INPUT_SET_STATE(XInputSetStateStub) 
{ 
    return (ERROR_DEVICE_NOT_CONNECTED); 
}
global_variable x_input_get_state *XInputGetState_ = XInputGetStateStub; 
global_variable x_input_set_state *XInputSetState_ = XInputSetStateStub; 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To make things a bit clearer, let's group the two calls together to map exactly to these steps:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
// NOTE(casey): XInputGetState
#define X_INPUT_GET_STATE(name) DWORD WINAPI name(DWORD dwUserIndex, XINPUT_STATE *pState)
typedef X_INPUT_GET_STATE(x_input_get_state);
X_INPUT_GET_STATE(XInputGetStateStub) 
{ 
    return (ERROR_DEVICE_NOT_CONNECTED); 
}
global_variable x_input_get_state *XInputGetState_ = XInputGetStateStub;
#define XInputGetState XInputGetState_

// NOTE(casey): XInputSetState
#define X_INPUT_SET_STATE(name) DWORD WINAPI name(DWORD dwUserIndex, XINPUT_VIBRATION *pVibration)
typedef X_INPUT_SET_STATE(x_input_set_state);
X_INPUT_SET_STATE(XInputSetStateStub) 
{  
    return (ERROR_DEVICE_NOT_CONNECTED); 
}
global_variable x_input_set_state *XInputSetState_ = XInputSetStateStub;
#define XInputSetState XInputSetState_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Now, to filling out the details. As the documentation for XInputGetState says, we need one of the following libraries: “Xinput1_3.dll”, “Xinput1_4.dll” or “Xinput9_1_0.dll”. Since “Xinput1_3.dll” should be available on most machines, let's start with it. 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
internal void
Win32LoadXInput()
{
    HMODULE XInputLibrary = LoadLibraryA("Xinput1_4.dll"); 
    if (!XInputLibrary)
    {
        XInputLibrary = LoadLibraryA("Xinput1_3.dll"); 
    }
    if (!XInputLibrary)
    {
        XInputLibrary = LoadLibraryA("Xinput9_1_0.dll"); 
    }
    if(XInputLibrary)
    {
        XInputGetState = (x_input_get_state *)GetProcAddress(XInputLibrary, "XInputGetState");
        if(!XInputGetState) { XInputGetState = XInputGetStateStub; }

        XInputSetState = (x_input_set_state *)GetProcAddress(XInputLibrary, "XInputSetState");
        if(!XInputSetState) { XInputSetState = XInputSetStateStub; }
    }
    else
    {
        // We still don't have any XInputLibrary
        XInputGetState = XInputGetStateStub;
        XInputSetState = XInputSetStateStub;
        // TODO(casey): Diagnostic
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Finally, we need secured the situations where we don't find XInput libraries or where something goes wrong. This is where the stub functions come into play: our game will not simply crash, it will gracefully do nothing.


Keyboard input handling is done directly inside Win32MainWindowCallback alongside other messages.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
case WM_SYSKEYDOWN:
case WM_SYSKEYUP:
case WM_KEYDOWN:
case WM_KEYUP:
{
    // Handle keyboard messages here
    bool IsDown = ((LParam & (1 << 31)) == 0);
    bool WasDown = ((LParam & (1 << 30)) != 0);
    u32 VKCode = WParam;

    if (IsDown != WasDown)
    {
        // Deal with the key codes
        if (VKCode == 'W')
        {
        } 
        else if (VKCode == VK_ESCAPE)
        {
            GlobalRunning = false;
        } 
    }
}
case WM_PAINT: 
{
    //... 
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Notes on Day 7
===================================
We use Direct Sound to be as much compatible as possible with the older systems. Audio is complicated because it can never stop and should always be as close to perfect as possible.

First allocate a circular buffer. Each time sound reproduction arrives to the end it loops around to the beginning, so that it goes around that buffer in circles.

It is pretty common to have 48 kHz which means there are 48000 samples per second so a 2 seconds buffer would contain 96000 of the sound samples. If our game is running at 60 frames per second, we should be producing 800 samples per frame in order to line up our audio to the video. And we can't really write straight under the cursor as we risk to produce bad data as a result. This means that we need to write a bit ahead of the cursor. Let's say it's our game cursor. We will be writing there our sound frames, and when hardware cursor will catch up, new sounds will be played. In conclusion, our objective is to always be a little bit ahead of the hardware cursor.

Why two seconds buffer? If we're playing the game at 60 frames per second, a 16 ms buffer should be enough! We're doing it to prevent possible “frame drops”. There are two possible approaches here:

We'll be writing waaay ahead from where we are right now. This however implies a lot of potential data overwriting.
Writing only one frame ahead of us. This would imply that we stick to 16 frames per second, and if we take too long to draw the next frame, we skip. This is the solution we're going for.

1. Open a handle to DirectSound.
2. Get this buffer up and running, where DirectSound would have its infinite loop on, and place the same sound over and over and over.

At the end, this is just some memory that we're writing to. As we did with the bitmap, we write sound bits to a buffer, and DirectSound fetches those bits to reproduce the sound to the speakers.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
#include <dsound.h>

// NOTE(yakvi): DirectSoundCreate
#define DIRECT_SOUND_CREATE(name) HRESULT WINAPI name(LPCGUID pcGuidDevice, LPDIRECTSOUND *ppDS, LPUNKNOWN pUnkOuter)
typedef DIRECT_SOUND_CREATE(direct_sound_create);


internal void Win32LoadXInput()
{
    // ...
}

internal void Win32InitDSound()
{
    // NOTE(casey): Load the library 
    // NOTE(casey): Create a DirectSound object 
    // NOTE(casey): Set cooperative level
    // NOTE(casey): "Create" a primary buffer
    // NOTE(casey): "Create" a secondary buffer     
}


...
// NOTE(casey): Load the library 
HMODULE DSoundLibrary = LoadLibraryA("dsound.dll");

if(DSoundLibrary)
{
    // NOTE(casey): Create a DirectSound object 
    // NOTE(casey): Set cooperative level
    // NOTE(casey): "Create" a primary buffer
    // NOTE(casey): "Create" a secondary buffer     
}
...
if (Window)
{
    HDC DeviceContext = GetDC(Window);
    Win32ResizeDIBSection(&GlobalBackbuffer, 1280, 720);
    Win32InitDSound();

    // ...
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Notes on Day 8
===================================

Notes on Day 9
===================================

Notes on Day 10
===================================

Notes on Day 11
===================================

Notes on Day 12
===================================

Notes on Day 13
===================================

Notes on Day 14
===================================

Notes on Day 15
===================================

Notes on Day 16
===================================

Notes on Day 17
===================================

Notes on Day 18
===================================

Notes on Day 19
===================================

Notes on Day 20
===================================
