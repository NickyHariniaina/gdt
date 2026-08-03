# What is a GDT

A GDT ( Global Descriptor Table ), is a table that we put in memory to tell the CPU: This is how you should orginaze
the memory and who can do what.
Without a GDT the CPU can't do anything.

SO The CPU **needs** the *GDT*

# About GDT itself

A GDT contains some **descriptors**, each descriptors actually describe each part of the memory.
**1 descriptors = 8byte**

## Descriptors

> 1 descriptors actually contains:
        -> the base: where do the part start
        -> the limit: where does the part end
        -> the policy or something like that: who can access to that part

### The NULL descriptor
First descriptors ( index 0 ) - Base = 0, limit = 0, access = 0.

The CPU use this desc to catch error. So if anyone attend to call it. It will produce an error.

### The kernel descriptor - Ring 0
The limit is 0xFFFFF ( use manipulator online to check the decimal value )
This is where MY code is. So it's like: only the kernel can execute it.

### Data Descriptor - Ring 0
This is where the kernel data are stored
Same content as the kernel descriptors

### User scope descriptorS - Ring 3
Maybe the same desc as the above but with lower access to let user actually do something less dangerous

## System privilege
- Ring 0:
    Only used by the kernel itself. Anything specified Ring 0 can do anything.
- Ring 1:
- Ring 2:
- Ring 3:
    Less powerful than Ring 0 ( logic )

**Every descriptors has a ring that tells him what he can do**


## Access bytes table ( !!!! IMPORTANT !!!! )

0x9A -> Ring 0
0x92 -> Ring 0 ( Data )
0xFA -> Ring 3
0xF2 -> Ring 3 ( Data )
0x89 -> TSS ( task management )


# GDT creation with C.

First keep in mind that a descriptor structure looks like this in memory:
Offset  Taille  Description
------  ------  -----------
0-1     2       Limite (bits 0-15)
2-3     2       Base (bits 0-15)
4       1       Base (bits 16-23)
5       1       Access Byte
6       1       Flags (4 bits) + Limite (4 bits)
7       1       Base (bits 24-31)

The code can be found in gdt.c
But here is the explanation:

- First, we have to create the create_gdt_entry function that takes four paremeters.
- target, base, limit, access, flags.
- Those are all unsigned integer.
- first we use set the limit( the 2 first offset )
- using 0xFF as an operand and another number. I don't know if it is magic or not.
- the target is the actual descriptor
- we set the first target[0] by doing an bit AND operation on the limit and the 0xFF
- for the second part, we do a bitshift operation with the limit ( shift to the left ) by 8 and then AND operation with the 0xFF
- for the BASE.
- same as above. bit AND operation but on the 0xFF and the base args we set earlier.
- target[3] will actually do the same as target[1], using base instead of the limit since we are setting the base, and then on the target[4] we do the same things but we use 16 not 8 as an operand for the shift.
- the access byte will actually be placed at the 5 ( like in the descriptors table structure )
- we just set it with `= access`
- To set the 6th offset: just take `flags` and `limits`
- first it is gonna be a bit OR operation.
- the first operand is the flag OR-ed with 0x0F and then left shifted by 4
- the second operand is the limit right shifted by 16  and then AND-ed with 0x0F
- the last part of the descriptors will just be the base shifted to right by 24 and then AND-ed with 0xFF

!!!!!!!!! I should correct the step above for the target[6] because it is not correct... yet... my current implementation is the best way if it is in C
We have some selector we can actually load in assembly.

