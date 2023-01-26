Demonstration of using interface targets to encapsulate common header files and support
exposing them as PCH components.

Use Case
--------

Say you have a large library with many very common includes that routinely slow down
your build times.

Rather than every down-stream target manually adding you as a list of includes to their
PCH definition, you can either create a "OurPCH.h" or you can do everything in one
place (the build system) and use cmake's interface libraries as a property bag to
make these things inheritable.


a- Add the PCH files as PUBLIC/INTERFACE to a target,
b- link the target privately to upstream targets.


Doing the experiment
--------------------

My standard recommended build method is:

```
> cmake -G Ninja -DCMAKE_VERBOSE_MAKEFILE=ON -S . -B build
> cmake --build ./build
```

Review the top-level CMakeLists.txt file, and note the commented-out line:

```
# But uncomment this to make the executable also use config.h in its pch.
#target_link_libraries (my-exe project-config-pch)
```

Build the project as-is, then ook for `cmake_pch.hxx` usually in `build/CMakeFiles/my-exe.dir`,
and note it has one include.

Now edit the top-level CMakeLists.txt file, and uncomment the `target_link_libraries` pch line:

```
# But uncomment this to make the executable also use config.h in its pch.
target_link_libraries (my-exe project-config-pch)
```

Save and rebuild, now revisit `my-exe/cmake_pch.hxx`. You'll see that `config.h` was inherited
up into the PCH file list for the executable.

What did we see here, then?

lib1 _privately_ associated with project-config-pch, so that connection was not exposed to
my-exe; therefore despite linking to lib1, myexe does not automatically get config.h as a pch
component. When we then added the association directly to myexe, we also gained the pch
component.
