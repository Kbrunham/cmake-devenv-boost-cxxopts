cmake_minimum_required(VERSION 3.12)

project(devenv-superbuild C CXX)

include(ExternalProject)

option(PERFORM_FULL_BUILD "Build environment and source (ON) or environment only (OFF)" OFF)

# Add boost
set(BOOST_VERSION "1.82.0")
if (WIN32)
	set(BOOST_BOOTSTRAP_CMD "bootstrap.bat")
	set(BOOST_B2_CMD "b2.exe")
	set(BOOST_INCDIR_PATH "${CMAKE_CURRENT_SOURCE_DIR}/extlibs/boost/include/boost-1_82")
else()
	set(BOOST_BOOTSTRAP_CMD "./bootstrap.sh")
	set(BOOST_B2_CMD "./b2")
	set(BOOST_INCDIR_PATH "${CMAKE_CURRENT_SOURCE_DIR}/extlibs/boost/include")
endif()
ExternalProject_Add(boost
	PREFIX ${CMAKE_CURRENT_BINARY_DIR}/boost
	# URL https://dl.bintray.com/boostorg/release/1.75.0/source/boost_1_75_0.tar.bz2
	# URL_HASH SHA256=953db31e016db7bb207f11432bef7df100516eeb746843fa0486a222e3fd49cb
	GIT_REPOSITORY https://github.com/boostorg/boost.git
	GIT_TAG boost-${BOOST_VERSION}
	GIT_PROGRESS 1
	BUILD_IN_SOURCE 1
	CONFIGURE_COMMAND ${BOOST_BOOTSTRAP_CMD}
	BUILD_COMMAND ${BOOST_B2_CMD} 
		install 
		--with-filesystem
		--with-regex
		--with-serialization
		--build-type=minimal
		--link=static
		--prefix=${CMAKE_CURRENT_SOURCE_DIR}/extlibs/boost
	INSTALL_COMMAND ""
	LOG_DOWNLOAD 1
	LOG_UPDATE 1
	LOG_CONFIGURE 1
	LOG_BUILD 1
	LOG_INSTALL 1
	INSTALL_DIR ${CMAKE_CURRENT_SOURCE_DIR}/extlibs/boost
)

# Add OpenSSL
set(OPENSSL_VERSION "OpenSSL_1_1_1u")
if (WIN32)
	set(OPENSSL_CONFIGURE_CMD_PLATFORM "VC-WIN64A")
	set(OPENSSL_BUILD_CMD "nmake")
else()
	set(OPENSSL_CONFIGURE_CMD_PLATFORM "linux-x86_64")
	set(OPENSSL_BUILD_CMD "make")
endif()
ExternalProject_Add(openssl
	PREFIX ${CMAKE_CURRENT_BINARY_DIR}/openssl
	#URL https://github.com/openssl/openssl/archive/OpenSSL_1_1_1u.tar.gz
	GIT_REPOSITORY https://github.com/openssl/openssl.git 
	GIT_TAG ${OPENSSL_VERSION}
	GIT_PROGRESS 1	
	BUILD_IN_SOURCE 1
	CONFIGURE_COMMAND perl
		Configure
		${OPENSSL_CONFIGURE_CMD}
		--prefix=${CMAKE_CURRENT_SOURCE_DIR}/extlibs/openssl
		no-shared
		--openssldir=${CMAKE_CURRENT_SOURCE_DIR}/extlibs/openssl/local
		${OPENSSL_CONFIGURE_CMD_PLATFORM}
	BUILD_COMMAND ${OPENSSL_BUILD_CMD}
	INSTALL_COMMAND ${OPENSSL_BUILD_CMD}
		install
	LOG_DOWNLOAD 1
	LOG_UPDATE 1
	LOG_CONFIGURE 1
	LOG_BUILD 1
	LOG_INSTALL 1
)


# Give the option to perform a full build. This then allows for specifying the dependencies
# when compiling the source project stand alone.
if (PERFORM_FULL_BUILD)
	ExternalProject_Add(src_project
		PREFIX src_project
		SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src"
		CMAKE_CACHE_ARGS
			-DBoost_NO_BOOST_CMAKE:BOOL=ON
			-DBoost_NO_SYSTEM_PATHS:BOOL=ON
			-DBOOST_ROOT:PATH=${CMAKE_CURRENT_SOURCE_DIR}/extlibs/boost
			-DBOOST_INCLUDEDIR:PATH=${BOOST_INCDIR_PATH}
			-DOPENSSL_ROOT_DIR:PATH=${CMAKE_CURRENT_SOURCE_DIR}/extlibs/openssl
			DEPENDS boost openssl
		INSTALL_COMMAND ""
		TEST_BEFORE_INSTALL 1
	)
endif()
