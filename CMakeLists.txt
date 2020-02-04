cmake_minimum_required(VERSION 3.10)

set(LIB_NAME SoilLib)

project(${LIB_NAME})

option(SOIL_BUILD_TESTS "Build tests" OFF)

if (SOIL_BUILD_TESTS)
    add_subdirectory(test)
endif()


# -------------------------------------------
# Configure some Apple specific build options
# -------------------------------------------
IF(APPLE)
    SET(CMAKE_OSX_ARCHITECTURES x86_64;i386)
    SET(CMAKE_OSX_DEPLOYMENT_TARGET 10.7)
    SET(CMAKE_XCODE_ATTRIBUTE_SUPPORTED_PLATFORMS macosx)
    SET(CMAKE_XCODE_ATTRIBUTE_VALID_ARCHS "i386 x86_64")

    # Xcode Generator
    IF(XCODE_VERSION)
        SET(CMAKE_OSX_SYSROOT macosx)
        # Makefiles or other
        # Need the full path to the SDK.
    ELSE(XCODE_VERSION)
        execute_process(COMMAND xcodebuild -sdk macosx -version Path | head -n 1 OUTPUT_VARIABLE CMAKE_OSX_SYSROOT)
        string(REGEX REPLACE "(\r?\n)+$" "" CMAKE_OSX_SYSROOT "${CMAKE_OSX_SYSROOT}")
    ENDIF(XCODE_VERSION)
ENDIF(APPLE)


# -------------------------------------------
# Libraries
# -------------------------------------------
IF(APPLE)
    FIND_LIBRARY(COREFOUNDATION_LIBRARY NAMES CoreFoundation)
    find_package(OpenGL REQUIRED)
#    FIND_LIBRARY(OPENGL_LIBRARY NAMES OpenGL)
elseif (ANDROID)
    # Android requires GL ES 2.0 package automatically
    find_library(OpenGLES2 NAMES GLESv2)
else ()
    find_package(OpenGL REQUIRED)
endif ()

aux_source_directory(src SOIL_SOURCES)

add_library(${LIB_NAME} STATIC ${SOIL_SOURCES})
#add_library(${LIB_NAME} SHARED ${SOIL_SOURCES})

if (ANDROID)
    target_link_libraries(${LIB_NAME} PUBLIC ${OpenGLES2})
else()
    target_link_libraries(${LIB_NAME} PUBLIC OpenGL::GL ${COREFOUNDATION_LIBRARY})
endif()

#added flag to make adding static lib to shared lib
#target_compile_options(${LIB_NAME} PUBLIC -fPIC)

target_include_directories(${LIB_NAME} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
                                              $<INSTALL_INTERFACE:include/SOIL>)

install(TARGETS ${LIB_NAME}
        EXPORT SoilTarget
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

set(SOIL_HEADERS src/image_DXT.h
        src/image_helper.h
        src/SOIL.h
        src/stb_image_aug.h
        src/stbi_DDS_aug.h
        src/stbi_DDS_aug_c.h)

install(FILES ${SOIL_HEADERS} DESTINATION include/SOIL)
