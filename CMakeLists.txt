list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_SOURCE_DIR}/bldsys/cmake/module")

if(NOT DEFINED CMAKE_C_COMPILER_LAUNCHER AND NOT DEFINED CMAKE_CXX_COMPILER_LAUNCHER)
    find_program(CCACHE_FOUND ccache)
    find_program(SCCACHE_FOUND sccache)
    if (SCCACHE_FOUND)
        message("setting SCCACHE to ${SCCACHE_FOUND}")
        set(CMAKE_C_COMPILER_LAUNCHER ${SCCACHE_FOUND})
        set(CMAKE_CXX_COMPILER_LAUNCHER ${SCCACHE_FOUND})
    elseif(CCACHE_FOUND)
        message("setting CCACHE to ${CCACHE_FOUND}")
        set(CMAKE_C_COMPILER_LAUNCHER ${CCACHE_FOUND})
        set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE_FOUND})
    endif()
endif()

# Cmake 最低版本号
cmake_minimum_required(VERSION 3.15)

# set the project name
project(VulkanTutorialSample VERSION 1.1)

# 配置Vulkan环境
find_package(Vulkan)

if(Vulkan_FOUND)
    if(NOT Vulkan_dxc_exe_FOUND)
        find_program(Vulkan_dxc_EXECUTABLE
                    NAMES dxc
                    HINTS
                    "$ENV{VULKAN_SDK}/Bin"
                    "$ENV{VULKAN_SDK}/bin"
                    )
    endif()
    if(Vulkan_dxc_EXECUTABLE)
        message(STATUS "Found DirectX Shader Compiler under ${Vulkan_dxc_EXECUTABLE}")
    else()
        message(STATUS "Couldn't find DirectX Shader Compiler executable, make sure it is present in Vulkan SDK or add it manually via Vulkan_dxc_EXECUTABLE cmake variable. HLSL shaders won't be compiled.")
    endif()
    # slang compiler
    if(NOT Vulkan_slang_exe_FOUND)
        find_program(Vulkan_slang_EXECUTABLE
                    NAMES slangc
                    HINTS
                    "$ENV{VULKAN_SDK}/Bin"
                    "$ENV{VULKAN_SDK}/bin"
                    )
    endif()
    if(Vulkan_slang_EXECUTABLE)
        message(STATUS "Found slang Shader Compiler under ${Vulkan_slang_EXECUTABLE}")
    else()
        message(STATUS "Couldn't find slang Shader Compiler executable, make sure it is present in Vulkan SDK or add it manually via Vulkan_slang_EXECUTABLE cmake variable. Slang shaders won't be compiled.")
    endif()
endif()

# 定义 vulkan 接口库（可选，实际可直接使用 Vulkan::Vulkan）
add_library(vulkan INTERFACE)
# target_include_directories(Vulkan_INCLUDE_DIRS)
# target_link_libraries(Vulkan_LIBRARIES)  # 链接到系统Vulkan库

# 编译器选择
# globally add VKB_DEBUG for the debug build
add_compile_definitions($<$<CONFIG:DEBUG>:VKB_DEBUG>)

# globally set -fno-strict-aliasing, needed due to using reinterpret_cast
if (NOT MSVC)
  add_compile_options(-fno-strict-aliasing)
endif()

if(MSVC AND (DEFINED CMAKE_C_COMPILER_LAUNCHER))
    message(DEBUG "Setting MSVC flags to /Z7 for ccache compatibility.  Current flags: ${CMAKE_CXX_FLAGS_DEBUG}")
    string(REPLACE "/Zi" "/Z7" CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")
    string(REPLACE "/Zi" "/Z7" CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG}")
    string(REPLACE "/Zi" "/Z7" CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO}")
    string(REPLACE "/Zi" "/Z7" CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO}")
    message(DEBUG "New flags: ${CMAKE_CXX_FLAGS_DEBUG}")
endif()

# create output folder
file(MAKE_DIRECTORY output)

# Add path for local cmake scripts
list(APPEND CMAKE_MODULE_PATH
    ${CMAKE_CURRENT_SOURCE_DIR}/bldsys/cmake
    ${CMAKE_CURRENT_SOURCE_DIR}/bldsys/cmake/module)


# Add third party libraries
add_subdirectory(third_party)

SET(SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/VulkanTutorialSample/main.cpp)

# add the executable
add_executable(${PROJECT_NAME} ${SRC_LIST})

target_include_directories(${PROJECT_NAME} SYSTEM INTERFACE ${Vulkan_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} INTERFACE Vulkan::Vulkan)  # 链接到系统Vulkan库

