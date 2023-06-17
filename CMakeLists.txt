cmake_minimum_required(VERSION 3.23)

set(PROJECT_NAME SafeInt)
project(${PROJECT_NAME} VERSION 3.0.28)

set(LIBRARY_NAME SafeInt)

add_library(${LIBRARY_NAME} INTERFACE safe_math.h safe_math_impl.h SafeInt.hpp)

target_sources(${LIBRARY_NAME} INTERFACE safe_math.h safe_math_impl.h SafeInt.hpp)

target_include_directories(${LIBRARY_NAME}
  INTERFACE
  $<BUILD_INTERFACE:${${PROJECT_NAME}_SOURCE_DIR}>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

set_property(TARGET ${LIBRARY_NAME} PROPERTY CXX_STANDARD 11)
target_compile_features(${LIBRARY_NAME} INTERFACE cxx_std_11)

option(SAFEINT_BUILD_TESTS "Build tests (default OFF)" OFF)

# Runtime tests are:
# - default
# - without built-in 128-bit support
# - without intrinsics

# Compile time tests are:
# - default C++11
# - C++14
# - TODO - consider adding in 17, 20 just to see if anything breaks
# - compile without exceptions

# Supported compilers:
# - Microsoft
# - clang
# - gcc
# - Intel (not regularly tested)
# other compilers on a best effort
if(SAFEINT_BUILD_TESTS)
        add_subdirectory(Test/ClangTest)
        add_subdirectory(Test/GccTest)
        add_subdirectory(Test/MsvcTest)
        enable_testing()
endif()

add_library(${PROJECT_NAME}::${LIBRARY_NAME} ALIAS ${LIBRARY_NAME})

include(GNUInstallDirs)

install(TARGETS ${LIBRARY_NAME}
        EXPORT ${PROJECT_NAME}_Targets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

include(CMakePackageConfigHelpers)
write_basic_package_version_file("${PROJECT_NAME}ConfigVersion.cmake"
                                 VERSION ${PROJECT_VERSION}
                                 COMPATIBILITY SameMajorVersion)

configure_package_config_file(
  "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
  "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  INSTALL_DESTINATION
  ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)


install(EXPORT ${PROJECT_NAME}_Targets
        FILE ${PROJECT_NAME}Targets.cmake
        NAMESPACE ${EAO_NAMESPACE}::${LIBRARY_NAMESPACE}::
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
              "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

install(DIRECTORY "${PROJECT_SOURCE_DIR}/Source/Public/" DESTINATION include)
