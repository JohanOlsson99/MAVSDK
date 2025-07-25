set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

find_package(MAVLink REQUIRED)

get_target_property(
    MAVLINK_INCLUDE_DIR
    MAVLink::mavlink
    INTERFACE_INCLUDE_DIRECTORIES
)

configure_file(version.h.in version.h)
configure_file(include/mavsdk/mavlink_include.h.in include/mavsdk/mavlink_include.h)

target_sources(mavsdk
    PRIVATE
    autopilot.cpp
    base64.cpp
    call_every_handler.cpp
    connection.cpp
    connection_result.cpp
    crc32.cpp
    system.cpp
    system_impl.cpp
    file_cache.cpp
    flight_mode.cpp
    fs_utils.cpp
    hostname_to_ip.cpp
    inflate_lzma.cpp
    math_utils.cpp
    mavsdk.cpp
    mavsdk_impl.cpp
    mavlink_channels.cpp
    mavlink_command_receiver.cpp
    mavlink_command_sender.cpp
    mavlink_component_metadata.cpp
    mavlink_ftp_client.cpp
    mavlink_ftp_server.cpp
    mavlink_mission_transfer_client.cpp
    mavlink_mission_transfer_server.cpp
    mavlink_parameter_cache.cpp
    mavlink_parameter_client.cpp
    mavlink_parameter_server.cpp
    mavlink_parameter_subscription.cpp
    mavlink_parameter_helper.cpp
    mavlink_receiver.cpp
    mavlink_request_message.cpp
    mavlink_request_message_handler.cpp
    mavlink_statustext_handler.cpp
    mavlink_message_handler.cpp
    param_value.cpp
    ping.cpp
    plugin_impl_base.cpp
    serial_connection.cpp
    server_component.cpp
    server_component_impl.cpp
    server_plugin_impl_base.cpp
    socket_holder.cpp
    tcp_client_connection.cpp
    tcp_server_connection.cpp
    timeout_handler.cpp
    udp_connection.cpp
    vehicle.cpp
    log.cpp
    cli_arg.cpp
    geometry.cpp
    mavsdk_time.cpp
    string_utils.cpp
    timesync.cpp
)

cmake_policy(SET CMP0079 NEW)
target_link_libraries(mavsdk
    PRIVATE
    Threads::Threads
    LibLZMA::LibLZMA
)

if (NOT BUILD_WITHOUT_CURL)
    target_sources(mavsdk
        PRIVATE
        curl_wrapper.cpp
        http_loader.cpp
    )

    target_link_libraries(mavsdk
        PRIVATE
        CURL::libcurl
    )

    list(APPEND UNIT_TEST_SOURCES
        ${PROJECT_SOURCE_DIR}/mavsdk/core/curl_test.cpp
        ${PROJECT_SOURCE_DIR}/mavsdk/core/hostname_to_ip_test.cpp
        ${PROJECT_SOURCE_DIR}/mavsdk/core/http_loader_test.cpp
    )
else()
    target_compile_definitions(mavsdk
        PRIVATE
        BUILD_WITHOUT_CURL=1
    )
endif()

set_target_properties(mavsdk PROPERTIES
    VERSION ${MAVSDK_VERSION_STRING}
    SOVERSION ${MAVSDK_SOVERSION_STRING}
)
if (MSVC)
    set_target_properties(mavsdk PROPERTIES
        DEBUG_POSTFIX "d"
    )
endif()

if (IOS)
    target_link_libraries(mavsdk
        PUBLIC
        "-framework Foundation"
        "-framework Security"
    )
endif()

# Link to Windows networking lib.
if (MSVC OR MINGW)
    target_link_libraries(mavsdk
        PRIVATE
        ws2_32
    )
endif()

if(ANDROID)
    target_link_libraries(mavsdk
        PRIVATE
        log
    )
endif()

if((BUILD_STATIC_MAVSDK_SERVER AND ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU")) OR
    (${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "(armv6|armv7|aarch64)"))
    target_link_libraries(mavsdk PRIVATE atomic)
endif()

set_target_properties(mavsdk
    PROPERTIES COMPILE_FLAGS ${warnings}
    )

target_include_directories(mavsdk
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/mavsdk>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include/mavsdk>
    $<BUILD_INTERFACE:${MAVLINK_INCLUDE_DIR}>
    $<INSTALL_INTERFACE:include>
    PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
    )

install(TARGETS mavsdk
    EXPORT mavsdk-targets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )

install(FILES
    include/mavsdk/autopilot.h
    include/mavsdk/base64.h
    include/mavsdk/component_type.h
    include/mavsdk/connection_result.h
    include/mavsdk/deprecated.h
    include/mavsdk/handle.h
    include/mavsdk/system.h
    include/mavsdk/mavsdk.h
    include/mavsdk/log_callback.h
    include/mavsdk/plugin_base.h
    include/mavsdk/server_plugin_base.h
    include/mavsdk/geometry.h
    include/mavsdk/server_component.h
    include/mavsdk/mavlink_address.h
    include/mavsdk/vehicle.h
    include/mavsdk/overloaded.h
    ${CMAKE_CURRENT_BINARY_DIR}/include/mavsdk/mavlink_include.h
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/mavsdk"
)

install(DIRECTORY ${MAVLINK_INCLUDE_DIR}/mavlink
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/mavsdk"
)

list(APPEND UNIT_TEST_SOURCES
    ${PROJECT_SOURCE_DIR}/mavsdk/core/callback_list_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/call_every_handler_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/cli_arg_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/file_cache_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/locked_queue_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/geometry_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/math_utils_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/mavsdk_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/mavsdk_time_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/mavlink_channels_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/mavlink_mission_transfer_client_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/mavlink_mission_transfer_server_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/mavlink_statustext_handler_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/ringbuffer_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/timeout_handler_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/unittests_main.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/mavlink_parameter_cache_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/string_utils_test.cpp
    ${PROJECT_SOURCE_DIR}/mavsdk/core/base64_test.cpp
)

if (NOT BUILD_WITHOUT_CURL)
    list(APPEND UNIT_TEST_SOURCES
        ${PROJECT_SOURCE_DIR}/mavsdk/core/mavlink_component_metadata_test.cpp
    )
endif()
set(UNIT_TEST_SOURCES ${UNIT_TEST_SOURCES} PARENT_SCOPE)
