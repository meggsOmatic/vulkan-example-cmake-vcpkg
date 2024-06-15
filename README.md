# Vulkan CMake VCPkg Example

This is a cleanly setup project to configure and build the [Multisampling Example](https://vulkan-tutorial.com/Multisampling) from the [Vulkan Tutorial Website](https://vulkan-tutorial.com/), using CMake and VCPkg, with all dependencies set up correctly. The goal is to have something that will *Just Work* so you can focus on the Vulkan stuff instead of the configuration stuff.

I have intentionally contributed nothing notable to the C++ and shader code, which is as close to unmodified as possible. The interesting/useful bits here are the build and dependency configuration. Hopefully this will serve as a starting point to clone your own empty Vulkan project.

The Multisample tutorial was chosen because it was the last sample before the Vulkan tutorial switched to compute shaders. It has all the bits to load models and textures and get them on-screen.

<img src="https://github.com/meggsOmatic/vulkan-example-cmake-vcpkg/assets/5649419/c5a7c0c1-8c2a-4ed1-92c8-11b140d7d321" height=50% width=50% >

## How to use
On Windows, the steps to make this *Just Work* are:

1. Install Visual Studio 2022 Community Edition. Make sure you include the desktop C++ tools, VCPkg, CMake, and Git for Windows.
2. Open a Visual Studio terminal window (i.e. with vcvars included) and run `vcpkg integrate install` if you never have.
3. Clone the repository. Just this repository.
4. Open the repo's directory in Visual Studio as a CMake project.
5. Wait a bit for vcpkg to fetch and resolve and build the dependencies.
6. Hit [F5] to build the vksample.exe project itself.

## How it works

STB, GLFW, GLM, GLSLC, the Vulkan SDK, and the Vulkan validation layers are all downloaded and set up through VCPkg. 
```
{
  "dependencies": [
    "glfw3",
    "glm",
    "stb",
    "tinyobjloader",
    "vulkan-sdk-components",
    "vulkan-validationlayers"
  ]
}
```

The correct dependencies for all of them are all in the CMakeLists.txt file.
```
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
```

CMake uses GLSLC to build the shaders and deploy the SPIR-V bytecode to the output directory, and rebuild them as needed if you change the shader source.
```
set(SHADER_OUTPUT "")
function(add_shader SRCNAME DSTNAME)
  set(SRCPATH "${CMAKE_SOURCE_DIR}/shaders/${SRCNAME}")
  set(DSTPATH "${CMAKE_CURRENT_BINARY_DIR}/shaders/${DSTNAME}")
  add_custom_command(
    OUTPUT "${DSTPATH}"
    MAIN_DEPENDENCY "${SRCPATH}"
    COMMAND "${PACKAGE_PREFIX_DIR}/tools/shaderc/glslc.exe" "${SRCPATH}" -o "${DSTPATH}")
  list(APPEND SHADER_OUTPUT "${DSTPATH}")
  set(SHADER_OUTPUT "${SHADER_OUTPUT}" PARENT_SCOPE)
endfunction()
add_shader("shader.vert" "vert.spv")
add_shader("shader.frag" "frag.spv")
add_custom_target(shader_bytecode DEPENDS ${SHADER_OUTPUT})
add_dependencies(vksample shader_bytecode)
```

CMake also copies the contents of the `models` and `textures` directories to your output directory as part of the build process, where they can be loaded by the C++ code.
```
add_custom_command(TARGET vksample POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory
  "${CMAKE_SOURCE_DIR}/textures"
  "${CMAKE_CURRENT_BINARY_DIR}/textures")

add_custom_command(TARGET vksample POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory
  "${CMAKE_SOURCE_DIR}/models"
  "${CMAKE_CURRENT_BINARY_DIR}/models")
```

Note that the models directory has a special exclusion in `.gitignore` so that `*.obj` files there are NOT ignored. Unfortunately, `.obj` is an extension for both compiled object code and mesh objects, and we don't want to ignore our meshes!
```
# We exclude .obj in general, but we want them under the models directory
!models/*.obj
```

## Finding the Vulkan Validation Layers

The Vulkan Validation layers are a slight pain, because they require you to set a `VK_ADD_LAYER_PATH` environment variable to point to the DLLs. When the SDK comes in via VCPkg that's not going to be in a consistent place on your system. To work around that, I made a single change to the tutorial source code. At the top of the `main()` function, before loading Vulkan, I set that environment variable (only for the current project) based on your launch path:
```
int main() {
  auto path = std::filesystem::current_path() / "vcpkg_installed" /
              "x64-windows" / "bin";
  std::string set = "VK_ADD_LAYER_PATH=" + path.string();
  _putenv(set.c_str());
```

It's ugly, and it'll need to change for targets other than x64-windows. I don't have such targets to test on, but would welcome a pull request from you!
