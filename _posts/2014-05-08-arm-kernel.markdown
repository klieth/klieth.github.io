---
layout: post
title: "ARM Kernel"
date: 2014-05-08 04:30:00
categories: arm kernel
---

## The Original Idea
I've always been curious about how a kernel does what it does. In my mind, the kernel is some sort of magical piece of software that manages the connections between the programs I write and everything else that happens in the computer. It displays graphics for me, helps me get input from the keyboard simply, and even defines these mystical streams called `stdout` and `stdin` that manage input and output for me. My normal modus operandi is to sit down and implement whatever I don't understand. And that's the idea here; I want to have a working kernel.

I've tried a couple times to stumble through tutorials I've found online about putting together a simple kernel, but they haven't made any sense to me in the past. That, combined with a less than wonderful Operating Systems course in my Junior year, has made it difficult for me to approach this project seriously.

I've been brainstorming about this project for a while now--maybe the past year or so. There are two reasons I'm not going to bother including a list of ideas I've been brainstorming about: first, they have a larger scope than this project will most likely ever achieve and second, I'm guaranteed to deviate from them, even if succeed in creating a toy kernel.

Using ARM as a target for the kernel was somewhat of an arbitrary decision. I wanted to avoid Intel-based processors because of my confusing experiences with x86 assembly in the past--experiences I didn't wish to repeat.

## Platform Details
First, I decided to use ARMv6 assembly, targeting the ARM Versatile/PB board with an arm1176 CPU.

Since I don't have a physical board, I'm using [QEMU][2] to emulate the hardware. My kernel is compiled using the `arm-none-eabi-binutils` and `arm-none-eabi-gcc` packages. The specific commands I use to compile to the kernel are as follows:

    > arm-none-eabi-gcc -march=armv6 -o kernel.o -c kernel.s
	> arm-none-eabi-ld -N -Ttext=0x10000 -o kernel.elf kernel.o

The first command compiles the assembly in `kernel.s` for the ARMv6 architecture, outputting the file into `kernel.o`. The second command then calls the linker with `kernel.o` as input, which places the `text` section at `0x10000`, the starting location of the program counter. The resulting file is `kernel.elf`, the complete kernel image, which can be run in QEMU.

The command to run the examples is:

    > qemu-system-arm -M versatilepb -cpu arm1176 -kernel kernel.elf

For the examples that do not use a display and write only to a serial port, it is a good idea to also include the `-nographic` option. In that case, you can quit QEMU by using `<C-a> x`.

## Required Background Knowledge
This post assumes knowledge of how a lot of different parts of the computer work at a much lower level than you would normally need to know. If you don't know how a CPU works at a circuit level or don't understand Memory Mapped I/O or at any point are confused about any of the concepts, I'll try to give a brief explanation here. Otherwise, you can skip down to [The Starting Point](#start)

I don't really have room to cover all the ARM assembly that I used in this post, so I'm going to explain mostly high level concepts.

- *Subroutine*: I use this word mostly interchangably with *function*. In fact, I define each subroutine as a `%function` with the `.type` directive. However, *function* implies to me a complete entity written in a higher level language, which none of my chunks of code are. Therefore, I wanted another word for it.
- *Memory Mapped I/O*: I/O happens when the processor needs to communicate with any piece of hardware outside itself, which happens fairly regularly. Maybe it needs to read from the hard disk or send some information to the display. In this post, I will often need to write data to a serial port. Instead of managing the I/O devices on its own, the processor has specified certain memory locations that act as an intermediate step to communicating with the hardware. In the case of the serial port, I can write to the memory location of the port and then the serial port hardware, on its own, will read from that memory location and handle sending the data on its own.  
It would make no sense for the CPU to directly control the external hardware, because this would take large amounts of time away from executing the program. The hardware is build to know how to perform a task very well, so it is possible to completely separate it from the CPU. Mapping it to memory is the simplest of multiple ways to communicate with hardware devices, which is perfect for this project.
- *The Stack*: This is a large region of memory that contains data stored sequentially, instead of being individually addressed by a register. Typically, the stack starts at the largest address available and grows *down*. Therefore, when you're allocating new memory to put on the stack, you actually *subtract* from the current location of the stack.
- *Registers*: These are a small number of special memory locations that the CPU can directly perform operations on. There are a number of general purpose registers, which means that they don't have a specific meaning to the CPU and can be used freely for any calculation. There are around 12 general purpose registers, but in this post, I only end up using the first four, denoted `r0`, `r1`, `r2`, and `r3`. There are also a few registers that have a special meaning.
	- *Stack pointer*: This is a special register, denoted by `sp`, that keeps track of the most recently allocated location on the stack. Therefore, it points to the address of the *bottom* of the stack.
	- *Link register*: When jumping to a subroutine or function, this register will hold the address that the program jumped from so that it is easy to return there.
	- *Program counter*: This register holds the memory location of the currently executing command. Changing this register will jump to a different part of the program. Typically, however, it's safer to use a branch instruction such as `b` or `bl` to safely control the change.
