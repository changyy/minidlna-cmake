cmake_minimum_required(VERSION 2.8)
project (minidlna)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set(SELF_CPPFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

add_subdirectory(sqlite-cmake)
set(SELF_CPPFLAGS "${SELF_CPPFLAGS};-I${CMAKE_CURRENT_SOURCE_DIR}/sqlite-cmake/sqlite-autoconf-3071502")
set(SELF_DEPENDS "${SELF_DEPENDS};sqlite3-shared")

if(NOT TARGET build_libexif)
add_subdirectory(libexif-cmake)
endif(NOT TARGET build_libexif)
set(SELF_CPPFLAGS "${SELF_CPPFLAGS};-I${CMAKE_CURRENT_BINARY_DIR}/libexif")
set(SELF_DEPENDS "${SELF_DEPENDS};build_libexif")

if(NOT TARGET build_ffmpeg)
add_subdirectory(ffmpeg-cmake)
endif(NOT TARGET build_ffmpeg)
#set(SELF_CPPFLAGS "${SELF_CPPFLAGS};-I${CMAKE_CURRENT_BINARY_DIR}/ffmpeg")
set(SELF_CPPFLAGS "${SELF_CPPFLAGS};-I${CMAKE_CURRENT_BINARY_DIR}/ffmpeg/ffmpeg-1.2.1")
set(SELF_DEPENDS "${SELF_DEPENDS};build_ffmpeg")

if(NOT TARGET build_libjpeg)
add_subdirectory(libjpeg-cmake)
endif(NOT TARGET build_libjpeg)
set(SELF_DEPENDS "${SELF_DEPENDS};build_libjpeg")

if(NOT TARGET build_libid3tag)
add_subdirectory(libid3tag-cmake)
endif(NOT TARGET build_libid3tag)
set(SELF_DEPENDS "${SELF_DEPENDS};build_libid3tag")

if(NOT TARGET build_libogg)
add_subdirectory(libogg-cmake)
endif(NOT TARGET build_libogg)
set(SELF_DEPENDS "${SELF_DEPENDS};build_libogg")

if(NOT TARGET build_libvorbis)
add_subdirectory(libvorbis-cmake)
endif(NOT TARGET build_libvorbis)
set(SELF_DEPENDS "${SELF_DEPENDS};build_libvorbis")

if(NOT TARGET build_libflac)
add_subdirectory(libFLAC-cmake)
endif(NOT TARGET build_libflac)
set(SELF_DEPENDS "${SELF_DEPENDS};build_libflac")

set(SELF_CPPFLAGS "${SELF_CPPFLAGS};-I${CMAKE_BINARY_DIR}/include")
set(SELF_LDFLAGS "${SELF_LDFLAGS};-L${CMAKE_BINARY_DIR}/lib;-lz;-lsqlite3;-ldl;-lpthread;-lavformat;-lavcodec;-lavutil")

set(TOOL_TARGET_NAME "build_minidlna")
set(TOOL_TARGET_BUILD_OUTPUT "BUILD_MINIDLNA")
set(TOOL_TARGET_COMMENT "build service minidlna")
set(TOOL_TARGET_SOURCE minidlna-1.1.0.tar.gz)
set(TOOL_TARGET_DIR minidlna-1.1.0)
set(CONFIG_ADD_OPTIONS "")
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget http://downloads.sourceforge.net/project/minidlna/minidlna/1.1.0/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} && cd ${CMAKE_BINARY_DIR}/${TOOL_TARGET_DIR} && ${target_platform_preconfigure_options} ./configure --host="${target_platform}" --prefix=${CMAKE_BINARY_DIR} LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" ${CONFIG_ADD_OPTIONS} && make -j2 && make install
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget http://downloads.sourceforge.net/project/minidlna/minidlna/1.1.0/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} && cd ${CMAKE_BINARY_DIR}/${TOOL_TARGET_DIR} && ./configure --prefix=${CMAKE_BINARY_DIR} LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" ${CONFIG_ADD_OPTIONS} && make -j2 && make install 
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(${TOOL_TARGET_NAME} ALL DEPENDS ${SELF_DEPENDS} ${TOOL_TARGET_BUILD_OUTPUT} )
