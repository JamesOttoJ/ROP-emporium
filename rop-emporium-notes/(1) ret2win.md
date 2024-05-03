Based on the title, I assume I'll just have to get the address of the given function and jump there

### Initial Analysis

Main function prints some data before calling the `pwnme()` function. `pwnme()` allocates 32 bytes to the input storage variable, but it takes 56. This will lead to a buffer overflow that we can exploit.

### Analyzing the Stack

When doing a buffer overflow, it is important to keep in mind the stack layout.
- Allocated variables (variable bytes, but only 32 bytes for the input variable in this case)
- Address of the calling function's stack pointer, AKA stored RSP (8 bytes in x86_64)
- Return address in calling function, AKA stored RIP (8 bytes in x86_64).

Actual stack from when it reads data (pulled from gdb):
```
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdd30 --> 0x0 // input[0] - input[7]
0008| 0x7fffffffdd38 --> 0x0 // input[8] - input[15]
0016| 0x7fffffffdd40 --> 0x0 // input[16] - input[23]
0024| 0x7fffffffdd48 --> 0x0 // input[24] - input[31]
0032| 0x7fffffffdd50 --> 0x7fffffffdd60 --> 0x1 // Stored RSP
0040| 0x7fffffffdd58 --> 0x4006d7 (<main+64>:	mov    edi,0x400828) //Stored RIP
```

This means that our input will start by filling the variable space. Then, it will overwrite the stored stack pointer. Finally, it will overwrite the stored return address (this is what we want)

To find the address, I look for strings in the binary. It came with flag.txt, so it probably has to open it by referencing the name. We can see the string, "/bin/cat flag.txt". This is then referenced in the `ret2win()` function. All we need to do now is grab the address of our payload and put it where the saved RIP is.

### Building the Payload

Now that we know the size of everything and the address we want, we can build part of our payload (remember, the bytes of the address are reversed because of little endian):
`python -c 'print("A" * 32 + "B" * 8 + "\x56\x07\x40", end="")' | ./ret2win`

As will fill the variable, Bs will fill the stored RSP, and the hex is the location of the function.

If the payload doesn't work, pipe the python result into `od -t x` instead of `./ret2win`. Make sure that it doesn't end with 0x0a (newline).