- *Text section*: The part of the assembly code that contains executable instructions. It's denoted by the `.text` directive in my code.

## The Starting Point## {#start}
I had no clue where to start, so I poked around on Google for a while. I got an idea of where to start from a blog called Syngpolyma I found documenting a similar process to what I hope to acheive, but using almost entirely compiled C code. This wasn't good enough for me as I really wanted to understand what's going on at a deeper level, so I'm starting off today with the same process I found in the blog, but with everything written directly in ARM assembly.

[Here is a link][1] to the Singpolyma blog.

## The Simplest Kernel
It's surprisingly difficult to get anything to happen when you're starting with bare metal. Without standard services provided by the kernel or at the very least a display driver, there isn't any good way to display anything on the screen. We'll start off with a kernel that loads and then hangs, doing absolutely nothing. Here's the code:

{% highlight nasm %}
.text
.global _start
_start:
	ldr sp, =0x07FFFFFF
	bl main

.type main,%function
main:
	b main
{% endhighlight %}

When you run this code, QEMU will start up and do...absolutely nothing. This is approximately equivalent to the following C code:

{% highlight c %}
int main(void) {
	while(1);
	return 0;
}
{% endhighlight %}

However, there are many details normally abstracted by C that we have to manually take care of in the assembly. I'll take it apart in smaller chunks.

{% highlight nasm %}
.text
.global _start
{% endhighlight %}

These are a few directives to the assembler that make it easier to keep everything straight. Of particular importance is the `.text` directive which states where the executable code starts.

{% highlight nasm %}
_start:
	ldr sp, =0x07FFFFFF
	bl main
{% endhighlight %}

Here, we are performing two important commands. First, we have to initialize the stack pointer. The address 0x07FFFFFF is at the top of the memory area, which, for a stack that grows down, is exactly where we want to start. Then, we use the branch-and-link operation to call the `main` function. It's a little pointless to perform the link part of the command for the time being because we aren't ever going to jump back here, but it's a good convention to follow.

{% highlight nasm %}
.type main,%function
{% endhighlight %}

Another assembler directive. In our program, `main` is the function that's going to kick off all the work.

{% highlight nasm %}
main:
	b main
{% endhighlight %}

And here's our infinite loop. There's no reason to have a return command, as it would be unreachable code.


##Hello, World!##
Now it's time to display some output. Because I was following the Singpolyma blog, instead of crossing my fingers and messing around with graphics, I wrote output to a serial port. QEMU by default attaches the serial port to stdout of the terminal. While I don't necessarily agree with this approach of skipping writing to the screen, it is a valid way to debug and is by far the easiest to get something displaying quickly.

{% highlight nasm %}
.hello:
	.ascii "Hello, World!\012\000"

.text
.global _start
_start:
	ldr sp, =0x07FFFFFF
	bl main

.type main,%function
main:
	ldr r0, .helloptr
	ldr r2, .serialport
	b .entry
.loop:
	strb r1, [r2]
	add r0,r0,#1
.entry:
	ldrb r1, [r0]
	cmp r1,#0
	bne .loop
.end:
	b .end

.helloptr:
	.word .hello
.serialport:
	.word 0x101F1000
{% endhighlight %}

This roughly corresponds to the following C code:
{% highlight c %}
int main(void) {
	char *hello = "Hello, World!\n";
	while (*hello) {
		*(volatile char *)0x101F1000 = *hello;
		hello++;
	}
	while (1);
	return 0;
}
{% endhighlight %}

I'll just touch briefly on the most important parts of the assembly. First, the pointers. This is a little complex with multiple levels of depth so I won't dwell on it, but think carefully about what's happening when a pointer to a pointer is being loaded.

{% highlight nasm %}
.hello:
	.ascii "Hello, World!\012\000"

@@    ....

.helloptr:
	.word .hello
.serialport:
	.word 0x101F1000
{% endhighlight %}

Here, `.hello` is a pointer to the first character of the `"Hello, world\n"` string. However, in order to be able to load that value of the pointer and perform math on it, we need another pointer to refer to that. Hence the `.helloptr` declaration at the bottom. The `.serialport` is a little simpler; it defines the constant memory location where the serial port can be accessed.

Inside the string itself, those funky codes at the end are what's generated by a C compiler when you have a string constant. `\012` corresponds to the `newline (\n)` character. `\000` is the null terminator of the string.

{% highlight nasm %}
main:
	ldr r0, .helloptr
	ldr r2, .serialport
	b .entry
{% endhighlight %}

This is the beginning of the main function. It prepares the two registers we'll use. The first tracks the location in the string and the second contains the memory address of the serial port. Then, once those are loaded, we jump into what seems to be the middle of the loop. Adding an extra label to jump to in the middle of the loop gives us a better place to start the loop. This actually is something I picked up from looking at the assembly gcc generates. I believe it is little shorter than following the order of execution that seems to happen in the C code.

