# Adapted from https://gist.github.com/jaynus/6b03724ba94500d267edb002cdfc72ee
# Thanks!

cmake_minimum_required( VERSION 3.0 )

project( nanomsgcxx )

SET(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/nnxx/)

set(NANOMSGXX_SOURCES
  ${SOURCE_DIR}/bus.h
  ${SOURCE_DIR}/chrono.h
  ${SOURCE_DIR}/error.cpp
  ${SOURCE_DIR}/error.h
  ${SOURCE_DIR}/inproc.h
  ${SOURCE_DIR}/ipc.h
  ${SOURCE_DIR}/message
  ${SOURCE_DIR}/message_control.cpp
  ${SOURCE_DIR}/message_control.h
  ${SOURCE_DIR}/message.cpp
  ${SOURCE_DIR}/message.h
  ${SOURCE_DIR}/message_istream.cpp
  ${SOURCE_DIR}/message_istream.h
  ${SOURCE_DIR}/message_istream.hpp
  ${SOURCE_DIR}/message_iterator.cpp
  ${SOURCE_DIR}/message_iterator.h
  ${SOURCE_DIR}/message_ostream.cpp
  ${SOURCE_DIR}/message_ostream.h
  ${SOURCE_DIR}/message_ostream.hpp
  ${SOURCE_DIR}/message_streambuf.cpp
  ${SOURCE_DIR}/message_streambuf.h
  ${SOURCE_DIR}/message_streambuf.hpp
  ${SOURCE_DIR}/nn.cpp
  ${SOURCE_DIR}/nn.h
  ${SOURCE_DIR}/pair.h
  ${SOURCE_DIR}/pipeline.h
  ${SOURCE_DIR}/poll.cpp
  ${SOURCE_DIR}/poll.h
  ${SOURCE_DIR}/pubsub.cpp
  ${SOURCE_DIR}/pubsub.h
  ${SOURCE_DIR}/reqrep.cpp
  ${SOURCE_DIR}/reqrep.h
  ${SOURCE_DIR}/socket
  ${SOURCE_DIR}/socket.cpp
  ${SOURCE_DIR}/socket.h
  ${SOURCE_DIR}/socket.hpp
  ${SOURCE_DIR}/survey.cpp
  ${SOURCE_DIR}/survey.h
  ${SOURCE_DIR}/tcp.cpp
  ${SOURCE_DIR}/tcp.h
  ${SOURCE_DIR}/testing
  ${SOURCE_DIR}/timeout.cpp
  ${SOURCE_DIR}/timeout.h
  ${SOURCE_DIR}/unittest.h
  ${SOURCE_DIR}/../nanomsg/ext/nnxx_ext.c
)

add_library(nanomsgxx STATIC ${NANOMSGXX_SOURCES})
target_include_directories(nanomsgxx PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/>)
target_compile_definitions(nanomsgxx PRIVATE NN_STATIC_LIB)
set_property(TARGET nanomsgxx PROPERTY CXX_STANDARD 14)

if(WIN32) 
	target_compile_options(nanomsgxx PUBLIC /EHs)
endif()

# Dependency: nanomsg static library
# If nanomsg is not defined as a target, get it as an external project

if (NOT TARGET nanomsg)
  include(ExternalProject)
  ExternalProject_Add(nanomsg_external
    GIT_REPOSITORY        https://github.com/nanomsg/nanomsg.git
    GIT_TAG               master
    #URL                   ${CMAKE_CURRENT_SOURCE_DIR}/nanomsg
    SOURCE_DIR            ${CMAKE_CURRENT_BINARY_DIR}/nanomsg
    CMAKE_ARGS            -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install -DNN_STATIC_LIB=ON
  )
  add_library(nanomsg STATIC IMPORTED)
  add_dependencies(nanomsg nanomsg_external)
  # TODO: make platform-independent
  set_target_properties(nanomsg PROPERTIES IMPORTED_LOCATION ${CMAKE_BINARY_DIR}/install/lib/nanomsg.lib)
  #set_target_properties(nanomsg PROPERTIES IMPORTED_LOCATION nanomsg.lib)
  set_property(TARGET nanomsg PROPERTIES INTERFACE_LINK_DIRECTORIES ${CMAKE_BINARY_DIR}/install/include)
  if (WIN32)
      set_target_properties(nanomsg PROPERTIES INTERFACE_LINK_LIBRARIES "ws2_32;mswsock;advapi32")
  endif()
  include_directories(${CMAKE_BINARY_DIR}/install/include)
  link_directories(${CMAKE_BINARY_DIR}/install/lib)
endif()

target_link_libraries(nanomsgxx PUBLIC nanomsg)

# Installation 

install(DIRECTORY ${SOURCE_DIR} DESTINATION include/nnxx FILES_MATCHING PATTERN "*.h")
install(DIRECTORY ${SOURCE_DIR} DESTINATION include/nnxx FILES_MATCHING PATTERN "*.hpp")

install (TARGETS nanomsgxx
         ARCHIVE DESTINATION lib
         LIBRARY DESTINATION lib
         RUNTIME DESTINATION bin)
         