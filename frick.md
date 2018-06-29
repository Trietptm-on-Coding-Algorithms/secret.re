<!-- TITLE: frick -->
<!-- SUBTITLE: frick is a kick ass frida cli for reverse engineer inspired by the epic GDB init gef by @hugsy, with commands design similar to uDdbg. -->

More or less a great attempt to make reverse engineering fun++ && pain_in_the_ass--

# Features for the eyes
* interactive commands with shortcuts 
* nice ui and colors (thanks @hugsy)
* commands history
* save/load previous target offsets and target to attach and work in less then a second

# Good stuffs
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

# Quick links
<a href="/frick/commands">Commands list</a>