cmake_minimum_required (VERSION 3.5 FATAL_ERROR)

# Set compilers for MinGW / *Nix if requested
if ((MINGW OR UNIX) AND MIRAGE_COMPILER)
	MESSAGE (STATUS "C/C++ Compilers: MinGW or *Nix detected, setting to requested ${MIRAGE_COMPILER}.")

	set (CMAKE_C_COMPILER ${MIRAGE_COMPILER})
	set (CMAKE_CXX_COMPILER ${MIRAGE_COMPILER})
elseif (MINGW OR UNIX)
	MESSAGE (STATUS "C/C++ Compilers: MinGW or *Nix detected, defaulting to gcc.")

	set (CMAKE_C_COMPILER "gcc")
	set (CMAKE_CXX_COMPILER "gcc")
endif ()

# Detect mingw on windows and linux cross-compile
if (MINGW)
	set (MIRAGE_COMPILER "mingw")
endif ()

# Detect architecture
if (MIRAGE_M64)
	set (ARCH "x64")
else ()
	set (ARCH "x86")
endif ()

# Project setup
project (MirageRender CXX)
set (VERSION_R 0)
set (VERSION_B 1)
set (VERSION_A 1)

# Set project bin, dll and res directories
if (MIRAGE_BIN)
	set (BIN_DIR "${PROJECT_SOURCE_DIR}/${MIRAGE_BIN}")
else ()
	set (BIN_DIR "${PROJECT_SOURCE_DIR}/3rdparty/bin_${MIRAGE_COMPILER}_${ARCH}")
endif ()
set (DLL_DIR "${PROJECT_SOURCE_DIR}/3rdparty/bin/${MIRAGE_COMPILER}/${ARCH}")
set (RES_DIR "${PROJECT_SOURCE_DIR}/res")

# Set project lib directory
set (LIB_DIR "${PROJECT_SOURCE_DIR}/3rdparty/lib/${MIRAGE_COMPILER}/${ARCH}")

MESSAGE (STATUS "Resolved library dir: ${LIB_DIR}")

# Sources, headers + resources
file (GLOB_RECURSE SOURCES "${PROJECT_SOURCE_DIR}/src/*.cpp" "${PROJECT_SOURCE_DIR}/src/*.h")
file (GLOB_RECURSE RESOURCES "${PROJECT_SOURCE_DIR}/res/*")

# SDL2 and Lua 5.3.x
if (MSVC)
	# Include directories
	set (SDL2_INCLUDE_DIR ${SDL2_INCLUDE_DIR} "${PROJECT_SOURCE_DIR}/3rdparty/include")
	set (LUA_INCLUDE_DIR ${LUA_INCLUDE_DIR} "${PROJECT_SOURCE_DIR}/3rdparty/include")

	# Library directories
	set (SDL2_LIBRARY ${SDL2_LIBRARY} "${LIB_DIR}/SDL2main.lib" "${LIB_DIR}/SDL2.lib")
	set (LUA_LIBRARIES ${LUA_LIBRARIES} "${LIB_DIR}/lua5.3.lib")
