cmake_minimum_required(VERSION 3.10)
project(SpaceRox VERSION 1.0 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(MINGW)
	set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
	set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
	set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
	set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
	add_compile_options(-Wno-narrowing)
	add_link_options(-mwindows)
endif()

list(APPEND CMAKE_PREFIX_PATH "/usr/local")

find_package(SDL2 REQUIRED)
find_package(SDL2_ttf REQUIRED)
find_package(SDL2_mixer REQUIRED)
if(NOT MINGW)
	find_package(Vulkan REQUIRED)
endif()
find_package(libmx2 REQUIRED)

file(GLOB SPACE_ROX_SOURCES CONFIGURE_DEPENDS
	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
)

if(NOT SPACE_ROX_SOURCES)
	message(FATAL_ERROR "No source files found in src/")
endif()

if(MINGW)
	add_executable(SpaceRox WIN32 ${SPACE_ROX_SOURCES})
else()
	add_executable(SpaceRox ${SPACE_ROX_SOURCES})
endif()

if(MINGW)
	target_link_libraries(SpaceRox PRIVATE
		SDL2::SDL2main
		libmx2::mx
		libmx2::mxvk
		SDL2::SDL2
		SDL2_ttf::SDL2_ttf
		SDL2_mixer::SDL2_mixer
	)
else()
	target_link_libraries(SpaceRox PRIVATE
		libmx2::mx
		libmx2::mxvk
		SDL2::SDL2
		SDL2_ttf::SDL2_ttf
		SDL2_mixer::SDL2_mixer
		Vulkan::Vulkan
		${CMAKE_DL_LIBS}
	)
endif()

set_target_properties(SpaceRox PROPERTIES
	RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}"
)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/data")
	add_custom_command(TARGET SpaceRox POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_directory
			"${CMAKE_CURRENT_SOURCE_DIR}/data"
			"$<TARGET_FILE_DIR:SpaceRox>/data"
		COMMENT "Copying data directory to build output"
	)
else()
	message(WARNING "data/ directory not found")
endif()

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/font.ttf")
	add_custom_command(TARGET SpaceRox POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_if_different
			"${CMAKE_CURRENT_SOURCE_DIR}/font.ttf"
			"$<TARGET_FILE_DIR:SpaceRox>/font.ttf"
		COMMENT "Copying font.ttf to build output"
	)
else()
	message(WARNING "font.ttf not found")
endif()

message(STATUS "----------- SpaceRox build summary -----------")
message(STATUS "  C++ standard : C++${CMAKE_CXX_STANDARD}")
message(STATUS "  Compiler     : ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "  SDL2         : found")
message(STATUS "  SDL2_ttf     : found")
message(STATUS "  SDL2_mixer   : found")
if(NOT MINGW)
	message(STATUS "  Vulkan       : found")
endif()
message(STATUS "  libmx2       : found")
message(STATUS "  Build type   : ${CMAKE_BUILD_TYPE}")
message(STATUS "----------------------------------------------")
