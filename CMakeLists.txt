cmake_minimum_required(VERSION 3.0.0)

project(SignalEstimator)

option(BUILD_GUI "Build Qt GUI" ON)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
endif()

set(COMPILER_FLAGS "")

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  list(APPEND COMPILER_FLAGS
    "-Weverything"
    "-Wno-alloca"
    "-Wno-c++98-c++11-c++14-compat"
    "-Wno-c++98-compat"
    "-Wno-c++98-compat-pedantic"
    "-Wno-double-promotion"
    "-Wno-float-equal"
    "-Wno-gnu-zero-variadic-macro-arguments"
    "-Wno-implicit-int-float-conversion"
    "-Wno-old-style-cast"
    "-Wno-padded"
    "-Wno-reserved-id-macro"
    "-Wno-switch-enum"
    "-Wno-system-headers"
    "-Wno-unused-private-field"
    "-Wno-used-but-marked-unused"
    "-Wno-weak-vtables"
    )
endif()

if(${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
  list(APPEND COMPILER_FLAGS
    "-Wall"
    "-Wextra"
    "-Wno-system-headers"
    "-Wno-old-style-cast"
    "-Wno-switch-enum"
    )
endif()

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

add_executable(signal-estimator
  src/cli/AlsaReader.cpp
  src/cli/AlsaUtils.cpp
  src/cli/AlsaWriter.cpp
  src/cli/ContinuousGenerator.cpp
  src/cli/FileDumper.cpp
  src/cli/Frame.cpp
  src/cli/GradientCounter.cpp
  src/cli/JSONFormatter.cpp
  src/cli/LatencyEstimator.cpp
  src/cli/LossEstimator.cpp
  src/cli/Main.cpp
  src/cli/RateLimiter.cpp
  src/cli/Realtime.cpp
  src/cli/RunMaxCounter.cpp
  src/cli/SchmittTrigger.cpp
  src/cli/SmaCounter.cpp
  src/cli/StrikeGenerator.cpp
  src/cli/TextFormatter.cpp
  src/cli/Time.cpp
  )

target_compile_options(signal-estimator
  PRIVATE ${COMPILER_FLAGS}
  )

target_include_directories(signal-estimator SYSTEM
  PRIVATE 3rdparty/cxxopts/include
  )

find_package(Threads)
target_link_libraries(signal-estimator
  asound
  m
  ${CMAKE_THREAD_LIBS_INIT}
  )

install(
  TARGETS signal-estimator
  RUNTIME DESTINATION bin
  )

if(BUILD_GUI)
  find_package(Qt5Core REQUIRED)
  find_package(Qt5Widgets REQUIRED)
  find_package(Qwt REQUIRED)

  qt5_wrap_cpp(GENERATED_SOURCES
    src/gui/MainWindow.hpp
    src/gui/NotFoundDialog.hpp
    )

  set(UI_SOURCES
    src/gui/MainWindow.ui
    src/gui/NotFoundDialog.ui
    )

  foreach(UI_FILE IN LISTS UI_SOURCES)
    qt5_wrap_ui(GENERATED_SOURCES ${UI_FILE})
  endforeach()

  add_library(generated_objects OBJECT
    ${GENERATED_SOURCES}
    )

  add_executable(signal-estimator-gui
    src/gui/Devices.cpp
    src/gui/Main.cpp
    src/gui/MainWindow.cpp
    src/gui/NotFoundDialog.cpp
    src/gui/PointsBuffer.cpp
    src/gui/SignalEstimator.cpp
    )

  target_compile_options(signal-estimator-gui
    PRIVATE ${COMPILER_FLAGS}
    )

  foreach(TARGET IN ITEMS generated_objects signal-estimator-gui)
    target_include_directories(${TARGET} SYSTEM PRIVATE
      ${CMAKE_CURRENT_BINARY_DIR}
      ${QWT_INCLUDE_DIR}
      ${Qt5Core_INCLUDE_DIRS}
      ${Qt5Widgets_INCLUDE_DIRS}
      )
  endforeach()

  target_link_libraries(signal-estimator-gui
    generated_objects
    ${QWT_LIBRARY}
    Qt5::Core
    Qt5::Widgets
    )

  install(
    TARGETS signal-estimator-gui
    RUNTIME DESTINATION bin
    )
endif(BUILD_GUI)

add_custom_command(TARGET signal-estimator POST_BUILD
  COMMAND "${CMAKE_COMMAND}" -E copy
     "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json"
     "${PROJECT_SOURCE_DIR}/compile_commands.json"
     COMMENT "Copying compile_commands.json to project root"
     )
