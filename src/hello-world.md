# Hello, world!

Now that we’ve got the headers out of the way, let’s do the traditional first
program: Hello, world!

## The smallest kernel

Our hello world will be just _20_ lines of assembly code. Let’s begin.
Open a file called `boot.asm` and put this in it:

```x86asm
start:
    hlt
```

You’ve seen the `name:` form before: it’s a _label_. This lets us name a line
of code. We’ll call this label `start`, which is the traditional name.
GRUB will use this convention to know where to begin.

The `hlt` statement is our first bit of ‘real’ assembly. So far, we had just
been declaring data. This is actual, executable code. It’s short for ‘halt’.
In other words, it ends the program.

By giving this line a label, we can call it, like a function. That’s what
GRUB does: “Call the function named `start`.” This function has just one
line: stop.

Unlike many other languages, you’ll notice that there’s no way to say if
this ‘function’ takes any arguments or not. We’ll talk more about that later.

This code won’t quite work on its own though. We need to do a little bit more
bookkeeping first. Here’s the next few lines:

```x86asm
global start

section .text
bits 32
start:
    hlt
```

Three new bits of information. The first:

```x86asm
global start
```

This says “I’m going to define a label `start`, and I want it to be available
outside of this file.” If we don’t say this, GRUB won’t know where to find its
definition. You can kind of think of it like a ‘public’ annotation in other
languages.

```x86asm
section .text
```

We saw `section` breifly, but I told you we’d get to it later. The place where
we get to it is at the end of this chapter. For the moment, all you need to
know is this: code goes into a section named `.text`. Everything that comes
after the `section` line is in that section, until another `section` line.

```x86asm
bits 32
```

GRUB will boot us into protected mode, aka 32-bit mode. So we have to specify
that directly. Our Hello World will only be in 32 bits. We’ll transition from
32-bit mode to 64-bit mode in the next chapter, but it’s a bit involved.
So let’s just stay in protected mode for now.

That’s it! We could theoretically stop here, but instead, let’s actually print
the “Hello world” text to the screen. We’ll start off with an ‘H’:

```x86asm
global start

section .text
bits 32
start:
    mov word [0xb8000], 0x0248 ; H
    hlt
```

This new line is the most complicated bit of assembly we’ve seen yet. There’s a
lot packed into this little line.

The first important bit is `mov`. This is short for `move`, and it sorta looks
like this:

```text
mov size place, thing
```

Oh, `;` starts a comment, remember? So the `; H` is just for us. I put this
comment here because this line prints an `H` to the screen!

Yup, it does. Okay, so here’s why: `mov` copies `thing` into `place`. The amount
of stuff it copies is determined by `size`.

```x86asm
;   size place      thing
;   |    |          |
;   V    V          V
mov word [0xb8000], 0x0248 ; H
```

“Copy one word: the numer `0x248` to ... some place.

The place is a hexidecimal number, but has square brackets `[]` around it.
Those brackets are special. They mean “the address in memory located by this
number.” In other words, we’re copying the number `0x0248` into the specific
memory location `0xb8000`. That’s what this line does.

Why? Well, we’re using the screen as a “memory mapped” device. Specific
positions in memory correspond to certian positions on the screen. And
the position `0xb8000` is one of those positions: the upper-left corner of the
screen.

Now, we are copying `0x0248`. Why this number? Well, it’s in three parts:

```text
 __ background color
/  __foreground color
| /
V V  
0 2 48 <- letter, in ASCII
```

We’ll start at the right. First, two numbers are the letter, in ASCII. `H` is
72 in ASCII, and 48 is 72 in hexidecimal: `(4 * 16) + 8 = 72`. So this will
write `H`.

The other two numbers are colors. There are 16 colors available, each with a
numer. Here’s the table:

```
| Value | Color          |
|-------|----------------|
| 0x0   | black          |
| 0x1   | blue           |
| 0x2   | green          |
| 0x3   | cyan           |
| 0x4   | red            |
| 0x5   | magenta        |
| 0x6   | brown          |
| 0x7   | gray           |
| 0x8   | dark gray      |
| 0x9   | bright blue    |
| 0xA   | bright green   |
| 0xB   | bright cyan    |
| 0xC   | bright red     |
| 0xD   | bright magenta |
| 0xE   | yellow         |
| 0xF   | white          |
```

So, `02` is a black background with a green foreground. Classic. Feel free to
change this up, use whatever combination of colors you want!

So this gives us a `H` in green, over black. Next letter: `e`.

```x86asm
global start

section .text
bits 32
start:
    mov word [0xb8000], 0x0248 ; H
    mov word [0xb8002], 0x0265 ; e
    hlt
```

Lower case `e` is `65` in ASCII, at least, in hexidecimal. And `02` is our same
color code. But you’ll notice that the memory location is different.

Okay, so we copied four hexidecimal digits into memory, right? For our `H`.
`0248`. A hexidecimal digit has sixteen values, so two of them are 32. We are
in 32-bit mode, so two digits is one byte in memory. Since we need one byte for
the colors, and one byte for the `H`, that’s two bytes. Hence, if our first memory
position is at `0`, the second letter will start at `2`.

This math gets easier the more often you do it. And we won’t be doing _that_ much
more of it. There is a lot of working with hex numbers in operating systems work,
so you’ll get better as we practice.

With this, you should be able to get the rest of Hello, World. Go ahead and try
if you want: each letter needs to bump the location twice, and you need to look
up the letter’s number in hex.

If you don’t want to bother with all that, here’s the final code:

```x86asm
global start

section .text
bits 32
start:
    mov word [0xb8000], 0x0248 ; H
    mov word [0xb8002], 0x0265 ; e
    mov word [0xb8004], 0x026c ; l
    mov word [0xb8006], 0x026c ; l
    mov word [0xb8008], 0x026f ; o
    mov word [0xb800a], 0x022c ; ,
    mov word [0xb800c], 0x0220 ; 
    mov word [0xb800e], 0x0277 ; w
    mov word [0xb8010], 0x026f ; o
    mov word [0xb8012], 0x0272 ; r
    mov word [0xb8014], 0x026c ; l
    mov word [0xb8016], 0x0264 ; d
    mov word [0xb8018], 0x0221 ; !
    hlt
```

Finally, now that we’ve got all of the code working, we can assemble our
`boot.asm` file with `nasm`, just like we did with the `multiboot_header.asm`
file:

```bash
$ nasm -f elf64 boot.asm
```

This will produce a `boot.o` file. We’re almost ready to go!

## Linking it together

EXPLAIN ALL THIS

```text
ENTRY(start)

SECTIONS {
    . = 1M;

    .boot :
    {
        /* ensure that the multiboot header is at the beginning */
        *(.multiboot_header)
    }

    .text :
    {
        *(.text)
    }
}
```

```bash
$ ld -n -o kernel.bin -T linker.ld multiboot_header.o boot.o
```