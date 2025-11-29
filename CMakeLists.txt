cmake_minimum_required(VERSION 3.0.2)
project(socketcan_interface)
include(FetchContent)

FetchContent_Declare(
  console_bridge
  GIT_REPOSITORY https://github.com/ros/console_bridge.git
  GIT_TAG 81ec67f6daf3cd19ef506e00f02efb1645597b9c
)
FetchContent_MakeAvailable(console_bridge)



include_directories(${console_bridge_SOURCE_DIR}/include)
set(console_bridge_DIR ${PROJECT_SOURCE_DIR})

FetchContent_Declare(
  class_loader
  GIT_REPOSITORY https://github.com/ros/class_loader.git
  GIT_TAG 007c4e2d359bd9294671ff5605a771affebe8de6
)
FetchContent_Populate(class_loader)

FetchContent_MakeAvailable(class_loader)
include_directories(${class_loader_SOURCE_DIR}/include)


# find_package(catkin REQUIRED
#   COMPONENTS
#     class_loader
# )



find_package(Boost REQUIRED
  COMPONENTS
    chrono
    system
    thread
)

find_package(Threads REQUIRED)

include_directories(
  include
  ${Boost_INCLUDE_DIRS}
)

# ${PROJECT_NAME}_string
add_library(${PROJECT_NAME}_string
    src/string.cpp
)

# socketcan_dump
add_executable(socketcan_dump
  src/candump.cpp
)

target_link_libraries(socketcan_dump
   ${PROJECT_NAME}_string
   ${console_bridge_LIBRARIES}
   
   ${Boost_LIBRARIES}
   ${CMAKE_THREAD_LIBS_INIT}
)

# socketcan_bcm
add_executable(socketcan_bcm
  src/canbcm.cpp
)

target_link_libraries(socketcan_bcm
   ${PROJECT_NAME}_string
   ${console_bridge_LIBRARIES}

   ${Boost_LIBRARIES}
)

# ${PROJECT_NAME}_plugin
add_library(${PROJECT_NAME}_plugin
  src/${PROJECT_NAME}_plugin.cpp
)
target_link_libraries(${PROJECT_NAME}_plugin
  ${console_bridge_LIBRARIES}
  ${catkin_LIBRARIES}
  ${Boost_LIBRARIES}
)

install(TARGETS
    socketcan_bcm
    socketcan_dump
    ${PROJECT_NAME}_plugin
    ${PROJECT_NAME}_string
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

install(DIRECTORY include/${PROJECT_NAME}/
    DESTINATION include/${PROJECT_NAME}
    FILES_MATCHING PATTERN "*.h"
)

install(FILES ${PROJECT_NAME}_plugin.xml
    DESTINATION share/${PROJECT_NAME}
)
if(CATKIN_ENABLE_TESTING)
  find_package(xmlrpcpp REQUIRED)

  catkin_add_gtest(${PROJECT_NAME}-test_dummy_interface
    test/test_dummy_interface.cpp
  )
  target_link_libraries(${PROJECT_NAME}-test_dummy_interface
    ${PROJECT_NAME}_string
    ${console_bridge_LIBRARIES}
 
    ${Boost_LIBRARIES}
  )

  catkin_add_gtest(${PROJECT_NAME}-test_delegates
    test/test_delegates.cpp
  )
  target_link_libraries(${PROJECT_NAME}-test_delegates
    ${PROJECT_NAME}_string
 
    ${Boost_LIBRARIES}
  )

  catkin_add_gtest(${PROJECT_NAME}-test_settings
    test/test_settings.cpp
  )
  target_include_directories(${PROJECT_NAME}-test_settings PRIVATE
    ${xmlrpcpp_INCLUDE_DIRS}
  )
  target_link_libraries(${PROJECT_NAME}-test_settings
    ${PROJECT_NAME}_string
 
    ${xmlrpcpp_LIBRARIES}
  )

  catkin_add_gtest(${PROJECT_NAME}-test_string
    test/test_string.cpp
  )
  target_link_libraries(${PROJECT_NAME}-test_string
    ${PROJECT_NAME}_string
    ${console_bridge_LIBRARIES}
 
  )

  catkin_add_gtest(${PROJECT_NAME}-test_filter
    test/test_filter.cpp
  )
  target_link_libraries(${PROJECT_NAME}-test_filter
    ${PROJECT_NAME}_string
    ${console_bridge_LIBRARIES}
 
    ${Boost_LIBRARIES}
  )

  catkin_add_gtest(${PROJECT_NAME}-test_dispatcher
    test/test_dispatcher.cpp
  )
  target_link_libraries(${PROJECT_NAME}-test_dispatcher
    ${PROJECT_NAME}_string
 
    ${Boost_LIBRARIES}
  )
  target_compile_options(${PROJECT_NAME}-test_dispatcher PRIVATE -Wno-deprecated-declarations)
endif()
