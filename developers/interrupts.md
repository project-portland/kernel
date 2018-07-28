# Interrupt listing
## Notes
See also [the Portland C Library](https://github.com/benjidial/portland-c-library).  The inline assembly shows examples of how to call these from assembly, and using the library makes calling these from C much easier and less bulky.

## Listing
* `0x40` - `0x47`: [PFS driver](#pfs-driver)
* `0x48` - `0x4f`: [VGA driver](#vga-driver)
* `0x50` - `0x57`: [Keyboard driver](#keyboard-driver)
* `0x58` - `0x5f`: [Memory management](#memory-management)
* `0x60` - `0xff`: Reserved

---
## PFS driver
* `0x40`: read from a file
  * `eax`: starting position in file
  * `ebx`: pointer to [file structure](../structures#file)
  * `cx`: number of bytes to read
  * `edx`: pointer to buffer to read into
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: corrupted file structure or file was deleted
    * `0x10`: `ebx` and/or `edx` are/is `0x00000000`
    * `0x11`: file not that big

* `0x41`: write to a file
  * `eax`: starting position in file
  * `ebx`: pointer to [file structure](structures#file)
  * `cx`: number of bytes to write
  * `edx`: pointer to buffer to read from
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: corrupted file structure or file was deleted
    * `0x10`: `ebx` and/or `edx` are/is `0x00000000`
    * `0x11`: file not that big

* `0x42`: create and open a file
  * `al`: number of reserved sectors
  * `ebx`: pointer to buffer for [file structure](structures#file)
  * `cx`: size of file in bytes
  * `edx`: pointer to [byte string](structures#byte-string) of name of file
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: not enough room on drive
    * `0x10`: `ebx` and/or `edx` are/is `0x00000000`
    * `0x11`: `cx` > `al` \* 512

* `0x43`: open a file
  * `ebx`: pointer to buffer for [file structure](structures#file)
  * `edx`: pointer to [bytes string](structures#byte-string) of name of file
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: file does not exist
    * `0x10`: `ebx` and/or `edx` are/is `0x00000000`

* `0x44`: resize a file
  * `al`: number of reserved sectors
  * `cx`: size in bytes
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: not enough room on drive
    * `0x10`: `cx` > `al` \* 512

* `0x45`: delete a file
  * `ebx`: pointer to [bytes string](structures#byte-string) of name of file
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: file does not exist
    * `0x10`: `ebx` is `0x00000000`

* `0x46`: run file
  * `ebx`: pointer to [bytes string](structures#byte-string) of name of file
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: not enough memory
    * `0x03`: file does not exist
    * `0x10`: `ebx` is `0x00000000`
  * `edx` after:
    * `eax` at end of program execution

* `0x47`: get info about a file
  * `ebx`: pointer to buffer for [file info structure](structures#file-info)
  * `edx`: pointer to [bytes string](structures#byte-string) of name of file
  * `al` after:
    * `0x00`: success
    * `0x01`: I/O error
    * `0x02`: file does not exist
    * `0x10`: `ebx` and/or `edx` are/is `0x00000000`

## VGA driver
* `0x48`: print byte
  * `al`: byte to print (reserved if `0x00`)

* `0x49`: print word
  * `ax`: word to print (reserved if `0xXX00`)

* `0x4a`: print [byte string](structures#byte-string)
  * `ebx`: pointer to byte string to print
  * `al` after:
    * `0x00`: success
    * `0x10`: `ebx` is `0x00000000`

* `0x4b`: print [word string](structures#word-string)
  * `ebx`: pointer to word string to print
  * `al` after:
    * `0x00`: success
    * `0x10`: `ebx` is `0x00000000`

* `0x4c`: clear the screen

* `0x4d`: set color
  * `al`: color

* `0x4e`: move cursor
  * `al`: column
  * `ah`: row

* `0x4f`: reserved

## Keyboard driver
* `0x50`: get character from keyboard
  * `al`:
    * `0x00`: blocking
    * `0x01`: nonblocking
  * `al` after:
    * `0x00`: success
    * `0x01`: `al` is a bad value
  * `dl` after: character (`0x00` if blocking and none was available)

* `0x51`: get [string](structures#byte-string) from keyboard
  * `al`:
    * `0x00`: blocking
    * `0x01`: nonblocking
  * `ebx`: pointer to buffer to put string into
  * `cx`: string length
  * `al` after:
    * `0x00`: success
    * `0x10`: `ebx` is `0x00000000` or `al` is a bad value

* `0x52`: get line from keyboard
  * `al`:
    * `0x00`: blocking
    * `0x01`: nonblocking
  * `ebx`: pointer to buffer to put [string](structures#byte-string) into
  * `cx`: maximum string length
  * `al` after:
    * `0x00`: success
    * `0x01`: `ebx` is `0x00000000` or `al` is a bad value

* `0x53`: put character into keyboard buffer
  * `al`: character

* `0x54`: put string into keyboard buffer
  * `ebx`: pointer to buffer to put [string](structures#byte-string) into
  * `al` after:
    * `0x00`: success
    * `0x01`: `ebx` is `0x00000000`

* `0x55` - `0x57`: reserved

## Memory managerment
* `0x58`: allocate memory
  * `ax`: number of bytes to allocate
  * `ebx` after: pointer to allocated memory (`0x00000000` not enough memory or if `ax` is `0x0000`)

* `0x59`: deallocate memory
  * `ebx`: pointer to allocated memory
  * `al` after:
    * `0x00`: success
    * `0x10`: `ebx` is `0x00000000`
    * `0x11`: `ebx` does not point to the start of an allocated block

* `0x5a`: check available memory
  * `eax` after: memory available in bytes
  * `edx` after: length of largest contiguous area of available memory in bytes

* `0x5b` - `0x5f`: reserved

---
**[Back to index](index)**