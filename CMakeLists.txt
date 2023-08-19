cmake_minimum_required(VERSION 3.1.3 FATAL_ERROR)

project(gl3w)

set(CMAKE_VERBOSE_MAKEFILE false)
set(CMAKE_POSITION_INDEPENDENT_CODE ON) # -fPIC

set(SOURCE_DIR src)
set(HEADER_DIR include)

set(HEADER_FILES
	"${HEADER_DIR}/GL/gl3w.h"
	"${HEADER_DIR}/GL/glcorearb.h"
	"${HEADER_DIR}/KHR/khrplatform.h"
)

set(SOURCE_FILES
	"${SOURCE_DIR}/gl3w.c"
)

# add and depend on OpenGL
find_package(OpenGL REQUIRED)

# find python interpreter
find_package(PythonInterp REQUIRED)

if(CMAKE_VERSION VERSION_LESS 3.8.0)
	set(EXTERNAL_INCLUDE_DIRS ${EXTERNAL_INCLUDE_DIRS} ${OPENGL_INCLUDE_DIR})
	set(EXTERNAL_LIBRARIES ${EXTERNAL_LIBRARIES} ${OPENGL_LIBRARIES})
else()
	# Since CMake 3.8 the IMPORTED targets available
	set(EXTERNAL_LIBRARIES ${EXTERNAL_LIBRARIES} OpenGL::GL OpenGL::GLU)
endif()
# add command to create the header and source files
add_custom_command(
	OUTPUT
		${SOURCE_DIR}/gl3w.c
		${HEADER_DIR}/GL/gl3w.h
		${HEADER_DIR}/GL/glcorearb.h
		${HEADER_DIR}/KHR/khrplatform.h
	COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_CURRENT_SOURCE_DIR}/gl3w_gen.py
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/gl3w_gen.py
)

# create gl3w target
add_library(${PROJECT_NAME} STATIC
	${CMAKE_CURRENT_BINARY_DIR}/${SOURCE_DIR}/gl3w.c
	${CMAKE_CURRENT_BINARY_DIR}/${HEADER_DIR}/GL/gl3w.h
	${CMAKE_CURRENT_BINARY_DIR}/${HEADER_DIR}/GL/glcorearb.h
	${CMAKE_CURRENT_BINARY_DIR}/${HEADER_DIR}/KHR/khrplatform.h
)

target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/${HEADER_DIR})
