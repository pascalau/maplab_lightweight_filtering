cmake_minimum_required(VERSION 3.5)

project(lightweight_filtering
  VERSION 0.0.0
  LANGUAGES CXX
)

# Include helper packages
include(CTest)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Set C++ 11
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 11)
endif()

# Output more warnings
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Attempt to find catkinized kindr
find_package(kindr REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(glog QUIET)
if(NOT glog_FOUND)
  find_package(PkgConfig)
  pkg_check_modules(glog REQUIRED)
endif()


# Add header library target
add_library(${PROJECT_NAME} INTERFACE)
target_include_directories(${PROJECT_NAME} INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_link_libraries(${PROJECT_NAME} INTERFACE Eigen3::Eigen kindr::kindr glog)

# Add unit tests
if(BUILD_TESTING)
  enable_testing()
  find_package(GTest REQUIRED)
  add_executable(testState src/testState.cpp)
  target_link_libraries(testState ${PROJECT_NAME} GTest::GTest GTest::Main glog)
  add_test(testState testState)
  # add_executable(testSigmaPoints src/testSigmaPoints.cpp)
  # target_link_libraries(testSigmaPoints ${PROJECT_NAME} GTest::GTest GTest::Main glog)
  # add_test(testSigmaPoints testSigmaPoints)
  add_executable(testPrediction src/testPrediction.cpp)
  target_link_libraries(testPrediction ${PROJECT_NAME} GTest::GTest GTest::Main glog)
  add_test(testPrediction testPrediction)
  add_executable(testUpdate src/testUpdate.cpp)
  target_link_libraries(testUpdate ${PROJECT_NAME} GTest::GTest GTest::Main glog)
  add_test(testUpdate testUpdate)
  add_executable(testModelBase src/testModelBase.cpp)
  target_link_libraries(testModelBase ${PROJECT_NAME} GTest::GTest GTest::Main glog)
  add_test(testModelBase testModelBase)
  # add_executable(testFilterBase src/testFilterBase.cpp)
  # target_link_libraries(testFilterBase ${PROJECT_NAME} GTest::GTest GTest::Main glog)
  # add_test(testFilterBase testFilterBase)
  add_executable(testGIFPrediction src/testGIFPrediction.cpp)
  target_link_libraries(testGIFPrediction ${PROJECT_NAME} GTest::GTest GTest::Main glog)
  add_test(testGIFPrediction testGIFPrediction)
endif()

# Generate configure
install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}_export
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
# header files, note the "/" after include
install(DIRECTORY include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

export(EXPORT ${PROJECT_NAME}_export
  FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake
  NAMESPACE ${PROJECT_NAME}::)

install(EXPORT ${PROJECT_NAME}_export
  FILE ${PROJECT_NAME}Targets.cmake
  NAMESPACE ${CMAKE_PROJECT_NAME}::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/cmake)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

set(INCLUDE_INSTALL_DIR ${CMAKE_INSTALL_INCLUDEDIR})
configure_package_config_file(
  cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/cmake
  PATH_VARS INCLUDE_INSTALL_DIR)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/cmake)
