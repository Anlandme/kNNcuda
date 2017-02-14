cmake_minimum_required(VERSION 2.8.8)


project(kdtreeCuda C CXX)

# ---[ utility
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules/")
include(cmake/Utils.cmake)
include(cmake/Targets.cmake)

# -- [ this option causes problems for some reason, so turn it off before searching for CUDA
if(MSVC)
	# Setting this to true brakes Visual Studio builds.
	set(CUDA_ATTACH_VS_BUILD_RULE_TO_CUDA_FILE OFF CACHE BOOL "CUDA_ATTACH_VS_BUILD_RULE_TO_CUDA_FILE")
endif()

# ---[ find dependencies
find_package(CUDA 8.0 REQUIRED)
find_package(Boost REQUIRED)

if(Boost_FOUND)
    include_directories(${Boost_INCLUDE_DIRS}) 
    message(STATUS "-- Found Boost package" )
else(Boost_FOUND)
    message( SEND_ERROR "-- could not find Boost package" )
endif(Boost_FOUND)


if(LINUX)
if ( GLUT_Xmu_LIBRARY )
    message( "-- Found X.Org X11 libXmu/libXmuu runtime libraries" )
else ( GLUT_Xmu_LIBRARY )
    message( SEND_ERROR "-- Could not find X.Org X11 libXmu/libXmuu runtime libraries" )
endif ( GLUT_Xmu_LIBRARY )


endif(LINUX)

add_definitions(-std=c++11)

include_directories(${CUDA_INCLUDE_DIRS})

# ---[ misc settings
if(USE_PROJECT_FOLDERS)
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
  set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMakeTargets")
endif()

# ---[ cuda settings
set(HAVE_CUDA 1)
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "--ftz=true;--prec-div=false;--prec-sqrt=false")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-arch=sm_50")



#if(UNIX OR APPLE)
#  list(APPEND CUDA_NVCC_FLAGS "-Xcompiler;-fPIC;")
#endif()

warnings_disable(CMAKE_CXX_FLAGS /wd4985)


set(NORMALS_CUDA_SRCS
	src/calcNormalsCuda.cu
)

#add library to link by calcNormalsCuda.h usage
cuda_add_library(normalsCuda ${NORMALS_CUDA_SRCS})

cuda_add_executable(cudaNormals src/main.cpp include/rply/rply.c)

target_link_libraries(cudaNormals
	normalsCuda
	${CUDA_LIBRARIES}
	${CUDA_CUDA_LIBRARY}
)