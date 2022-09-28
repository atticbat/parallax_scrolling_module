#version, recentmost was 3.24 at start of project 
cmake_minimum_required(VERSION 3.0)
#set vars: name and sources
set(PROJECT_NAME "parallax_scrolling")
set(SRC_FILES src/parallax_scrolling.cpp)
add_library(PSM EXCLUDE_FROM_ALL ${SRC_FILES})
#compiling DevIL for image display
include(ExternalProject)
#DevIL dependencies v---------------v
set(LIBPNG_URL "git://git.code.sf.net/p/libpng/code")
set(LIBPNG_PATH LIBPNG)
if (NOT EXISTS "${LIBPNG_PATH}")
	ExternalProject_Add(${LIBPNG_PATH}
		DOWNLOAD_COMMAND	git clone ${LIBPNG_URL} ${LIBPNG_PATH}
		PREFIX			${LIBPNG_PATH}
	)
endif()
target_include_directories(PSM PRIVATE build/${LIBPNG_PATH})
set(ZLIB_URL "https://git.tukaani.org/xz.git")
set(ZLIB_PATH ZLIB)
if (NOT EXISTS "${ZLIB_PATH}")
	ExternalProject_Add(${ZLIB_PATH}
		DOWNLOAD_COMMAND	git clone ${ZLIB_URL} ${ZLIB_PATH}
		PREFIX			${ZLIB_PATH}
	)
endif()
target_include_directories(PSM PRIVATE build/${ZLIB_PATH})
set(LIBJPEG_URL "https://github.com/csparker247/jpeg-cmake.git")
set(LIBJPEG_PATH LIBJPEG)
if (NOT EXISTS "${LIBJPEG_PATH}")
	ExternalProject_Add(${LIBJPEG_PATH}	#need to extract rather than clone
		DOWNLOAD_COMMAND	git clone ${LIBJPEG_URL} ${LIBJPEG_PATH}
		PREFIX			${LIBJPEG_PATH}
	)
endif()
target_include_directories(PSM PRIVATE build/${LIBJPEG_PATH})
set(LIBTIFF_URL "https://gitlab.com/libtiff/libtiff.git")
set(LIBTIFF_PATH LIBTIFF)
if (NOT EXISTS "${LIBTIFF_PATH}")
	ExternalProject_Add(${LIBTIFF_PATH}
		DOWNLOAD_COMMAND	git clone ${LIBTIFF_URL} ${LIBTIFF_PATH}
		PREFIX			${LIBTIFF_PATH}
	)
endif()
target_include_directories(PSM PRIVATE build/${LIBTIFF_PATH})
#[=[
Little-CMS is a problematic repository to work with because:
[[
1. it doesn't contain a cmake list
2. it's cmake adaptation on github requires something called CheckProfileZOO,
	which, from what I understand, only should apply if it's made in Windows
3. the library files it produces is called liblcms2.dylib rather than
	liblcms.dylib which is what my following dependency is looking for
]]
things I can try to deal with this:
[[
1. set_target_properties(target1 PROPERTIES prop1 value1)
2. find_file (<VAR> name1 path1) to find the file before I pass it to (1)
3. shell script to go over files in library folder and remove the '2' in names
]]
]=]
set(LCMS_URL "https://github.com/mm2/Little-CMS.git")
set(LCMS_PATH LCMS)
if (NOT EXISTS "${LCMS_PATH}")
	ExternalProject_Add(${LCMS_PATH}
		DOWNLOAD_COMMAND	git clone ${LCMS_URL} ${LCMS_PATH}
		PREFIX			${LCMS_PATH}
		BUILD_COMMAND		make
		INSTALL_COMMAND		""
		CONFIGURE_COMMAND	sh ./../${LCMS_PATH}/configure
	)
	ExternalProject_Add_Step(${LCMS_PATH} CHANGE_NAME
		WORKING_DIRECTORY	"${LCMS_PATH}/src/${LCMS_PATH}-build/src/.libs/"
		DEPENDEES		build
		COMMAND			make -C ../../
		COMMAND			mv ../liblcms2.la ../liblcms.la
		COMMAND			mv liblcms2.dylib liblcms.dylib
		COMMAND			mv liblcms2.la liblcms.la
		COMMAND			mv liblcms2.a liblcms.a
		COMMAND			mv liblcms2.lai liblcms.lai
		COMMAND			mv ../../../${LCMS_PATH}/include/lcms2.h ../../../${LCMS_PATH}/include/lcms.h 
	)
endif()
target_include_directories(PSM PRIVATE build/${LCMS_PATH})
set(LIBMNG_URL "https://github.com/LuaDist/libmng.git")
set(LIBMNG_PATH LIBMNG)
if (NOT EXISTS "${LIBMNG_PATH}")
	ExternalProject_Add(${LIBMNG_PATH}
		DOWNLOAD_COMMAND	git clone ${LIBMNG_URL} ${LIBMNG_PATH}
		PREFIX			${LIBMNG_PATH}
	)
endif()
#never got past this point xdd
target_include_directories(PSM PRIVATE build/${LIBMNG_PATH})
#set(MNG_INCLUDE_DIR ${LIBMNG_PATH} CACHE PATH "" FORCE)
#set(LCMS_INCLUDE_DIR ${LITTLECMS_PATH} CACHE PATH "" FORCE)
#DevIL dependencies ^---------------^
target_include_directories(PSM PRIVATE includes)
#set name of executable
project(${PROJECT_NAME})
#both these should already exist
find_package(OpenGL REQUIRED)
find_package(GLUT REQUIRED)
#set(CMAKE_MACOSX_RPATH 1)
set(DEVIL_URL "https://github.com/LuaDist/libdevil.git")
set(DEVIL_PATH DevIL)
if (NOT EXISTS "${DEVIL_PATH}")
	ExternalProject_Add(${DEVIL_PATH}
		DOWNLOAD_COMMAND	git clone ${DEVIL_URL} ${DEVIL_PATH} #will clone to folder named DevIL
		PREFIX			${DEVIL_PATH}	#sets root directory for external project
#		CMAKE_ARGS		-D MNG_INCLUDE_DIR:PATH=${LIBMNG_PATH}, -D LCMS_INCLUDE_DIR:PATH=${LCMS_PATH}, -D CMAKE_MACOSX_RPATH=ON
	)
	ExternalProject_Add_StepDependencies(${DEVIL_PATH}
		build			LCMS_LIB
	)
endif()
target_include_directories(PSM PRIVATE build/${DEVIL_PATH}/src-IL/include)
#add_compile_options(-std=c++14)
add_executable(${PROJECT_NAME} ${SRC_FILES})
#look into what this does exactly
set_property(TARGET ${PROJECT_NAME} APPEND_STRING PROPERTY LINK_FLAGS "-framework OpenGl")
#I assume this should add DevIL to the global env for my program
file(GLOB DEVIL_LIB "build/${DEVIL_PATH}/src/${DEVIL_PATH}-build/lib/libIL.dylib")
target_link_libraries(${PROJECT_NAME} ${DEVIL_LIB})
