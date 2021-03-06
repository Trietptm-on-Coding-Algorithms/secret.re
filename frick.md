<!-- TITLE: frick -->
<!-- SUBTITLE: frick - aka the first debugger built on top of frida - is a kick ass frida cli for reverse engineer inspired by the epic GDB init gef by @hugsy. -->

More or less a great attempt to make reverse engineering fun++ && pain_in_the_ass--

# Description
Frida makes reverse engineering better. By allowing arbitrary code injection at any time and the abilities to play with pointer and use native functions through js api, frida turns debugging into the next level. Scripting exploit and stuffs to reach challenges will fit any case, however, I decided to code a cli for the following reasons:

* any step forward (aka any new line of code) require package restart (or a static code to reload the script)
* hooking and tracing a routin which is invoked hundred times is a pain.
* having colors highlighting pointers/values and structs would turn me to code some static "framework" in any case
* Whatever is the goal, using the fastest way is always a must. I coded inside everything to go faster (that fit my approach)

# Why it pwn asses
* Quick shortcuts for any command
* Each command result can be stored into a variable, which can be used as args to other commands
* Declaring and use native functions at runtime
* Multihexdump, highlighting valid pointers and values with different colors
* Deep pointer recursion in registers display
* Destruct command for highlight arrays and unknown structures
* Can be automated running commands inside callbacks
* Easily write any data type
* PThreads creation notication
* Shortcuts to access and store values into registers 
* Can load a saved or manually modified session to be ready asap
* Capstone engine integrated
* Android can attach to dt_init, leaking module base from linker before initializations
* It's a debugger! All threads and pthreads will sleep until next
* You can dynamically load blobs into target process using command inject (syscall 385)
* Unicorn emulation with automatic (dynamic) memory mappings from target device
# Install and Run
```
git clone https://github.com/iGio90/frick
cd frick
python main.py
```

In the moment im writing this, there is not yet an update system. Use git to keep the project synced with master.

# Step by Step
## Get familiar

The very first important thing to begin with, is understanding the session file. This file will be created in the frick root and it's basically a list of commands that can be loaded to quickly begin the session.

Assuming we are targetting package **com.package** and the function at offset **0x1000** of the shared library **libtest**

```python
python main.py
-> frick started - GL HF!
add 0x1000
-> 0x1000 added to target offsets
attach com.package libtest.so
-> frida attached
-> script injected
-> target arch: arm
-> pointer size: 4
-> leaked target base at 0xcb4f1000
-> attached to 0xcb4f2000
-> 0xcb4f2000 added to target offsets
```

later we can run **session save** command to store our commands to restart the session
```python
session save
-> session saved

session load
# will add 0x1000 and attach to the same package/module
```

We are now attached to the function (or arbitrary address) and once the program will hit the hook, we will have a context to play with.

## Context commands
Once we hit the hook, we can unleash the power of frida and frick to do a lot of different stuffs. Here some examples:

### Reading registers
```python
registers
-------------------------------------------------------------------------------[ 0xf1e2e695 ]----
R0  : 0x5a
R1  : 0xd0db74e8 -> 0x642f0001
R2  : 0x6e
R3  : 0xf1e3d16a -> 0x2b720000 -> 0x0
R4  : 0x5a
R5  : 0xd0db74e8 -> 0x642f0001
R6  : 0xd0db74ea -> 0x7665642f -> 0x0
R7  : 0x1
R8  : 0xd0db7cd0 -> 0x39333339 -> 0x0
R9  : 0x0
R10 : 0xd0db75a0 -> 0x0
R11 : 0xd0db7678 -> 0x0
R12 : 0xf1ea3b2c -> 0xf1e2e695 -> 0x1f000f8
SP  : 0xd0db74e0 -> 0x4
PC  : 0xf1e3d109 -> 0xebd1051c -> 0x0
LR  : 0xf1e3d109 -> 0xebd1051c -> 0x0
```

### Reading memory

