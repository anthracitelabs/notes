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
