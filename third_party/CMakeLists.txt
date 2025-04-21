cmake_minimum_required(VERSION 3.10)

project(third_party LANGUAGES C CXX)

# glfw
if(NOT ANDROID AND NOT IOS)
    if (NOT DIRECT_TO_DISPLAY)
        # GLFW
        option(GLFW_BUILD_DOCS OFF)
        option(GLFW_BUILD_TESTS OFF)
        option(GLFW_BUILD_EXAMPLES OFF)
        option(GLFW_BUILD_TESTS OFF)
        option(GLFW_INSTALL OFF)
        option(BUILD_SHARED_LIBS ON)
        if (VKB_WSI_SELECTION STREQUAL WAYLAND)
            set(GLFW_BUILD_WAYLAND ON)
        endif()

        add_subdirectory(glfw)

        if (TARGET update_mappings)
            set_target_properties(update_mappings PROPERTIES FOLDER "ThirdParty")
        endif()
        set_property(TARGET glfw PROPERTY FOLDER "ThirdParty")
    endif()
endif()