else ()
	# Include directories
	if (MINGW)
		set (SDL2_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/3rdparty/include")
		set (SDL2_PATH "${PROJECT_SOURCE_DIR}")
		set (ENV{SDL2DIR} "${PROJECT_SOURCE_DIR}")
		set (LUA_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/3rdparty/include")
	endif ()

	# Library directories
	if (MINGW)
		set (SDL2_LIBRARY "${LIB_DIR}/libSDL2main.a" "${LIB_DIR}/libSDL2.dll.a")
		set (LUA_LIBRARIES "${LIB_DIR}/liblua.a")
	endif ()
endif ()

MESSAGE (STATUS "Resolved SDL2 Include dir(s): ${SDL2_INCLUDE_DIR}")
MESSAGE (STATUS "Resolved SDL2 Library dir(s): ${SDL2_LIBRARY}")
MESSAGE (STATUS "Resolved Lua Include dir(s): ${LUA_INCLUDE_DIR}")
MESSAGE (STATUS "Resolved Lua Library dir(s): ${LUA_LIBRARIES}")

# CMake modules directory
set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/CMakeModules")

# Add to be linked executables to project
add_executable (MirageRender ${SOURCES})

# Group source files properly for VS
source_group ("src" FILES ${SOURCES})

# Compiler flags for each generator
if (MSVC)
	MESSAGE (STATUS "Compiler flags: MSVC Detected.")

	target_compile_options (MirageRender PUBLIC
		-std=c++11
	)

	if (CMAKE_BUILD_TYPE STREQUAL "Release")
		MESSAGE (STATUS "Compiler flags: MSVC Release detected.")

		target_compile_options (MirageRender PUBLIC
			/O2
		)
	else ()
		MESSAGE (STATUS "Compiler flags: MSVC Debug detected.")
	endif ()
else ()
	MESSAGE (STATUS "Compiler flags: MinGW or *Nix detected.")

	target_compile_options (MirageRender PUBLIC
		-std=c++11
		-Wall
		-Wextra
		-Wno-unused-variable
		-Wno-unused-parameter
		-Wno-unused-function
	)

	if (MIRAGE_M64)
		MESSAGE (STATUS "Compiler flags: MinGW or *Nix 64bit detected.")

		target_compile_options (MirageRender PUBLIC
			-m64
		)
	else ()
		MESSAGE (STATUS "Compiler flags: MinGW or *Nix 32bit detected.")

		target_compile_options (MirageRender PUBLIC
			-m32
		)
	endif ()

	if (CMAKE_BUILD_TYPE STREQUAL "Release")
		MESSAGE (STATUS "Compiler flags: MinGW or *Nix Release detected.")

		target_compile_options (MirageRender PUBLIC
			-Ofast
		)
	else ()
		MESSAGE (STATUS "Compiler flags: MinGW or *Nix Debug detected.")
	endif ()
endif ()

# Linker flags for each generator
if (MINGW)
	MESSAGE (STATUS "Linker flags: MinGW Detected.")

	target_link_libraries (MirageRender PUBLIC
		-lmingw32
		-mconsole
		-static
	)
endif ()

if (MINGW OR UNIX)
	MESSAGE (STATUS "Linker flags: MinGW or *Nix Detected.")

	target_link_libraries (MirageRender PUBLIC
		-lstdc++
		-lm
		-static-libgcc
		-static-libstdc++
	)

	if (CMAKE_BUILD_TYPE STREQUAL "Release")
		MESSAGE (STATUS "Linker flags: MinGW or *Nix Release detected.")
	else ()
		MESSAGE (STATUS "Linker flags: MinGW or *Nix Debug detected.")
	endif()
endif ()

# Include project headers
target_include_directories (MirageRender PUBLIC "${PROJECT_SOURCE_DIR}/src/")

# Dependencies
find_package (SDL2 REQUIRED)
find_package (Lua 5.3 REQUIRED)

target_include_directories (MirageRender PUBLIC ${SDL2_INCLUDE_DIR})
target_include_directories (MirageRender PUBLIC ${LUA_INCLUDE_DIR})

target_link_libraries (MirageRender PUBLIC
	${SDL2_LIBRARY}
	${LUA_LIBRARIES}
)

# Install project
install (TARGETS MirageRender DESTINATION ${BIN_DIR})
install (DIRECTORY ${RES_DIR} DESTINATION ${BIN_DIR})
install (DIRECTORY ${DLL_DIR}/ DESTINATION ${BIN_DIR})

# Configure package
set (CPACK_PACKAGE_NAME "MirageRender")
set (CPACK_PACKAGE_VERSION ${VERSION_R}.${VERSION_B}.${VERSION_A})
set (CPACK_MONOLITHIC_INSTALL 1)
include (CPack)