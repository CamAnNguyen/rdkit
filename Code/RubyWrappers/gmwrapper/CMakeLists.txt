cmake_minimum_required(VERSION 3.8)

# Apple: Don't modify install_name when touching RPATH.
if(POLICY CMP0068)
  cmake_policy(SET CMP0068 NEW)
endif()
# target_sources: use absolute path for INTERFACE_SOURCES.
if(POLICY CMP0076)
  cmake_policy(SET CMP0076 NEW)
endif()
# SWIG: use standard target name.
if(POLICY CMP0078)
  cmake_policy(SET CMP0078 NEW)
endif()
# SWIG: use SWIG_MODULE_NAME property.
if(POLICY CMP0086)
  cmake_policy(SET CMP0086 NEW)
endif()

project (GraphMolRuby)

include_directories( ${RDKit_ExternalDir} )

find_package(Ruby REQUIRED)

# message("rdkit include: ${RDKIT_INCLUDE_DIR}")
# message("rdkit wrapper libs: ${RDKit_Wrapper_Libs}")
# message("cmake cur source: ${CMAKE_CURRENT_SOURCE_DIR}")
# message("BOOST ROOT: ${BOOST_ROOT}")

INCLUDE_DIRECTORIES(${RDKit_INCLUDE_DIR} ${RUBY_INCLUDE_PATH})

SET_SOURCE_FILES_PROPERTIES(GraphMolRuby.i PROPERTIES CPLUSPLUS ON )

#FILE(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../*.java COPY_SOURCE)
#FILE(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/src/org/RDKit COPY_DEST)

# Setup a few variables for environment-specific things
# if(WIN32)
#   ADD_DEFINITIONS("/W3 /wd4716 /bigobj")
#   SET(PATH_SEP ";")
#   SET(COPY_CMD xcopy ${COPY_SOURCE} ${COPY_DEST} /Y /I)
# else()
#   SET(PATH_SEP ":")
#   SET(COPY_CMD cp -p ${COPY_SOURCE} ${COPY_DEST})
# endif()

# Coax SWIG into playing nicely with Apple environments
# if(APPLE)
#   SET(CMAKE_SIZEOF_VOID_P 4)
# endif(APPLE)

# if(CMAKE_SIZEOF_VOID_P MATCHES 4)
#   SET(CMAKE_SWIG_FLAGS -package "GraphMolRubyWrap")
# else()
#   if (WIN32)
#     SET(CMAKE_SWIG_FLAGS -package "GraphMolRubyWrap" "-DSWIGWORDSIZE64" "-DSWIGWIN")
#   else()
#       SET(CMAKE_SWIG_FLAGS -package "GraphMolRubyWrap" "-DSWIGWORDSIZE64")
#   endif()
# endif()

# SET(CMAKE_SWIG_FLAGS -package "GraphMolRubyWrap" "-DSWIGWORDSIZE64")
SET(CMAKE_SWIG_OUTDIR ${CMAKE_CURRENT_SOURCE_DIR}/swig_ruby )

if(RDK_BUILD_INCHI_SUPPORT)
  SET(CMAKE_SWIG_FLAGS "-DRDK_BUILD_INCHI_SUPPORT" ${CMAKE_SWIG_FLAGS} )
endif()
if(RDK_BUILD_AVALON_SUPPORT)
  SET(CMAKE_SWIG_FLAGS "-DRDK_BUILD_AVALON_SUPPORT" ${CMAKE_SWIG_FLAGS} )
endif()
if(RDK_USE_BOOST_IOSTREAMS)
  SET(CMAKE_SWIG_FLAGS "-DRDK_USE_BOOST_IOSTREAMS" ${CMAKE_SWIG_FLAGS} )
endif()

# enable this line to build the ErrorGenerator class for testing handling of C++ errors in the JNI layer
# SET(CMAKE_SWIG_FLAGS ${CMAKE_SWIG_FLAGS} "-D INCLUDE_ERROR_GENERATOR" )

FILE(GLOB SWIG_SRC_FILES "${CMAKE_CURRENT_SOURCE_DIR}/../*.i")

# we added all source files, now remove the ones that we're not supporting in this build:
if(NOT RDK_BUILD_AVALON_SUPPORT)
LIST(REMOVE_ITEM SWIG_SRC_FILES "${CMAKE_CURRENT_SOURCE_DIR}/../AvalonLib.i")
endif()

if(NOT RDK_BUILD_INCHI_SUPPORT)
LIST(REMOVE_ITEM SWIG_SRC_FILES "${CMAKE_CURRENT_SOURCE_DIR}/../Inchi.i")
endif()

SET(SWIG_MODULE_RDKit_EXTRA_DEPS ${SWIG_SRC_FILES} )
SET(CMAKE_SWIG_FLAGS -small -naturalvar -autorename)

SWIG_ADD_LIBRARY(RDKit TYPE MODULE LANGUAGE ruby SOURCES GraphMolRuby.i)

# it doesnt seem like the threading libs should need to be here, but
# as of Oct 2012 using boost 1.51 under at least ubuntu 12.04 we get a
# link error if they aren't there.
SWIG_LINK_LIBRARIES(RDKit
  ${RDKit_Wrapper_Libs}
  ${Boost_SERIALIZATION_LIBRARY}
  ${RDKit_THREAD_LIBS})

MESSAGE("serial: ${Boost_SERIALIZATION_LIBRARY}")
MESSAGE("THREAD: ${RDKit_THREAD_LIBS}")

# ADD_CUSTOM_COMMAND (
#    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/libRDKit_Ruby.so
#    COMMAND ${CMAKE_COMMAND} -E echo_append "Creating wrapper code for ruby..."
#    COMMAND ${SWIG_EXECUTABLE} -c++ -ruby -autorename -o ${SWIG_OUTPUT} -I${LIBYUI_INCLUDE_DIR} ${SWIG_INPUT}
#    COMMAND ${CMAKE_COMMAND} -E echo "Done."
#    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
#    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/../*.i
# )

INSTALL(TARGETS RDKit LIBRARY DESTINATION ${CMAKE_CURRENT_SOURCE_DIR})
