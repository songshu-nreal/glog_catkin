cmake_minimum_required(VERSION 2.8.3)
project(glog_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

set(VERSION 0.3.5)

if(ANDROID_NREAL)
    string(REPLACE "\"" "" TMP_BUILD_CONFIG ${ANDROID_NREAL_BUILD_CONFIG})
    string(REPLACE " " ";" BUILD_CONFIG ${TMP_BUILD_CONFIG})
endif()

ExternalProject_Add(glog_src
        URL file://${CMAKE_CURRENT_SOURCE_DIR}/v${VERSION}.zip
        UPDATE_COMMAND ""
        PATCH_COMMAND patch -p0 < ${CMAKE_SOURCE_DIR}/fix-unused-typedef-warning.patch &&
        patch -p0 < ${CMAKE_SOURCE_DIR}/fix-find-gflags.patch
        CONFIGURE_COMMAND cd ../glog_src/ && cmake .
        -DCMAKE_INSTALL_PREFIX:PATH=${CATKIN_DEVEL_PREFIX}
        -DBUILD_SHARED_LIBS:BOOL=false
        -DGFLAGS_LIBRARY_PATH=${CATKIN_DEVEL_PREFIX}/lib
        -Dgflags_DIR=${CATKIN_DEVEL_PREFIX}/lib/cmake/gflags
        -DBUILD_TESTING=OFF
        ${BUILD_CONFIG}
        BUILD_COMMAND cd ../glog_src/ && make -j 8
        INSTALL_COMMAND cd ../glog_src/ && make install -j 8
        )

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)
add_dependencies(${PROJECT_NAME} glog_src)
#target_link_libraries(${PROJECT_NAME} ${CATKIN_DEVEL_PREFIX}/lib/libglog${CMAKE_SHARED_LIBRARY_SUFFIX})
target_link_libraries(${PROJECT_NAME} ${CATKIN_DEVEL_PREFIX}/lib/libglog${CMAKE_STATIC_LIBRARY_SUFFIX})

cs_install()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS glog-extras.cmake)
