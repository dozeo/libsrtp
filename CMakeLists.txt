cmake_minimum_required(VERSION 2.8)

set(libname libsrtp)

project(${libname})

option(DEBUG "Enable debugging" OFF)
option(ERROR_TO_FILE "Report errors to file instead of stderr" OFF)
option(ENABLE_GENERIC_AESICM "Enable generic AES ICM" OFF)
option(BUILD_FOR_RISK "Build for RISK cpu type" OFF)
option(SYSLOG_LOGGING "Enable syslog logging" OFF)
option(WITH_OPENSSL "Enable building with openssl" ON)
option(BUILD_SHARED_LIB "Enable building of shared library instead of the static" OFF)

# -- start of checks

include(CheckIncludeFiles)
include(CheckFunctionExists)
include(CheckLibraryExists)
include(CheckTypeSize)
include(TestBigEndian)

TEST_BIG_ENDIAN(BIG_ENDIAN)

if (${ENABLE_GENERIC_AESICM})
	set(GENERIC_AESICM 1)
endif()

if (${DEBUG})
	set(ENABLE_DEBUGGING 1)
endif()

# TODO: determine if this is a X86 architecture
set(IS_X86 ON)
if (${IS_X86})
	set(HAVE_X86 1)
endif()

# errors to log file
if (${ERROR_TO_FILE})
	set(ERR_REPORTING_FILE "libsrtp_error.log")
	set(USE_ERR_REPORTING_FILE 1)
else()
	set(ERR_REPORTING_STDOUT 1)
endif()

if (${BUILD_FOR_RISK})
	set(CPU_RISC 1)
else()
	set(CPU_CISC 1)
endif()

if (${BIG_ENDIAN} EQUAL 0)
	set(WORDS_BIGENDIAN 0)
else()
	set(WORDS_BIGENDIAN 1)
endif()

if(EXISTS /dev/urandom)
	set(DEV_URANDOM "/dev/urandom")
endif()

check_include_files(arpa/inet.h HAVE_ARPA_INET_H)
check_include_files(byteswap.h HAVE_BYTESWAP_H)
check_function_exists(inet_aton HAVE_INET_ATON)
check_type_size(int16_t HAVE_INT16_T)
check_type_size(int32_t HAVE_INT32_T)
check_type_size(int8_t HAVE_INT8_T)
check_include_files(inttypes.h HAVE_INTTYPES_H)
check_library_exists(socket socket "" HAVE_LIBSOCKET)
check_include_files(machine/types.h HAVE_MACHINE_TYPES_H)
check_include_files(memory.h HAVE_MEMORY_H)
check_include_files(netinet/in.h HAVE_NETINET_IN_H)
check_function_exists(socket HAVE_SOCKET)
check_include_files(stdint.h HAVE_STDINT_H)
check_include_files(stdlib.h HAVE_STDLIB_H)
check_include_files(strings.h HAVE_STRINGS_H)
check_include_files(string.h HAVE_STRING_H)
check_include_files(syslog.h HAVE_SYSLOG_H)
check_include_files(sys/int_types.h HAVE_SYS_INT_TYPES_H)
check_include_files(sys/socket.h HAVE_SYS_SOCKET_H)
check_include_files(sys/stat.h HAVE_SYS_STAT_H)
check_include_files(sys/types.h HAVE_SYS_TYPES_H)
check_include_files(sys/uio.h HAVE_SYS_UIO_H)
check_type_size(uint16_t HAVE_UINT16_T)
check_type_size(uint32_t HAVE_UINT32_T)
check_type_size(uint64_t HAVE_UINT64_T)
check_type_size(uint8_t HAVE_UINT8_T)
check_include_files(unistd.h HAVE_UNISTD_H)
check_function_exists(usleep HAVE_USLEEP)
check_include_files(windows.h HAVE_WINDOWS_H)
check_include_files(winsock2.h HAVE_WINSOCK2_H)

check_type_size("unsigned long" SIZEOF_UNSIGNED_LONG)
check_type_size("unsigned long long" SIZEOF_UNSIGNED_LONG_LONG)

check_type_size(size_t SIZE_T_EXISTS)

if("${SIZE_T_EXISTS}" STREQUAL "")
	message(STATUS "SIZE_T missing")
	set(size_t 1)
	set(size_t_type "unsigned")
endif()

# TODO: determine this with cmake
#set(NON_CONST_ANSI_CONFORM TRUE)
if(${NON_CONST_ANSI_CONFORM})
	set(const 1)
	set(CONST_WHAT "")
endif()

# check if openssl exists
if(${WITH_OPENSSL})
	find_package(OpenSSL)
	if(NOT ${OPENSSL_FOUND})
		message(STATUS "OpenSSL was not found.. will continue WITHOUT_OPENSSL")
	endif()
endif()

if(MSVC)
	set(inline 1)
	set(INLINE_KEYWORD "_inline")
endif()

if(NOT "${HAVE_SYSLOG_H}" STREQUAL "")
	if (${SYSLOG_LOGGING})
		set(USE_SYSLOG 1)
	endif()
 	message(STATUS "Has syslog!")
endif()

file(STRINGS "VERSION" VERSION)

