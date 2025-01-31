# Main library of the native side of things (non-Godot using)
add_library(thrive_native SHARED
  "${PROJECT_BINARY_DIR}/Include.h" core/ForwardDefinitions.hpp
  interop/CInterop.cpp interop/CInterop.h
  interop/CStructures.h interop/JoltTypeConversions.hpp
  core/Logger.cpp core/Logger.hpp
  core/Mutex.hpp
  core/NonCopyable.hpp core/Reference.hpp
  core/RefCounted.hpp
  core/Spinlock.hpp
  # core/TaskSystem.cpp core/TaskSystem.hpp
  core/Time.hpp
  physics/BodyActivationListener.cpp physics/BodyActivationListener.hpp
  physics/BodyControlState.hpp
  physics/ContactListener.cpp physics/ContactListener.hpp
  physics/CustomConstraintTypes.hpp
  physics/Layers.hpp
  physics/PhysicalWorld.cpp physics/PhysicalWorld.hpp
  physics/PhysicsBody.cpp physics/PhysicsBody.hpp
  physics/ShapeCreator.cpp physics/ShapeCreator.hpp
  physics/ShapeWrapper.cpp physics/ShapeWrapper.hpp
  physics/SimpleShapes.cpp physics/SimpleShapes.hpp
  physics/TrackedConstraint.cpp physics/TrackedConstraint.hpp
  physics/StepListener.cpp physics/StepListener.hpp
  physics/DebugDrawForwarder.cpp physics/DebugDrawForwarder.hpp
  physics/PhysicsCollision.hpp
  physics/PhysicsRayWithUserData.hpp
  physics/ArrayRayCollector.hpp)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_compile_options(thrive_native PRIVATE /W4 /wd4068)

  if(WARNINGS_AS_ERRORS)
    target_compile_options(thrive_native PRIVATE /WX)
  endif()
else()
  target_compile_options(thrive_native PRIVATE -Wall -Wextra -Wpedantic -Werror
    -Wno-unknown-pragmas)

  if(WARNINGS_AS_ERRORS)
    target_compile_options(thrive_native PRIVATE -Werror)
  endif()
endif()


target_compile_definitions(thrive_native PRIVATE THRIVE_NATIVE_BUILD)

target_link_libraries(thrive_native PRIVATE Jolt)
target_link_libraries(thrive_native PUBLIC Boost::intrusive
  Boost::circular_buffer Boost::pool)

target_include_directories(thrive_native PUBLIC ${CMAKE_CURRENT_LIST_DIR})

# target_include_directories(thrive_native PUBLIC "../../third_party/JoltPhysics/")

# TODO: the one private precompiled header will probably cause issues with
# other libraries linking to this
target_precompile_headers(thrive_native PUBLIC "${PROJECT_BINARY_DIR}/Include.h"
  "core/ForwardDefinitions.hpp" "core/RefCounted.hpp"
  PRIVATE "../../third_party/JoltPhysics/Jolt/Jolt.h")

set_target_properties(thrive_native PROPERTIES
  CXX_STANDARD 20
  CXX_EXTENSIONS OFF)
  
if(UNIX OR CMAKE_GENERATOR MATCHES "Visual Studio")
  # This seems to fail native Windows builds (when trying to use msys2)
  set_target_properties(thrive_native PROPERTIES INTERPROCEDURAL_OPTIMIZATION ON)
else()
  message(STATUS "Not enabling interprocedural optimization (lto) "
    "as it is probably not working on this platform")
endif()

message(STATUS "TODO: static standard lib for easier distributing (make sure it is the clang one)")
# # Static standard lib
# target_link_libraries(thrive_native PRIVATE -static-libgcc -static-libstdc++)

# # Fully static executable
# if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
#   target_link_libraries(thrive_native PRIVATE -static)
# endif()

# if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
#   set_target_properties(thrive_native PROPERTIES LINK_FLAGS_RELEASE "-fuse-ld=lld")
# elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
#   set_target_properties(thrive_native PROPERTIES LINK_FLAGS_RELEASE "-fuse-ld=gold")
# endif()

if(NOT WIN32)
  set_target_properties(thrive_native PROPERTIES LINK_FLAGS_RELEASE "-fuse-ld=gold")
else()
  # set_target_properties(thrive_native PROPERTIES LINK_FLAGS_RELEASE "")
endif()

install(TARGETS thrive_native)
