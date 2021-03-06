cmake_minimum_required(VERSION 2.8)
SET(CMAKE_VERBOSE_MAKEFILE on)

project(grr_minicomm)

include_directories(../../..)
list(APPEND CMAKE_CXX_FLAGS "-std=c++0x ${CMAKE_CXX_FLAGS}")

add_library(core_lib
  ../../proto/jobs.pb.cc
  ../../proto/semantic.pb.cc
  compression.cc
  config.cc config.pb.cc
  crypto.cc
  message_queue.cc
  comms_utils.cc
  util.cc
)
set(core_lib_DEPS crypto protobuf z)

add_library(http_comms_lib http_connection.cc)
set(http_comms_lib_DEPS core_lib curl ${core_lib_DEPS})

add_library(subprocess_delegator_lib subprocess_delegator.cc)
set(subprocess_delegator_lib_DEPS core_lib ${core_lib_DEPS})

add_library(client_actions_lib client_action.cc
  ../../proto/sysinfo.pb.cc
  client_actions/enumerate_filesystems.cc
  client_actions/enumerate_interfaces.cc
  client_actions/get_platform_info.cc
)
set(set_client_actions_lib_DEPS core_lib ${core_lib_DEPS})

add_executable(minicomm client_main.cc client.cc)
set(minicom_external_DEPS
  ${subprocess_delegator_lib_DEPS}
  ${http_comms_lib_DEPS}
  ${core_lib_DEPS})
list(REMOVE_DUPLICATES minicom_external_DEPS)
target_link_libraries(minicomm
  core_lib
  http_comms_lib
  subprocess_delegator_lib
  ${minicom_external_DEPS}
)

install(TARGETS minicomm DESTINATION bin)

# Tests Libraries

find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIR})

add_library(test_helper_lib
  client_test_base.cc
  logging_control.cc
  test_util.cc)

set(test_helper_lib_DEPS gtest pthread)

add_executable(unit_tests grr_test_main.cc
  client_action_test.cc
  client_actions/enumerate_filesystems_test.cc
  client_actions/enumerate_interfaces_test.cc
  client_actions/get_platform_info_test.cc
  compression_test.cc
  comms_utils_test.cc
  config_test.cc
  crypto_test.cc
  message_queue_test.cc
  subprocess_delegator_test.cc
  util_test.cc
)
set(unit_tests_external_DEPS
  ${client_actions_lib_DEPS}
  ${core_lib_DEPS}
  ${subprocess_delegator_lib_DEPS}
  ${test_helper_lib_DEPS})
list(REMOVE_DUPLICATES unit_tests_external_DEPS)
target_link_libraries(unit_tests
  client_actions_lib
  core_lib
  subprocess_delegator_lib
  test_helper_lib
  ${unit_tests_external_DEPS}
)