set(PACKAGE_NAME "libsrtp")
set(PACKAGE_VERSION "${VERSION}")
set(PACKAGE_STRING "${PACKAGE_NAME}_${VERSION}")

configure_file(config.h.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/include/srtp/config.h")

# ---- End of checks
if (BUILD_SHARED_LIB)
	set(libtype SHARED)
	if(WIN32)
		set(pclibs "-lsrtp.dll")
	endif()
#	list(APPEND definitions "EXPORT_DLL")
else()
	set(libtype STATIC)
	set(pclibs "-lsrtp")
endif()
#set(libtype SHARED)

set(src
	"srtp/srtp.c"
	"srtp/ekt.c"
	"crypto/kernel/alloc.c"
	"crypto/hash/auth.c"
	"crypto/kernel/err.c"
	"crypto/math/datatypes.c"
	"crypto/kernel/crypto_kernel.c"
	"crypto/cipher/cipher.c"
	"crypto/kernel/key.c"
	"crypto/replay/rdbx.c"
	"crypto/replay/rdb.c"
	"crypto/math/stat.c"
	"crypto/rng/ctr_prng.c"
	"crypto/cipher/aes_cbc.c"
	"crypto/cipher/null_cipher.c"
	"crypto/hash/null_auth.c"
	"crypto/cipher/aes.c"
	)

set(config_header
	"${CMAKE_CURRENT_BINARY_DIR}/include/srtp/config.h")

set(hdrs
	"include/srtp/crypto/alloc.h"
	"include/srtp/ekt.h"
	"include/srtp/getopt_s.h"
	"include/srtp/inttypes.h"
	"include/srtp/rtp_priv.h"
	"include/srtp/srtp_priv.h"
	"include/srtp/stdint.h"
	"include/srtp/crypto/crypto_kernel.h"
	"include/srtp/crypto/auth.h"
	"include/srtp/crypto/err.h"
	"include/srtp/crypto/datatypes.h"
	"include/srtp/crypto/aes.h"
	"include/srtp/crypto/integers.h"
	"include/srtp/crypto/aes_cbc.h"
	"include/srtp/crypto/aes_icm.h"
	"${config_header}"
	)

if(OPENSSL_FOUND)
	list(APPEND src "crypto/cipher/aes_gcm_ossl.c")
	list(APPEND src "crypto/cipher/aes_icm_ossl.c")
	list(APPEND src "crypto/hash/hmac_ossl.c")
	list(APPEND src "crypto/rng/rand_source_ossl.c")

	list(APPEND hdrs "include/srtp/crypto/aes_gcm_ossl.h")
	list(APPEND hdrs "include/srtp/crypto/aes_icm_ossl.h")

	list(APPEND include_dirs ${OPENSSL_INCLUDE_DIR})
	list(APPEND otherlibs ${OPENSSL_LIBRARIES})
	list(APPEND definitions "OPENSSL")

	# TODO: Add openssl to pkgconfig output
	#set(pclibs "{pclibs} ${OPENSSL_LIBRARIES}")
else()
	# without openssl
	list(APPEND src "crypto/cipher/aes_icm.c")
	list(APPEND src "crypto/hash/sha1.c")
	list(APPEND src "crypto/hash/hmac.c")

	# determine os random source
	if(LINUX)
		list(APPEND src "crypto/rng/rand_linux_kernel.c")
	else()
		list(APPEND src "crypto/rng/rand_source.c")
	endif()
endif()

list(APPEND hdrs
	"include/srtp/srtp.h")

list(APPEND include_dirs "${CMAKE_CURRENT_BINARY_DIR}/include")
list(APPEND include_dirs "${CMAKE_CURRENT_SOURCE_DIR}/include")

if(WIN32)
	list(APPEND platformlibs "wsock32.lib" "ws2_32.lib")
	if (NOT BUILD_SHARED_LIB)
		set(pclibs "${pclibs} -lwsock32")
	endif()
endif()

list(APPEND libraries "${platformlibs}" "${otherlibs}")
list(APPEND definitions "SOME_DEFINES")

add_library(${libname} ${libtype} ${src} ${hdrs})

if(MSVC)
	set_target_properties(${libname} PROPERTIES
		COMPILE_FLAGS "/wd4018 /wd4244")
else()
	configure_file(libsrtp.pc.in ${CMAKE_BINARY_DIR}/libsrtp.pc
		@ONLY)
	install(FILES "${CMAKE_BINARY_DIR}/libsrtp.pc"
		DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/pkgconfig")
endif()

set_target_properties(${libname} PROPERTIES 
	PUBLIC_HEADER "${config_header}"
	INCLUDE_DIRECTORIES "${include_dirs}"
	INTERFACE_INCLUDE_DIRECTORIES "${include_dirs}"
	COMPILE_DEFINITIONS "${definitions}"
	PREFIX ""
	IMPORT_PREFIX ""
)

install(DIRECTORY include/ DESTINATION include
	FILES_MATCHING PATTERN "*.h")

target_link_libraries(${libname} ${libraries})

install(TARGETS ${libname}
	# EXPORT libname
	PUBLIC_HEADER DESTINATION include/srtp COMPONENT dev
	LIBRARY DESTINATION lib COMPONENT lib
	ARCHIVE DESTINATION lib COMPONENT lib
	RUNTIME DESTINATION bin COMPONENT lib)