```python
memory read 0xf1e3d109 128
-------------------------------------------------------------------------------[ 0xf1e3d109 ]----
F1E3D109: 1C 05 D1 EB F7 88 E9 00  68 04 28 F3 D0 06 E0 28  ........h.(....(
F1E3D119: B9 14 A1 20 46 EC F7 96  E9 05 46 03 E0 20 46 EB  ....F.....F...F.
F1E3D129: F7 9E E9 00 25 10 48 1E  99 78 44 00 68 00 68 40  ....%.H..xD.h.h@
F1E3D139: 1A 02 BF 28 46 1F B0 F0  BD EB F7 FC E8 00 BF 2A  ...(F..........*
F1E3D149: 63 06 00 C8 76 05 00 EB  8B 05 00 2F 64 65 76 2F  c...v....../dev/
F1E3D159: 73 6F 63 6B 65 74 2F 64  6E 73 70 72 6F 78 79 64  socket/dnsproxyd
F1E3D169: 00 00 00 72 2B 00 00 92  62 06 00 B0 B5 84 B0 DF  ...r+...b.......
F1E3D179: F8 40 C0 DD E9 08 4E FC  44 0B 9D CD E9 00 4E CD  .@....N.D.....N.
```

### Module informations

```python
info mod linker
----------------------------------------------------------------------------------------------[ linker ]----
name: linker
base: 0xf41f6000
size: 0x8d000 (577536)
path: /system/bin/linker

info mod libc.so
---------------------------------------------------------------------------------------------[ libc.so ]----
name: libc.so
base: 0xf1e13000
size: 0x93000 (602112)
path: /system/lib/libc.so
```

## Shortcuts and Placeholders

To minimize the effort, all the commands have nested shortcuts. The placeholder **$** can be used to point a register value.


```
memory read $pc 32
-------------------------------------------------------------------------------[ 0xf1e3d109 ]----
F1E3D109: 1C 05 D1 EB F7 88 E9 00  68 04 28 F3 D0 06 E0 28  ........h.(....(
F1E3D119: B9 14 A1 20 46 EC F7 96  E9 05 46 03 E0 20 46 EB  ....F.....F...F.

mem read $pc 32
-------------------------------------------------------------------------------[ 0xf1e3d109 ]----
F1E3D109: 1C 05 D1 EB F7 88 E9 00  68 04 28 F3 D0 06 E0 28  ........h.(....(
F1E3D119: B9 14 A1 20 46 EC F7 96  E9 05 46 03 E0 20 46 EB  ....F.....F...F.

m r $pc 32
-------------------------------------------------------------------------------[ 0xf1e3d109 ]----
F1E3D109: 1C 05 D1 EB F7 88 E9 00  68 04 28 F3 D0 06 E0 28  ........h.(....(
F1E3D119: B9 14 A1 20 46 EC F7 96  E9 05 46 03 E0 20 46 EB  ....F.....F...F.
```

---

```python
what = pack /proc/self/maps
print what
-> 246318648710016337982643999497875571
mem write $r11 what
mem read $r11 32
-------------------------------------------------------------------------------[ 0xd0d97678 ]----
D0D97678: 2F 70 72 6F 63 2F 73 65  6C 66 2F 6D 61 70 73 00  /proc/self/maps.
D0D97688: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................
```

---


```python
myalloc = m alloc 32
print myalloc
-> 0xd2a18d40 (3533802816)
m w myalloc deadbeef
$r11 = myalloc
0xd2a18d40 (3533802816)
regs
-------------------------------------------------------------------------------[ 0xf1e2e695 ]----
R0  : 0x5b
R1  : 0xd0d974e8 -> 0x642f0001
R2  : 0x6e
R3  : 0xf1e3d16a -> 0x2b720000 -> 0x0
R4  : 0x5b
R5  : 0xd0d974e8 -> 0x642f0001
R6  : 0xd0d974ea -> 0x7665642f -> 0x0
R7  : 0x1
R8  : 0xd0d97cd0 -> 0x39333339 -> 0x0
R9  : 0x0
R10 : 0xd0d975a0 -> 0x0
R11 : 0xd2a18d40 -> 0xefbeadde -> 0xb9b8003b
R12 : 0xf1ea3b2c -> 0xf1e2e695 -> 0x1f000f8
SP  : 0xd0d974e0 -> 0x4
PC  : 0xf1e3d109 -> 0xebd1051c -> 0x0
LR  : 0xf1e3d109 -> 0xebd1051c -> 0x0
```