{% highlight nasm %}
.loop:
	strb r1, [r2]
	add r0,r0,#1
.entry:
	ldrb r1, [r0]
	cmp r1,#0
	bne .loop
{% endhighlight %}

Here's the actual loop. The program jumps to `.entry`, grabbing a character from the string and testing if it's the NULL character which identifies the end of the string. If not, jumps back to `.loop`. There, it pushes the character to the serial port and the current character in the string is incremented and the process starts over. After the loop is done, we can then proceed into the infinite waiting loop.

Running this with QEMU produces a line on stdout saying `Hello, World!`. Perfect!

##Subroutines##
Printing a string is a fairly common process, so we want to be able to have a generic subroutine that will send the string to the serial port for us. Let's extract the relevant code out of the main method.

A quick note on parameters: the ARM standard says that the first four parameters are passed to a subroutine using the first few registers (`r0-r3`). We'll need a parameter to give the pointer to the string we're displaying.

Here's the subroutine:
{% highlight nasm %}
.type puts, %function
puts:
	ldr r2, .serialport
	b .entry
.loop:
	strb r1, [r2]
	add r0,r0,#1
.entry:
	ldrb r1, [r0]
	cmp r1,#0
	bx lr
{% endhighlight %}

This looks almost the same as what we had in the `main` function, with two major differences. First, we can now end a chunk of code with something slightly more sane than an infinite loop. In most cases, we'll want to jump back in the code to the place we called the subroutine from. That's exactly what the last `bx lr` line does. The command `bx` allows us to jump to the address stored in a register, in this case `lr` or the `link register` which is autmatically set by the `bl` command in the main method to point to the next instruction after the jump.

Second, we've skipped the `ldr r0, .stringpointer` command that was in the original version. That will be set as a parameter before the subroutine is called. Here is an example:

{% highlight nasm %}
main:
	ldr r0, .helloptr
	bl puts
{% endhighlight %}

##User Mode##
So far, everything that's happened has been in Supervisor mode. This mode grants access to everything in the computer, plenty of which should not be accessible to the user. For instance, User mode processes should only be able to access the memory that they have directly allocated. If they could access all of memory, they could alter data in other programs and cause security vulnerabilities. Therefore, User mode has to have a stack defined for it. In order to help protect the computer, we can switch it into User mode. At the most basic, we switch over and call a function in user mode like this:

{% highlight nasm %}
	mov r0, 0x10
	msr SPSR, r0
	ldr lr, =first
	movs pc, lr
{% endhighlight %}

In this chunk, the first two lines prepare the magic value `0x10` which tells the processor to switch to User mode when saved to the `SPSR` or `Saved Program Status Register`. Then, the next two lines load the address of a function, in this case a function named `first`, and move that address into `pc`, the program counter.

However, this won't work just yet--we still need to set up the stack for use in User mode. In order to do that, we'll use a third mode called System mode which is a combination of Supervisor and User modes. System mode uses the stack pointer and link register of User mode, but still has access to the entire computer in the same way Supervisor mode does. That makes it handy to initialize the stack pointer.

{% highlight nasm %}
	mov ip, sp
	msr CPSR_c, #0xDF
	mov sp, ip
	msr CPSR_c, #0xD3
	@@ ... continue to context switch
{% endhighlight %}

This chunk of code maps the User mode stack to the same location as the kernel mode stack by storing the stack pointer into the `ip` register, which survives the switch into System mode. It then sets the stack pointer inside System mode to the stored value, so the User mode stack is actually sharing part of the kernel mode stack. This isn't great if we ever want to have multiple child processes, but it'll do for now.

Combining this into its own subroutine to perform a context switch would probably be useful. Because of the caveats listed in the previous paragraph, it doesn't take any parameters. That will be changed in the future. Here's all the code for now:

{% highlight nasm %}
.type switch, %function
switch:
	mov ip, sp
	msr CPSR_c, #0xDF
	mov sp, ip
	msr CPSR_c, #0xD3
	mov r0, 0x10
	msr SPSR, r0
	ldr lr, =first
	movs pc, lr
{% endhighlight %}

The last thing we need is a function to run in user mode. Right now our code jumps to a function called `first`, so let's write that function.

{% highlight nasm %}
.userstr: .ascii "In user mode\012\000"

@@  ...........

.type first, %function
first:
	ldr r0, .userstrptr
	bl puts
.end:
	b .end

@@  ...........

.userstrptr:
	.word .userstr
{% endhighlight %}

This function repeats what happens in `main`, sending a string to the serial port, but this time from user mode. The string is set up the same way the original string was.

All the code at this point is available on [GitHub][gh].



[1]: https://singpolyma.net/2012/01/writing-a-simple-os-kernel-part-1/
[2]: http://wiki.qemu.org/Main_Page
[gh]: https://github.com/klieth/arm-kernel/tree/da66783f2672a7dab6cc1a928c16f6ed614ff59c
