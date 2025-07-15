cmake_minimum_required(VERSION 3.8)        # ROS 2 uses CMake ≥ 3.8
project(serial LANGUAGES CXX)

###############################################################################
# 1. Dependencies
###############################################################################
find_package(ament_cmake REQUIRED)

# Extra system libs depending on platform
if(APPLE)
  find_library(IOKIT_LIBRARY      IOKit)
  find_library(FOUNDATION_LIBRARY Foundation)
elseif(UNIX AND NOT APPLE)
  set(rt_LIBRARIES       rt)
  set(pthread_LIBRARIES  pthread)
endif()

###############################################################################
# 2. Source files
###############################################################################
set(serial_SRCS
  src/serial.cc
  include/serial/serial.h
  include/serial/v8stdint.h
)

if(APPLE)
  list(APPEND serial_SRCS
       src/impl/unix.cc
       src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
  list(APPEND serial_SRCS
       src/impl/unix.cc
       src/impl/list_ports/list_ports_linux.cc)
else()  # Windows
  list(APPEND serial_SRCS
       src/impl/win.cc
       src/impl/list_ports/list_ports_win.cc)
endif()

###############################################################################
# 3. Build the shared library
###############################################################################
add_library(${PROJECT_NAME} SHARED ${serial_SRCS})
target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_11)

if(APPLE)
  target_link_libraries(${PROJECT_NAME} PUBLIC ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX AND NOT APPLE)
  target_link_libraries(${PROJECT_NAME} PUBLIC rt pthread)
else()
  target_link_libraries(${PROJECT_NAME} PUBLIC setupapi)
endif()

###############################################################################
# 4. (Optional) example executable
###############################################################################
# add_executable(serial_example examples/serial_example.cc)
# target_link_libraries(serial_example PRIVATE ${PROJECT_NAME})

###############################################################################
# 5. Installation
###############################################################################
install(TARGETS ${PROJECT_NAME}
  EXPORT  export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

install(DIRECTORY include/ DESTINATION include)

###############################################################################
# 6. ament export & package
###############################################################################
ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

ament_package()

