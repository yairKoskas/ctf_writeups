# Babyheap | Author - ElikBelik77
#### Even babies can play with heaps, can you win the baby?
## Overview
 This challenge looks quite simple, we can add a note, delete a note, edit a note, show a note's content or exit
 What's actually happening, is that the notes are allocated on the heap, and thus make room for heap exploitation
We can examine the binary in gdb(or every other debugger\disassembler) and see there's a win function, so the goal is to redirect code execution to the win function. but how can we do it? 
***
**Tcache**
In short, the tcache is a linked-list cache in libc for allocated chunks, and malloc searches the tcache in order to allocate new chunks.
Now let's begin the fun part. 
First, we'll exploit a Double free bug - let's allocate a note at index 0, and a note in index 1 ,and free 0 twice.
We need to allocate the chunk at index 1 in order for the chunks to enter the tcache
We freed the zero chunk twice which means the tcache now looks like this - chunk0 -> chunk0
alloc(0)
alloc(1)
free(0)
free(0)
now, because malloc searches the tcache, we can allocate new chunks whose contents will point on each other
so we can just find a good place to put the pointer to the win function, but where can we find one?
***
**The Global Offset Table**
the Global offset table (GOT) is a symbol lookup table which is responsible (partially) for runtime lookup for dynamically linked functions.
so functions like strcpy, strcmp and other commonly-used libc functions are in there in runtime.
a call to a function goes something like this - 
strcpy -> strcpy@plt -> strcpy@got.plt -> strcpy in libc
(i missed the resolving part, but for the sake of simplicity let's keep it like that)
so we can just override the pointer strcpy@got.plt for example, and then call strcpy and we'll win.
strcpy is just a given example and we don't have access to it in the challenge, so what can we call?
***
we can see in the menu, that 5 is for exit, which means we can call exit at will!
so let's just allocate chunks with exit_got and win, and we can call exit and get the flag!

alloc(2) with content exit_got
alloc(3) # because we have chunk1 in the tcache also
alloc(4) with content win_pointer
and then just exit and let the magic happen
(note, you can also override the puts_got and get funnier output)

![image](/home/yairko/Pictures/babyheap.png)

# PWNed By Koskas