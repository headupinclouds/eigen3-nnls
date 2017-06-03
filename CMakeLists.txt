cmake_minimum_required(VERSION 3.0)

# Add HunterGate to find Eigen dependency with Hunter
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.19.0.tar.gz"
    SHA1 "e9d11aee66607cfcfbec5e3811d4804135b5bfac"
)  

PROJECT(eigen3-nnls VERSION 1.0.0)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

### Leave CMAKE_CXX_FLAGS to toolchain:

# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -frtti -fexceptions -Wno-unknown-pragmas")
# set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -ggdb")
# set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2")

# set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_SHARED_LINKER_FLAGS_INIT})
# set(CMAKE_SHARED_LINKER_FLAGS ${CMAKE_SHARED_LINKER_FLAGS_INIT})
# set(CMAKE_MODULE_LINKER_FLAGS ${CMAKE_MODULE_LINKER_FLAGS_INIT})

hunter_add_package(Eigen)
FIND_PACKAGE(Eigen3 REQUIRED)

### Comment below and install in a standard location

#SET(NNLS_INSTALL_DIR "${EIGEN3_INCLUDE_DIR}/Eigen/src/NNLS")
#MESSAGE("NNLS install directory: ${NNLS_INSTALL_DIR}")

#INCLUDE_DIRECTORIES(${EIGEN3_INCLUDE_DIRS} "${CMAKE_CURRENT_SOURCE_DIR}/src")

add_library(eigen3-nnls INTERFACE)
target_include_directories(eigen3-nnls
  INTERFACE
  "$<BUILD_INTERFACE:src>" # i.e., submodule use
  )

### Just skip src folder

#add_subdirectory(src)

option(NNLS_BUILD_TEST "Build test program" OFF)
if(NNLS_BUILD_TEST)
  add_subdirectory(test)
endif()

#INSTALL(FILES src/nnls.h DESTINATION ${NNLS_INSTALL_DIR})

### Install ###
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: use 'targets_export_name'
configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
    "${project_config}"
     INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS ${PROJECT_NAME}
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    FILES
    src/nnls.h
    DESTINATION "${include_install_dir}/"
)

install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
  )

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
  )