# Examples
## Multi hexdump
```python
hexdump $r1 $r3 128
D0D974E8: 01 00 2F 64 65 76 2F 73  6F 63 6B 65 74 2F 64 6E  ../dev/socket/dn		F1E3D16A: 00 00 72 2B 00 00 92 62  06 00 B0 B5 84 B0 DF F8  ..r+...b........
D0D974F8: 73 70 72 6F 78 79 64 00  00 00 00 00 00 00 00 00  sproxyd.........		F1E3D17A: 40 C0 DD E9 08 4E FC 44  0B 9D CD E9 00 4E CD E9  @....N.D.....N..
D0D97508: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................		F1E3D18A: 02 5C 00 F0 18 F8 04 46  0A 98 00 2C 04 60 08 D1  .\.....F...,.`..
D0D97518: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................		F1E3D19A: EB F7 42 E9 01 46 08 68  1C 28 02 D1 22 20 08 60  ..B..F.h.(.."..`
D0D97528: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................		F1E3D1AA: 04 E0 00 20 00 2C 08 BF  4F F0 FF 30 04 B0 B0 BD  .....,..O..0....
D0D97538: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................		F1E3D1BA: 00 BF E4 CC 05 00 2D E9  F0 4F 93 B0 80 46 70 48  ......-..O...FpH
D0D97548: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 40  ...............@		F1E3D1CA: 8B 46 9A 46 78 44 17 46  00 68 01 68 6E 48 12 91  .F.FxD.F.h.hnH..
D0D97558: 99 4F AC A3 E0 75 D9 D0  74 76 D9 D0 58 5E 0B D0  .O...u..tv..X^..		F1E3D1DA: 78 44 EB F7 C6 E8 38 B1  6C 49 79 44 EB F7 6A E9  xD....8.lIyD..j.
```

## Destruct
```python
destruct $r3 32
0x2b720000
    0x00000000
    0x00000000
    0x00000000
    0x00000000
0x00009262
0x0600b0b5
0x84b0dff8
0xe9ddc040
    0x27006400
    0x65002700
    0x20007900
    0x20130020
        0x00000000
        0x00000000
0x44fc4e08
    0x00000000
    0x00000000
    0x00000000
    0x00000000
0xe9cd9d0b
    0x0ade04f6
    0xf10e650a
        0x16323336
        0x15140601
    0xe2083c00
        0xc18822fb
        0xc19822fb
    0xe60b2d04
        0x35051521
        0xd1011521
0xe9cd4e00
    0x29d643af
        0x00000000
        0x00000000
    0x2c0b2fb2
        0x00000000
        0x00000000
    0x2c272926
        0x00000000
        0x00000000
    0x2a842c20
        0x00000000
        0x00000000
```

## Once callbacks
```python
once init
socket = find export socket libc.so
add ptr socket
end

add 0x1000

once 0x1000
$r2 = 0xdeadbeef
# unpause thread
run
end

attach com.package libtest.so
```

## Native Functions
```python
gettidptr = find exp gettid libc.so
# int -> return type
function add gettidptr int
-> 0xf1e304bd (gettid - libc.so)
function run gettid
-> 0x2d87 (11655)

fopenptr = find exp fopen libc.so
# pointer -> return type
# pointer pointer -> args
function add fopenptr pointer pointer pointer
-> 0xf1e65065 (fopen - libc.so)
```

## Inject a local blob
```python
# assuming libfrick.so is in frick root directory
inject libfrick.so friiiiiick
-> memfd:friiiiiick - mapped at 0xcb4e2000

# we can now use other commands to invoke subs in our binary during runtime.
# i.e find export and function command. 
# note: memfd: will always be before custom name
```

# Using custom scripts
Yeah I know. Sometimes we are too sticked to scripting... we need custom logic that fit the case etc etc so we also have a spot to use custom scripts and use the unique api ``cli`` (if needed), to break the process and have cli context
In the moment i'm writing this ``session save`` doesn't export scripts, so we can just manually put a line in the .session file or load the scripts during runtime. Using session would be like:
```
# scripts load can be safely used before/after command attach
scripts load .scripts/test.js
attach com.package libtest.so
```

content for ``test.js`` could be something like:

```javascript
var target_ptr = Module.findExportByName('libc.so', 'memcpy');
var p = Interceptor.attach(target_ptr, function () {
    p.detach();
		// cli api needs target offset and this.context object
    cli(target_ptr, this.context);
});
```

# Emulating
Once inside a cli context, it's also possible to emulate the cpu through unicorn integration. Simply run command ``emu start exit_point``
```
emu start 0xcf9d20cf
-> starting emulation at 0xcf7e20b9
192 instructions traced, 84 memory access
```

During emulation, an html file is written in ``.emulator`` folder in the root of frick, it will be automatically opened at the end of emulation containing something like this (with colors highlight)
```html
-> mapping 4096 at 0xcf7e2000


0xcf7e20b8:    LDR    r0, [sp, #0x14]
-> reading to an unmapped memory region at 3385320540
-> mapping 1032192 at 0xc9b84000
0xcf7e20b8:    LDR    r0, [sp, #0x14]
-> READ at 0xc9c7e45c, data size = 4, data value = 0xf4c15fe6
0xcf7e20ba:    STR.W    sl, [sp, #0xc]
-> WRITE at 0xc9c7e454, data size = 4, data value = 0x0
0xcf7e20be:    STR.W    r8, [sp, #8]
-> WRITE at 0xc9c7e450, data size = 4, data value = 0x20
0xcf7e20c2:    STR    r0, [sp, #0x30]
-> WRITE at 0xc9c7e478, data size = 4, data value = 0xe65fc1f4
0xcf7e20c4:    MOVS    r0, #0
0xcf7e20c6:    STR.W    sl, [sp, #0x2c]
-> WRITE at 0xc9c7e474, data size = 4, data value = 0x0
0xcf7e20ca:    STR.W    r8, [sp, #0x28]
-> WRITE at 0xc9c7e470, data size = 4, data value = 0x20
0xcf7e20ce:    STR    r0, [sp, #0x24]
-> WRITE at 0xc9c7e46c, data size = 4, data value = 0x0
0xcf7e20d0:    B    #0xcf7e21b8 (0x3bb1b8 - libg.so)


0xcf7e21b8:    LDR    r0, [sp, #0x24]
-> READ at 0xc9c7e46c, data size = 4, data value = 0x0
0xcf7e21ba:    MOVS    r1, #0
0xcf7e21bc:    CMP.W    r0, #0x100
0xcf7e21c0:    MOV.W    r0, #2
0xcf7e21c4:    IT    lt
0xcf7e21c6:    MOVLT    r1, #1
0xcf7e21c8:    STR    r0, [sp, #0x54]
-> WRITE at 0xc9c7e49c, data size = 4, data value = 0x2
0xcf7e21ca:    STRB.W    r1, [sp, #0x5a]
-> WRITE at 0xc9c7e4a2, data size = 1, data value = 0x1
0xcf7e21ce:    B    #0xcf7e217a (0x3bb17a - libg.so)
```

## Custom implementation
There will be the case in which there is the needed to add additional code or logic to certain instructions hit or before emulation start. It's possible to add a some callbacks provided by a custom python file with the command ``emulator implementation``

```
emu impl .scripts/emucb.py
-> .scripts/emucb.py has been set as custom implementation.
```

the content of emucb.py would be:

```python
# uc expose unicorn emulator, which allow total control on emulation as well.

# on_ready is invoked just before the start of emulation, allowing additional operations
def on_ready(uc, base, entry_point, exit_point):
    print(entry_point)


# on_hook will be invoked at each instruction, right before writing disasm informations to frick html result
def on_hook(uc, offset, address, size):
    print(address)
		
# will be asked before on_ready and will ensure the regions for those offsets are mapped into unicorn
def required_offsets(uc, base):
    return [base + 0x339e82, base + 0x000936FA, base + 0x1da9a4, uc.reg_read(UC_ARM_REG_R0)]
```

# Commands
|   command   |              short              |                                                           info                                                           |
|-------------|---------------------------------|--------------------------------------------------------------------------------------------------------------------------|
|  add        |                                 |  add offset from base 0x0 in arg0 with optional name for this target in other args                                       |
|  attach     |  att                            |  attach to target package name in arg0 with target module name in arg1                                                   |
|  backtrace  |  bt                             |                                                                                                                          |
|  destruct   |  des,ds                         |  read at address arg0 for len arg1 and optional depth arg2                                                               |
|  disasm     |  d,dis                          |  disassemble the given hex payload in arg0 or a pointer in arg0 with len in arg1                                         |
|  emulator   |  e,emu                          |  unicorn emulator                                                                                                        |
|  find       |  f,fi                           |  utilities to find stuffs                                                                                                |
|  functions  |  fn,fu,fun,func,funct,function  |  list native functions                                                                                                   |
|  help       |  h                              |                                                                                                                          |
|  hexdump    |  hd,hdump                       |  hexdump memory regions pointed by value in args for len in the last arg                                                 |
|  info       |  i,in                           |  get information about your target                                                                                       |
|  inject     |  inj                            |  wrapper of dlopen to inject a binary from a local path in arg0 and custom name in arg1                                  |
|  memory     |  m,mem                          |  memory operations                                                                                                       |
|  once       |  o,on                           |  add a callback for ptr target hit in arg0. the keyword 'init' can be used to do stuffs once module base is retrieved.   |
|  pack       |  pa                             |  pack value in arg0 to return a string usable with memory write                                                          |
|  print      |  p,pr                           |                                                                                                                          |
|  quit       |  ex,exit,q                      |                                                                                                                          |
|  registers  |  r,reg,regs                     |  interact with registers                                                                                                 |
|  remove     |  del,delete,rem                 |  remove an offsets from targets list                                                                                     |
|  run        |  c,cont,continue,go,next,start  |  continue the execution of the process to the next target offset                                                         |
|  scripts    |  sc,scr,script                  |  manage custom frida scripts                                                                                             |
|  session    |  s,ss                           |                                                                                                                          |
|  set        |                                 |                                                                                                                          |

## add sub commands
|  command  |   short    |                                       info                                        |
|-----------|------------|-----------------------------------------------------------------------------------|
|  dtinit   |  dti,init  |  mark this target as dt_init function. on android we leak the base before dlopen  |
|  pointer  |  p,ptr     |  add a virtual address in arg0 with optional name in other args                   |

## emulator sub commands
|  command  |  short  |                   info                    |
|-----------|---------|-------------------------------------------|
|  implementation  |  i,impl  |  set a custom unicorn script in arg0      |
|  start    |  s      |  start emulation with exit point in arg0  |

## find sub commands
|  command  |   short    |                                info                                 |
|-----------|------------|---------------------------------------------------------------------|
|  export   |  e,ex,exp  |  find export name arg0 in target module or in optional module arg1  |
|  pattern  |  p,pat     |  search in address arg0 and len in arg1 for pattern in args 2 (deadbeef)  |

## function sub commands
|  command  |  short  |                                              info                                              |
|-----------|---------|------------------------------------------------------------------------------------------------|
|  add      |  a      |  add a native function with pointer in arg0, return type in arg1 followed by args type if any  |
|  run      |  r      |  run native function pointed by arg0 followed by function args                                 |

## info sub commands
|  command  |        short         |                         info                         |
|-----------|----------------------|------------------------------------------------------|
|  modules  |  m,md,mo,mod,module  |  list all modules or single module in optional arg0  |
|  ranges   |  r,range,rg          |  list all ranges or single range in optional arg0    |
|  threads  |  t,th,thread         |  list all threads or single thread with optional tid in rg0  |

## memory sub commands
|  command  |      short      |                                     info                                     |
|-----------|-----------------|------------------------------------------------------------------------------|
|  alloc    |  a,al           |  allocate arg0 size in the heap and return the pointer                       |
|  dump     |  d              |  read bytes in arg0 for len in arg1 and store into filename arg2             |
|  protect  |  p,pr,pro,prot  |  protect address in arg0 for the len arg1 and the prot format in arg2 (rwx)  |
|  read     |  r,rd           |  read bytes from address in arg0 for len in arg1                             |
|  write    |  w,wr           |  write into address arg0 the bytes in args... (de ad be ef)                  |

## memory read sub commands
|    command    |            short             |                                          info                                          |
|---------------|------------------------------|----------------------------------------------------------------------------------------|
|  ansistring   |  ans,ansi,ansistr            |  read ansi string from address in arg0 and optional len in arg1                        |
|  asciistring  |  acs,ascii,asciistr          |  read ascii string from address in arg0 and optional len in arg1                       |
|  byte         |  b                           |  read a signed byte from address in arg0 with optional endianness in arg1 (le/be)      |
|  int          |  i                           |  read a signed int from address in arg0 with optional endianness in arg1 (le/be)       |
|  long         |  l                           |  read a signed long from address in arg0 with optional endianness in arg1 (le/be)      |
|  pointer      |  p,ptr                       |  read a pointer from address in arg0                                                   |
|  short        |  s                           |  read a signed short from address in arg0 with optional endianness in arg1 (le/be)     |
|  ubyte        |  ub                          |  read an unsigned byte from address in arg0 with optional endianness in arg1 (le/be)   |
|  uint         |  ui                          |  read an unsigned int from address in arg0 with optional endianness in arg1 (le/be)    |
|  ulong        |  ul                          |  read an unsigned long from address in arg0 with optional endianness in arg1 (le/be)   |
|  ushort       |  us                          |  read an unsigned short from address in arg0 with optional endianness in arg1 (le/be)  |
|  utf16string  |  u16s,utf16,utf16s,utf16str  |  read utf16 string from address in arg0 and optional len in arg1                       |
|  utf8string   |  u8s,utf8,utf8s,utf8str      |  read utf8 string from address in arg0 and optional len in arg1                        |

## registers sub commands
|  command  |  short  |                  info                   |
|-----------|---------|-----------------------------------------|
|  write    |  w,wr   |  write in register arg0 the value arg1  |

## scripts sub commands
|  command  |  short  |                                      info                                      |
|-----------|---------|--------------------------------------------------------------------------------|
|  load     |  l      |  load the frida script with path in arg0                                       |
|  open     |  o,op   |  create or open a new frida script with name in arg0 and start default editor  |
|  unload   |  u,ul   |  unload the frida script with path in arg0                                     |

## session sub commands
|  command  |  short  |                                           info                                           |
|-----------|---------|------------------------------------------------------------------------------------------|
|  load     |  l,ld   |  load session from previously saved information                                          |
|  save     |  s,sv   |  saves current target offsets, package and module to be immediatly executed with 'load'  |

## set sub commands
|  command   |  short  |           info            |
|------------|---------|---------------------------|
|  capstone  |  cs     |  capstone configurations  |

## set capstone sub commands
|  command  |   short    |              info               |
|-----------|------------|---------------------------------|
|  arch     |  a,ar      |  set the capstone arch in arg0  |
|  mode     |  m,md,mod  |  set the capstone mode in arg0  |
