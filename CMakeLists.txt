project(hge_unix CXX)
cmake_minimum_required(VERSION 2.8)
set(HGE_VERSION 1.8.1)

option(BUILD_TUTORIALS "Build the tutorial samples" OFF)
option(BUILD_TOOLS "Build the library tools" OFF)
option(USE_INTERNAL_ZLIB "Use the internal ZLib" OFF)
option(BUILD_SINGLE_LIB "Build a single library for HGE and Helpers" ON)
option(BUILD_C_API "Build a C api version of the library" OFF)
option(BUILD_C_HELPERS_API "Build a C api version of the helpers library" OFF)
option(BUILD_C_TUTORIALS "Build the tutorials for the C api" OFF)

if(UNIX OR APPLE)
	# Needed for FindVorbis
	set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

	# Add our option  for using CxImage,  default is on
	option(USE_CXIMAGE "Use CXImage (no)" ON)

	find_package(SDL2 REQUIRED)
	find_package(OpenAL REQUIRED)
	find_package(Vorbis REQUIRED)
	find_package(JPEG REQUIRED)
	find_package(PNG REQUIRED)

	# If we're using the external library, we must find it
	if(NOT USE_INTERNAL_ZLIB)
		find_package(ZLIB REQUIRED)
	endif(NOT USE_INTERNAL_ZLIB)

	if(USE_CXIMAGE)
		# CxImage required defines
		add_definitions(-DSUPPORT_CXIMAGE)
	else(USE_CXIMAGE)
		# If we're not using CxImage, we'll use this, more common external library
		find_package(DevIL REQUIRED)
		# Unfortunately though, it's not implemented/working ;)
		message(FATAL_ERROR "DevIL is not ready to be used yet")
	endif(USE_CXIMAGE)

	include_directories(
	    ${SDL_INCLUDE_DIR}		
		${VORBIS_INCLUDE_DIR}
		${IL_INCLUDE_DIR}
		${ZLIB_INCLUDE_DIR}
		${JPEG_INCLUDE_DIR}
		${PNG_INCLUDE_DIR}
		${CMAKE_SOURCE_DIR}/include
		${CMAKE_SOURCE_DIR}/core
        ${OPENAL_INCLUDE_DIR}
	)

	add_definitions(-D_LINUX)

	# The libraries to link against for HGE, and also for applications using it
	set(HGE_LIBRARIES
		${SDL_LIBRARY}
		${OPENAL_LIBRARY}
		${VORBISFILE_LIBRARY}
		${IL_LIBRARIES}
		${ILU_LIBRARIES}
		${ZLIB_LIBRARY}
		${JPEG_LIBRARY}
		${PNG_LIBRARY}
	)

	set(USE_SHARED SHARED)

	# Set the pkg-config requirements
	set(HGE_REQUIRES sdl2)
elseif(WIN32)
	# Windows
	add_definitions(-D_WINDOWS -D_WIN32)
	set(HGE_CFLAGS "-D_WINDOWS -D_WIN32 -mwindows")
	set(USE_INTERNAL_ZLIB ON)

	find_package(PNG REQUIRED)

	set(HGE_LIBRARIES
		d3d8
		winmm
		${PNG_LIBRARY}
	)

	if(MSVC AND NOT (CMAKE_BUILD_TYPE MATCHES Debug))
		set(HGE_LIBRARIES ${HGE_LIBRARIES}
			d3dx8
		)
	else()
		#Sorry MinGW32, you're not currently allowed to make a "release"
		add_definitions(-DDEBUG)
		set(HGE_LIBRARIES ${HGE_LIBRARIES}
			d3dx8d
		)
	endif()

	set(USE_SHARED SHARED)
else()
	message(FATAL_ERROR "Unknown platform")
endif(UNIX OR APPLE)

if(WIN32 OR ${BUILD_SINGLE_LIB})
	set(HGE_LIBRARY hge ${HGE_LIBRARIES})
	set(HGE_HELPER_LIBRARY hge ${HGE_LIBRARIES})
else()
	# This is the HGE library and required libraries to link against
	set(HGE_LIBRARY hge ${HGE_LIBRARIES})
	# And this is the HGE & Helper library and required libraries to link against
	set(HGE_HELPER_LIBRARY hge_helpers ${HGE_LIBRARY})
endif()

configure_file(hge-unix.pc.in hge-unix.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/hge-unix.pc DESTINATION lib/pkgconfig)

# Where HGE is located, this also builds the helper library
add_subdirectory(src/core)

if(BUILD_C_API)
	configure_file(hge-unix-c.pc.in hge-unix-c.pc @ONLY)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/hge-unix-c.pc DESTINATION lib/pkgconfig)
	add_subdirectory(c_api)
endif(BUILD_C_API)

# Did tutorials get enabled to be built?
if(BUILD_TUTORIALS)
	add_subdirectory(tutorials)
endif(BUILD_TUTORIALS)

# Are we building the tools?
if(BUILD_TOOLS)
	add_subdirectory(tools)
endif(BUILD_TOOLS)
