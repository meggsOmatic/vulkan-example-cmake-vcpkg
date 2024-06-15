# CMakeList.txt : CMake project for vksample, include source and define
# project specific logic here.
#
cmake_minimum_required (VERSION 3.28)

# Enable Hot Reload for MSVC compilers if supported.
if (POLICY CMP0141)
  cmake_policy(SET CMP0141 NEW)
  set(CMAKE_MSVC_DEBUG_INFORMATION_FORMAT "$<IF:$<AND:$<C_COMPILER_ID:MSVC>,$<CXX_COMPILER_ID:MSVC>>,$<$<CONFIG:Debug,RelWithDebInfo>:EditAndContinue>,$<$<CONFIG:Debug,RelWithDebInfo>:ProgramDatabase>>")
endif()

project ("vksample")

# Add source to this project's executable.
add_executable (vksample "vksample.cpp" "vksample.h")

# https://cmake.org/cmake/help/latest/module/FindVulkan.html
find_package(Vulkan REQUIRED)
target_link_libraries(vksample PRIVATE Vulkan::Vulkan)

find_package(glfw3 CONFIG REQUIRED)
target_link_libraries(vksample PRIVATE glfw)

find_package(Stb REQUIRED)
target_include_directories(vksample PRIVATE ${Stb_INCLUDE_DIR})

find_package(glm CONFIG REQUIRED)
target_link_libraries(vksample PRIVATE glm::glm)

find_package(tinyobjloader CONFIG REQUIRED)
target_link_libraries(vksample PRIVATE tinyobjloader::tinyobjloader)

set_property(TARGET vksample PROPERTY CXX_STANDARD 20)

# TODO: Add tests and install targets if needed.
