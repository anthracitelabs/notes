~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
void foo() {
    int selection = 1;
    switch(selection)
    {
        case 1:
        {
            int a = 0;
            // do something with a
        }
        break;
        case 2:
        {
            int b = 0;
            // do something with b
        }
        break;
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ASM
"foo()":
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 1
        cmp     DWORD PTR [rbp-4], 1
        je      .L2
        cmp     DWORD PTR [rbp-4], 2
        je      .L3
        jmp     .L5
.L2:
        mov     DWORD PTR [rbp-12], 0
        jmp     .L4
.L3:
        mov     DWORD PTR [rbp-8], 0
        nop
.L4:
.L5:
        nop
        pop     rbp
        ret
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
void foo() {
    int selection = 1;

    if(selection == 1)
    {
        int a = 0;
        // do something with a
    }
    else if(selection == 2)
    {
        int b = 0;
        // do something with b
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ASM
"foo()":
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 1
        cmp     DWORD PTR [rbp-4], 1
        jne     .L2
        mov     DWORD PTR [rbp-12], 0
        jmp     .L4
.L2:
        cmp     DWORD PTR [rbp-4], 2
        jne     .L4
        mov     DWORD PTR [rbp-8], 0
.L4:
        nop
        pop     rbp
        ret
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
