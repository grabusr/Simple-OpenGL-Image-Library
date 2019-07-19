cmake_minimum_required(VERSION 3.10)

set(LIB_NAME SoilLib)

project(${LIB_NAME})

option(SOIL_BUILD_TESTS "Build tests" OFF)

if (SOIL_BUILD_TESTS)
    add_subdirectory(test)
endif()

if (ANDROID)
    # Android requires GL ES 2.0 package automatically
    find_library(OpenGLES2 NAMES GLESv2)
else ()
    find_package(OpenGL REQUIRED)
endif ()

aux_source_directory(src SOIL_SOURCES)

add_library(${LIB_NAME} STATIC ${SOIL_SOURCES})

if (ANDROID)
    target_link_libraries(${LIB_NAME} PUBLIC ${OpenGLES2})
elseif()
    target_link_libraries(${LIB_NAME} PUBLIC OpenGL::GL)
endif()

#added flag to make adding static lib to shared lib
target_compile_options(${LIB_NAME} PUBLIC -fPIC)

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
