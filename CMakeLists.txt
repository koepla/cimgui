# cimgui project configuration

cmake_minimum_required(VERSION 3.11)
project(cimgui VERSION 0.0.1)

set(CMAKE_CXX_STANDARD 11)

file(GLOB CIMGUI_SOURCE_FILES
        # imgui
        "cimgui/cimgui.cpp"
        "cimgui/imgui/imgui.cpp"
        "cimgui/imgui/imgui_draw.cpp"
        "cimgui/imgui/imgui_demo.cpp"
        "cimgui/imgui/imgui_widgets.cpp"
        "cimgui/imgui/imgui_tables.cpp"
        "cimgui/imgui/backends/imgui_impl_opengl3.cpp"
        "cimgui/imgui/backends/imgui_impl_glfw.cpp"

        # imnodes
        "cimnodes/cimnodes.cpp"
        "cimnodes/imnodes/imnodes.cpp"

        # implot
        "cimplot/cimplot.cpp"
        "cimplot/implot/implot.cpp"
        "cimplot/implot/implot_items.cpp"
        "cimplot/implot/implot_demo.cpp"
)

file(GLOB CIMGUI_HEADER_FILES
        # imgui
        "cimgui/cimgui.h"
        "cimgui/imgui/imgui.h"
        "cimgui/imgui/imgui_internal.h"

        # imnodes
        "cimnodes/cimnodes.h"
        "cimnodes/imnodes/imnodes.h"
        "cimnodes/imnodes/imnodes_internal.h"

        # implot
        "cimplot/cimplot.h"
        "cimplot/implot/implot.h"
        "cimplot/implot/implot_internal.h"
)

include(FetchContent)
FetchContent_Declare(glfw GIT_REPOSITORY https://github.com/glfw/glfw.git)
FetchContent_MakeAvailable(glfw)

add_library(${PROJECT_NAME} STATIC ${CIMGUI_SOURCE_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${CIMGUI_HEADER_FILES}")

target_link_libraries(${PROJECT_NAME} PUBLIC glfw)
target_include_directories(${PROJECT_NAME} PUBLIC "cimgui/imgui/")
target_include_directories(${PROJECT_NAME} PUBLIC "cimgui/generator/output")
target_compile_definitions(${PROJECT_NAME} PUBLIC CIMGUI_USE_GLFW=1 CIMGUI_USE_OPENGL3=1 IMNODES_NAMESPACE=imnodes)

if (WIN32)
    target_compile_definitions(${PROJECT_NAME} PUBLIC "-DIMGUI_IMPL_API=extern \"C\" __declspec\(dllexport\)")
else(WIN32)
    target_compile_definitions(${PROJECT_NAME} PUBLIC "-DIMGUI_IMPL_API=extern \"C\" ")
endif(WIN32)

# make sure that we, and projects that use us have the correct directories
target_include_directories(${PROJECT_NAME}
        PUBLIC
        ${CMAKE_CURRENT_LIST_DIR}/cimgui
        ${CMAKE_CURRENT_LIST_DIR}/cimnodes
        ${CMAKE_CURRENT_LIST_DIR}/cimplot
)

# the 'install'
include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME}
    EXPORT "${PROJECT_NAME}Targets"
)

# in order to preserve the paths, we copy the headers manually
foreach(HEADER ${CIMGUI_HEADER_FILES})
    file(RELATIVE_PATH HEADER_FILE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}" "${HEADER}")
    get_filename_component(HEADER_DIRECTORY_PATH "${HEADER_FILE_PATH}" DIRECTORY)
    install(
            FILES ${HEADER}
            DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${HEADER_DIRECTORY_PATH}"
    )
endforeach()
