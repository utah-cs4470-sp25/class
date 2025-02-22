Running the C Code
===

To run generated program `file.c`:

1. Delete the "Compilation succeeded" line from the bottom of the program.
2. Make sure there is a folder `./rt/` with a recent copy of the runtime code
   - <https://github.com/utah-cs4470-sp25/runtime>
   - **Feb 21**: we are actively pushing to the runtime repo, if something breaks, try pulling
3. Compile using the right compiler and options ...
   - Prof. Ben uses `make c TEST=file.c` where the "c" target runs `clang`:
   - `clang -I/opt/homebrew/include -L/opt/homebrew/lib -lz -lpng16 $(TEST) rt/runtime.c rt/pngstuff.c`
   - the `-I` option points to a folder containing `png.h`
   - the `-L` option points to a folder containing `png.c`
   - the other instructions might just work on another machine
4. Run `./a.out`


### FAQ


#### Q. Undefined symbol error, `_main`

```
Undefined symbols for architecture arm64:
  "_main", referenced from:
     implicit entry/start for main executable
     (maybe you meant: _jpl_main)
```

You need to compile `rt/runtime.c` along with your `file.c`


#### Q. Undefined symbol error, `__readPNG`

```
Undefined symbols for architecture arm64:
  "__readPNG", referenced from:
      _read_image in runtime-2520cb.o
```

You need to compile `rt/pngstuff.c` along with your `file.c` and `rt/runtime.c`


#### Q. Undefined symbol error, 

```
rt/pngstuff.c:14:10: fatal error: 'png.h' file not found
#include <png.h>
```

You need to add `-I` and `-L` flags with folders that contain `png.h` and `png.c`.
Search your computer for those programs. If that fails, try installing `libpng`.

