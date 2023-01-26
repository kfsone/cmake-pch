cmake_minimum_required (VERSION 3.18)
project (cmake-pch LANGUAGES CXX)

set (CMAKE_CXX_STANDARD 11)
set (CMAKE_CXX_STANDARD_REQUIRED ON)

# Path to our include folder.
set (_inc_dir "${CMAKE_CURRENT_LIST_DIR}/include")

# CMake uses the obscure 'INTERFACE library' as property bags
add_library (project-config INTERFACE)
# These will be inherited by anyone who links against us, which is an oddly build-centric
# way to think about it.
target_include_directories (project-config INTERFACE ${_inc_dir})
# we depend on our source file
target_sources (project-config INTERFACE ${_inc_dir}/config.h)

# and a second one for the pch
add_library (project-config-pch INTERFACE)
# and it gets used as our precompiled header.
target_precompile_headers (project-config-pch INTERFACE ${_inc_dir}/config.h)

# Now for our actual libraries.
add_library (lib1 src/lib1.cpp include/lib1.h)

# Make the association and inherit properties
target_link_libraries (lib1 PUBLIC project-config)
# And privately make use of the pch without exposing it
target_link_libraries (lib1 PRIVATE project-config-pch)

target_precompile_headers (lib1 PRIVATE include/lib1.h)


# And our executable
add_executable (my-exe src/main.cpp include/main.h)

target_precompile_headers (my-exe PRIVATE include/main.h)

# This does not have any effect on our pch
target_link_libraries (my-exe lib1)

# But uncomment this to make the executable also use config.h in its pch.
#target_link_libraries (my-exe project-config-pch)

unset (_inc_dir)
