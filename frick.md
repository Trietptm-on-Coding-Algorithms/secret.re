<!-- TITLE: frick -->
<!-- SUBTITLE: frick is a kick ass frida cli for reverse engineer inspired by the epic GDB init gef by @hugsy, with commands design similar to uDdbg. -->

# Description
More or less a great attempt to make reverse engineering fun++ && pain_in_the_ass--

### Features for the eyes
* interactive commands with shortcuts 
* nice ui and colors (thanks @hugsy)
* commands history
* save/load previous target offsets and target to attach and work in less then a second

### Good stuffs
* the autodoc script deserved to be in the first place of the awesome stuffs
* custom hexdump highlighting pointers and values
* pointer recursion on registers display
* allow to store vars that can be the result of a command (see examples later)
* commands arguments evaluation (see examples later)
* command ``destruct`` should be really helpful while reversing structs (see screenshot later)
* read with ease any data type, signed/unsigned/le/be
* disasm with capstone engine
* multi hexdump ```hexdump $r0 $r1 $r5 256```
* Android can attach to DT_INIT [read more here on my blog](http://www.giovanni-rocca.com/giving-yourself-a-window-to-debug-a-shared-library-before-dt_init-with-frida-on-android/)
* allow to set callback for targets hit (see notes later)
* pthread creation notification highlighting target routine

##### checkout the [complete commands list](./COMMANDS.md)
##### or how to [improve and create new commands](./EXTENDING.md)
##### or join our [slack](https://join.slack.com/t/resecret/shared_invite/enQtMzc1NTg4MzE3NjA1LTlkNzYxNTIwYTc2ZTYyOWY1MTQ1NzBiN2ZhYjQwYmY0ZmRhODQ0NDE3NmRmZjFiMmE1MDYwNWJlNDVjZDcwNGE)


### TLDR;
It will hook all the given targets offsets, sleep the process and give you an interactive cli
which will allow you to do stuffs - including adding other targets - and of course - move to next.

### Get into the business

```
# run 
git clone https://github.com/iGio90/frick
cd frick
python main.py
```

```
-> frick started - GL HF!
add 0x017BA150 my pointer 1
-> 0x17ba150 added to target offsets
add 0x017BB68C my other target
-> 0x17bb68c added to target offsets
add 0x017BB7A8 one more
-> 0x17bb7a8 added to target offsets
attach com.package libtarget.so
-> frida attached
-> script injected
-> target arch: arm
-> target base at 0xc4af2000
-> attached to 0xc62ac150
-> attached to 0xc62ad7a8
-> attached to 0xc62ad68c
s save
-> session saved

# next time we run frick, we can just do 's load' 
# to load the same target and attach to the same offset
```

An example of command within the context could be:

``memory read 0x1000 128``

this will read 128 bytes at 0x1000.

The same result can be achieved with:

``m r 0x1000 128``

``mem r 0x1000 64+64``

all the arguments are evalueted with my logic - which could be bad. feel free to improve.

Once in a contex - the placeholder **$** can be used to point to a register value - I.E:

``memory read $r0 128``

will read 128 bytes in pointer value held by r0.

in addition to this, it's possible to store variables.

``a = 10 + 10``

``print a + $r0``

will print 20 + the value held by r0.

Variables can be also generated with the result of a command - I.E:

``m r ptr $r0``

will read a pointer in the memory address pointed by the value held in r0. So we can do:

``myptr = memory read pointer $r0``

myptr will now be a value which can be used freely in args:

```
m r myptr 128
print myptr + $r0 + $r1 << 32
m r myptr uint
m read myptr ushort le
m write myptr de ad be ef

temp = m alloc 32
m w temp aa bb cc dd ee ff
```

## Some notes

##### Target callbacks (``once``)

One of the first stuck I met was to attach to a plt (i.e memcpy) so I decided to create the command once which can be used in a lot of way.
Command once act more or less like gdb commands funcionality. We are so allowed to do something like:

```
once init
-> enter one command per line. leave empty to remove an existing callback.
-> type 'end' or 'done' or empty line to finish
memcpy = find export memcpy libc.so
add ptr memcpy
end
-> 2 commands added to init callback

add 0x1000
-> 0x1000 added to target offsets
once 0x1000
-> enter one command per line. leave empty to remove an existing callback.
-> type 'end' or 'done' or empty line to finish
hexdump $r0 $r1 $r2 128
done
-> 1 commands added to 0x1000 callback
```

A callback with N* commands can be added to any target and ``init`` keyword. Init callback is invoked as soon as target base is obtained from the device.

##### destruct

Read arg1 bytes in pointer arg0. Then, recursively read all the pointers in that range for depth in arg2 (default 32 divided by 2 each recursion until < 8).
This should be extreme helpful to highlight structures and arrays of objects.

![Alt text](https://image.ibb.co/iaOgQJ/Schermata_2018_06_17_alle_23_23_06.png "frick")

---
```markdown
MIT License

Copyright (c) 2018 Giovanni - iGio90 - Rocca

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
# Quick links
<a href="/frick/commands">Commands list</a>