cmake_minimum_required(VERSION 3.13)

include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)
include($ENV{PICO_SDK_PATH}/tools/CMakeLists.txt)

project(host_controller C CXX ASM)

pico_sdk_init()

add_executable(${PROJECT_NAME})

# Example source
target_sources(${PROJECT_NAME} PUBLIC
        ${CMAKE_CURRENT_LIST_DIR}/hid_controller.c
        ${CMAKE_CURRENT_LIST_DIR}/main.c
        )

# Make sure TinyUSB can find tusb_config.h
target_include_directories(${PROJECT_NAME} PUBLIC
        ${CMAKE_CURRENT_LIST_DIR})

# In addition to pico_stdlib required for common PicoSDK functionality, add dependency on tinyusb_host
# for TinyUSB device support and tinyusb_board for the additional board support library used by the example
target_link_libraries(${PROJECT_NAME} PUBLIC pico_stdlib tinyusb_host tinyusb_board)

pico_add_extra_outputs(${PROJECT_NAME})

pico_enable_stdio_usb(${PROJECT_NAME} 0)
pico_enable_stdio_uart(${PROJECT_NAME} 1)

add_custom_target(openocd_program 
        COMMAND sudo openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000" -c "program ${PROJECT_NAME}.elf verify reset exit"
        COMMENT "Program the pi pico with the pico probe."
        )

add_custom_target(openocd_server
        COMMAND sudo openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000"
        COMMENT "Start the openocd server for debugging."
)

add_custom_target(gdb_start
        COMMAND gdb-multiarch
        COMMENT "Start gbd session."
)