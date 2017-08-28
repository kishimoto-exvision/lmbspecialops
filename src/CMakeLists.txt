find_package( PythonInterp REQUIRED )
find_package( OpenMP REQUIRED )


# retrieve tensorflow include dir
message( STATUS "PYTHON_EXECUTABLE: ${PYTHON_EXECUTABLE}" )

execute_process( COMMAND ${PYTHON_EXECUTABLE} "-c" "from __future__ import print_function; import tensorflow as tf; print(tf.sysconfig.get_include(), end='')" 
        OUTPUT_VARIABLE TENSORFLOW_INCLUDE_DIR )
message( STATUS "${TENSORFLOW_INCLUDE_DIR}" )

configure_file( config.h.in config.h )
if( BUILD_WITH_CUDA )
	find_package( CUDA REQUIRED )
	include( "${CMAKE_SOURCE_DIR}/cmake/cuda_gencode_options.cmake" )
	list( APPEND CUDA_NVCC_FLAGS "-std=c++11")
	list( APPEND CUDA_NVCC_FLAGS "${CUDA_GENCODE_STRING}" )
	list( APPEND CUDA_NVCC_FLAGS_DEBUG -g -G -O0 --ptxas-options=-v )

endif()

file( GLOB lmbspecialops_SOURCES  *.cc *.c )
file( GLOB lmbspecialops_CUSOURCES *.cu )


include_directories( 
    ${TENSORFLOW_INCLUDE_DIR}
	${PROJECT_BINARY_DIR}/lib
)

if( BUILD_WITH_CUDA )
	cuda_add_library( lmbspecialops SHARED ${lmbspecialops_SOURCES} ${lmbspecialops_CUSOURCES} )
else()
	add_library( lmbspecialops SHARED ${lmbspecialops_SOURCES} )
endif()
# do not add 'lib' prefix
set_target_properties( lmbspecialops PROPERTIES PREFIX "" )
set_target_properties( lmbspecialops PROPERTIES DEBUG_POSTFIX "_debug" )
set_target_properties( lmbspecialops PROPERTIES COMPILE_FLAGS "${OpenMP_CXX_FLAGS}" )
set_target_properties( lmbspecialops PROPERTIES LINK_FLAGS "${OpenMP_CXX_FLAGS}" )

if ( TENSORFLOW_WINDOWS_USE_LIBS_BUILT_FROM_SOURCE )
set_target_properties( lmbspecialops PROPERTIES LINK_FLAGS "/LIBPATH:${TENSORFLOW_WINDOWS_LIBS_DIR}" )
set( TENSORFLOW_WINDOWS_LIB_LIST
        zlibstatic.lib
        giflib.lib
        libpng12_static.lib
        libjpeg.lib
        lmdb.lib
        jsoncpp.lib
        farmhash.lib
        fft2d.lib
        highwayhash.lib
        libprotobuf.lib
        tf_protos_cc.lib
        tf_cc.lib
        tf_cc_ops.lib
        tf_cc_framework.lib
        tf_core_cpu.lib
        tf_core_direct_session.lib
        tf_core_framework.lib
        tf_core_lib.lib
        tf_core_ops.lib
)
target_link_libraries( lmbspecialops ${TENSORFLOW_WINDOWS_LIB_LIST} )
# It still fails to build tf_core_kernels.lib.
#target_link_libraries( lmbspecialops tf_core_kernels.lib )
set_target_properties( lmbspecialops PROPERTIES COMPILE_FLAGS "/wd4049 /wd4197 /wd4217 /wd4221 /wd4625 /wd4626 /wd4710 /wd4711 /wd4820" )
endif()

# use old ABI with gcc 5.x
set_target_properties( lmbspecialops PROPERTIES COMPILE_DEFINITIONS "_GLIBCXX_USE_CXX11_ABI=0" )
# enable c++11
set_target_properties( lmbspecialops PROPERTIES 
                CXX_STANDARD 11 
                CXX_STANDARD_REQUIRED ON 
                CXX_EXTENSIONS OFF)