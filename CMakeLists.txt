cmake_minimum_required( VERSION 3.5.1 )

project(lmbspecialops)
enable_testing()

if( NOT CMAKE_BUILD_TYPE )
        set( CMAKE_BUILD_TYPE "Release" CACHE STRING "Build configuration 'Release' or 'Debug'." FORCE )
        set( CMAKE_CONFIGURATION_TYPES "Release" CACHE STRING "Build configuration" FORCE )
endif()

option( BUILD_WITH_CUDA "If enabled builds cuda implementation of ops"  OFF )

option( TENSORFLOW_WINDOWS_USE_LIBS_BUILT_FROM_SOURCE "For Windows: Check if you will build it on Windows" ON )
set( TENSORFLOW_WINDOWS_LIBS_DIR "C:/SDKs/TensorFlow-cpu-1.3" CACHE PATH "For Windows: TensorFlow libraries folder path.  Please copy all .lib files to the folder." )
set( CMAKE_BUILD_TYPE "Release" CACHE STRING "Build Configuration Release or Debug." FORCE )


# enable all warnings
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall" )


add_subdirectory( src lib ) 
add_subdirectory( doc ) 
add_subdirectory( test test ) 

