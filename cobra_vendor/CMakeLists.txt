cmake_minimum_required(VERSION 3.16)
project(cobra_vendor NONE)

find_package(ament_cmake REQUIRED)

set(extra_cmake_args)

if(DEFINED CMAKE_BUILD_TYPE)
  list(APPEND extra_cmake_args "-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")
endif()

# The Cobra source code version/revision numbers
set(VER "4.8")
set(REV "5af7230ecf29a6c9b46ff1bdd60c5d9e51d032db")

include(ExternalProject)
ExternalProject_Add(cobra-${VER}
  PREFIX cobra-${VER}
  GIT_REPOSITORY https://github.com/nimble-code/cobra
  GIT_TAG ${REV}
  INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/cobra_install"
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND ""
  BUILD_COMMAND
  ${CMAKE_COMMAND} -E chdir <SOURCE_DIR>/src make all && ${CMAKE_COMMAND} -E chdir <SOURCE_DIR>/src_app make all LDFLAGS=-Wl,c.ar\ -pthread
  INSTALL_COMMAND
  # after cobra 4.8 the makefile installs things into /usr/local/bin, hence need sudo here
  ${CMAKE_COMMAND} -E chdir <SOURCE_DIR>/src sudo make install MAN=<SOURCE_DIR>/src_app LDFLAGS=-Wl,c.ar\ -pthread && ${CMAKE_COMMAND} -E chdir <SOURCE_DIR>/src_app sudo make install LDFLAGS=-Wl,c.ar\ -pthread
)

######## after cobra 4.8 the cobra makefile installer installs all of the below into /usr/local/bin. bin_linux dir is not generated
#install(PROGRAMS
#  ${CMAKE_CURRENT_BINARY_DIR}/cobra-${VER}/src/cobra-${VER}/bin_linux/cobra
#  ${CMAKE_CURRENT_BINARY_DIR}/cobra-${VER}/src/cobra-${VER}/bin_linux/cwe
#  ${CMAKE_CURRENT_BINARY_DIR}/cobra-${VER}/src/cobra-${VER}/bin_linux/json_convert
#  ${CMAKE_CURRENT_BINARY_DIR}/cobra-${VER}/src/cobra-${VER}/bin_linux/scope_check
#  ${CMAKE_CURRENT_BINARY_DIR}/cobra-${VER}/src/cobra-${VER}/bin_linux/find_taint
#  ${CMAKE_CURRENT_BINARY_DIR}/cobra-${VER}/src/cobra-${VER}/bin_linux/window.tcl
#  DESTINATION bin)

install(DIRECTORY 
  ${CMAKE_CURRENT_BINARY_DIR}/cobra-${VER}/src/cobra-${VER}/rules
  DESTINATION share/${PROJECT_NAME}
)

ament_environment_hooks(env_hook/cobra_vendor_base.dsv.in)

ament_package()
