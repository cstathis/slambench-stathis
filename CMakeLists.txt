cmake_minimum_required(VERSION 3.10)

project(slambench)

# CMake variables
#######################################################
MESSAGE(STATUS CMAKE_MODULE_PATH=${CMAKE_MODULE_PATH})

SET(EXECUTABLE_OUTPUT_PATH         ${PROJECT_BINARY_DIR}/bin CACHE PATH "Build directory" FORCE)
SET(LIBRARY_OUTPUT_PATH            ${PROJECT_BINARY_DIR}/lib CACHE PATH "Build directory" FORCE)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
SET(CMAKE_BUILD_FILES_DIRECTORY    ${PROJECT_BINARY_DIR}/bfd)
SET(CMAKE_BINARY_DIR               ${PROJECT_BINARY_DIR}/bindir)
SET(CMAKE_CACHEFILE_DIR            ${PROJECT_BINARY_DIR}/cache)
SET(CMAKE_BUILD_DIRECTORY          ${PROJECT_BINARY_DIR}/builddir)

# Most common compiler flags
#######################################################

set(GENERAL_COMPILATION_FLAGS "-g -O3 -Wall  -Wextra")

if(CMAKE_COMPILER_IS_GNUCC)
    add_definitions(-D_FILE_OFFSET_BITS=64) ## Workaround for 32 bit boards with fwrite 2G limit

    set(GENERAL_COMPILATION_FLAGS "${GENERAL_COMPILATION_FLAGS} -Wno-error=deprecated-declarations ") # Eigen raises deprecated declarations warnings and int-in-bool-context warnings.
    set(GENERAL_COMPILATION_FLAGS "${GENERAL_COMPILATION_FLAGS} -Wno-unknown-pragmas -Wno-error=deprecated-declarations -Wno-error=unused-result -Wno-error=reorder -Wno-deprecated-copy -Wno-misleading-indentation")
    set(GENERAL_COMPILATION_FLAGS "${GENERAL_COMPILATION_FLAGS} -Wno-error=implicit-fallthrough= -Wno-int-in-bool-context -Wno-catch-value -Wno-reorder -Wno-strict-aliasing -Wno-unused-parameter -Wno-unused-result -Wno-format")
endif()

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++17" COMPILER_SUPPORTS_CXX17)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
message(STATUS "Using flag -std=c++17")

set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   ${GENERAL_COMPILATION_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${GENERAL_COMPILATION_FLAGS}")


# DEFINE SLAMBENCH LIBS/INCLUDES
######################################################
find_package(Eigen3 REQUIRED)

SET(SLAMBENCH_INCLUDE_DIR  ${EIGEN3_INCLUDE_DIR} ${CMAKE_SOURCE_DIR}/framework/shared/include CACHE STRING "The include paths needed to use SLAMBENCH")
SET(SLAMBENCH_LIBRARIES    slambench-utils         -Wl,--whole-archive                slambench-io slambench-metrics  -Wl,--no-whole-archive)
SET(SLAMBENCH_C_WRAPPER                            -Wl,--whole-archive slambench-c-wrapper                            -Wl,--no-whole-archive)

# SLAMBENCH FUNCTIONS 
#######################################################
string(ASCII 27 Esc)
set(ColorReset  "${Esc}[m")
set(Red         "${Esc}[31m")
set(Green       "${Esc}[32m")

# TODO(stathis): Why is this here?
SET(NULL_FILE_PATH ${PROJECT_BINARY_DIR}/null.cpp)
if (NOT EXISTS ${NULL_FILE_PATH})
    file(WRITE ${NULL_FILE_PATH} "")
ENDIF()

# Invoked by benchmarks/*/CMakeLists.txt
# Ex. explore_implementations(orbslam2 src/*)
function(explore_implementations algorithm_name implementations_path)
    set(appname ${algorithm_name})

    # Looking for multiple versions of the same library e.g. "original".
    FILE(GLOB VERSIONS ${implementations_path})

    FOREACH(version_dir ${VERSIONS})
        IF(IS_DIRECTORY ${version_dir})
            # Store the name of the directory into the variable appversion.
            get_filename_component(appversion ${version_dir} NAME)
            MESSAGE(STATUS "Load version ${appversion} of ${appname}.")

            # Add the directory to the build.
            ADD_SUBDIRECTORY(${version_dir})
        ENDIF()
    ENDFOREACH(version_dir)
endfunction()

# Note(stathis): This is unused?
#function(generate_slam_binaries_only library_name )
#    message(STATUS "Potential interfaces are ${SLAMBENCH_INTERFACES}...")
#    FOREACH(INTERFACE ${SLAMBENCH_INTERFACES})
#        message(STATUS "${Green}Generation of ${library_name}-${INTERFACE} : Activated${ColorReset}")
#
#        add_executable(${library_name}-${INTERFACE}  ${NULL_FILE_PATH})
#        target_link_libraries(${library_name}-${INTERFACE} PRIVATE ${library_name} ${INTERFACE} ${library_name})
#        SET_TARGET_PROPERTIES(${library_name}-${INTERFACE} PROPERTIES LINK_FLAGS "${ARGN}")
#        install(TARGETS ${library_name}-${INTERFACE}  DESTINATION  bin/)
#    ENDFOREACH()
#endfunction()

# This is invoked by the specific version of the benchmark library.
# Ex. in benchmarks/orbslam2/src/original/CMakeLists.txt:
# generate_slam_library(${PROJECT_NAME}-original "")
# Where ${PROJECT_NAME} is orbslam2  
function(generate_slam_library library_name )
    # TODO(stathis): What is SLAMBENCH_INTERFACES?
    #message(FATAL_ERROR "Potential interfaces are ${SLAMBENCH_INTERFACES}...")
    #FOREACH(INTERFACE ${SLAMBENCH_INTERFACES})
    #    message(FATAL_ERROR "${Green}Generation of ${library_name}-${INTERFACE} : Activated${ColorReset}")
        # TODO(stathis): Why write an executable with an empty source file?
    #    add_executable(${library_name}-${INTERFACE}  ${NULL_FILE_PATH})
    #    target_link_libraries(${library_name}-${INTERFACE} PRIVATE ${library_name} ${INTERFACE} ${library_name})

        # ARGN is the list of arguments past the last accessed one
    #    SET_TARGET_PROPERTIES(${library_name}-${INTERFACE} PROPERTIES LINK_FLAGS "${ARGN}")
    #    install(TARGETS ${library_name}-${INTERFACE}  DESTINATION  bin/)
    #ENDFOREACH()

    message(STATUS "${Green}Generation of ${library_name}-library : Activated${ColorReset}")
    add_library(${library_name}-library SHARED ${NULL_FILE_PATH})

    # TODO(stathis): Why just the C wrapper?
    target_link_libraries(${library_name}-library PRIVATE ${SLAMBENCH_C_WRAPPER} ${library_name})
    SET_TARGET_PROPERTIES(${library_name}-library PROPERTIES LINK_FLAGS "${ARGN}")
    install(TARGETS ${library_name}-library  DESTINATION  lib/)

endfunction()

#### SUB FOLDERS ####
add_subdirectory(framework)  # SLAMBench framework
add_subdirectory(benchmarks) # Benchmarks
