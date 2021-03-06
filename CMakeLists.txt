#/CMakeLists.txt
cmake_minimum_required (VERSION 2.8.4)
project (iceball)

set(CMAKE_SOURCE_DIR src)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

if (CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
	add_definitions(-fno-strict-aliasing -Wall -Wextra -g) # keep debugging symbols even in Release builds
endif ()

include_directories(include)
if (WIN32)
	if (MSVC)
		set_source_files_properties(filename.c PROPERTIES LANGUAGE CXX )
	endif (MSVC)
endif (WIN32)

if (MINGW)
	set(CMAKE_PREFIX_PATH "dist/mingw/enet;dist/mingw/sdl2;dist/mingw/lua51;dist/mingw/sackit;dist/mingw/zlib;dist/mingw/glew" CACHE PATH "" FORCE)
elseif (MSVC)
	set(CMAKE_PREFIX_PATH "dist/msvc/enet;dist/msvc/sdl2;dist/msvc/lua51;dist/msvc/sackit;dist/msvc/zlib;dist/msvc/glew" CACHE PATH "" FORCE)
endif ()

find_package(ENet REQUIRED)
find_package(SDL2 REQUIRED)
find_package(zlib REQUIRED)
find_package(Lua REQUIRED)
find_package(sackit REQUIRED)
find_package(GLEW REQUIRED)
find_package(OpenGL REQUIRED)

include_directories(
	${ENet_INCLUDE_DIRS}
	${sackit_INCLUDE_DIRS}
	${ZLIB_INCLUDE_DIRS}
	${GLEW_INCLUDE_DIRS}
	${SDL2_INCLUDE_DIR}
	${LUA_INCLUDE_DIR}
)

file(GLOB LUA_FILES src/lua*)
set(MAIN_FILES
	src/dsp.c
	src/img.c
	src/json.c
	src/logtxt.c
	src/main.c
	src/map.c
	src/model.c
	src/network.c
	src/path.c
	src/vecmath.c
	src/wav.c
	src/png.c
)

set(GL_FILES
	src/gl/render.c
	src/gl/render_img.c
)

source_group(gl FILES ${GL_FILES})

source_group(lua FILES ${LUA_FILES})

add_custom_command(TARGET iceball POST_BUILD           # Adds a post-build event to MyTest
	COMMAND ${CMAKE_COMMAND} -E copy_if_different  # which executes "cmake - E copy_if_different..."
	"${LUA_LIBRARY}"      # <--this is in-file
	$<TARGET_FILE_DIR:MyTest>)                 # <--this is out-file path

# iceball target
add_executable(iceball ${MAIN_FILES} ${LUA_FILES} ${GL_FILES})
message("${ENet_LIBRARIES}\n${sackit_LIBRARY}\n${LUA_LIBRARIES}\n${OPENGL_LIBRARIES}\n${GLEW_LIBRARIES}\n${SDL2_LIBRARIES}")
target_link_libraries(iceball ${ENet_LIBRARIES} ${ZLIB_LIBRARIES} ${sackit_LIBRARY} ${LUA_LIBRARIES} ${SDL2_LIBRARIES} ${OPENGL_LIBRARIES} ${GLEW_LIBRARIES})

set_target_properties(iceball PROPERTIES COMPILE_DEFINITIONS "USE_OPENGL")

# iceball-dedi target
add_executable(iceball-dedi EXCLUDE_FROM_ALL ${MAIN_FILES} ${LUA_FILES})
target_link_libraries(iceball-dedi ${ENet_LIBRARIES} ${ZLIB_LIBRARIES} ${LUA_LIBRARIES} ${SDL_LIBRARY})
set_target_properties(iceball-dedi PROPERTIES COMPILE_DEFINITIONS "DEDI")

