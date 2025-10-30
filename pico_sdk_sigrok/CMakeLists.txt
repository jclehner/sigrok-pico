# == DO NOT EDIT THE FOLLOWING LINES for the Raspberry Pi Pico VS Code Extension to work ==
if(WIN32)
    set(USERHOME $ENV{USERPROFILE})
else()
    set(USERHOME $ENV{HOME})
endif()
set(sdkVersion 2.2.0)
set(toolchainVersion 13_3_Rel1)
set(picotoolVersion 2.1.0)
set(picoVscode ${USERHOME}/.pico-sdk/cmake/pico-vscode.cmake)
if (EXISTS ${picoVscode})
    include(${picoVscode})
endif()
# ====================================================================================
set(PICO_BOARD pico CACHE STRING "Board type")

cmake_minimum_required(VERSION 3.13)

include(pico_sdk_import.cmake)

project(pico_sdk_sigrok C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

# 0 = baseline, 1 = digital 26, 2 = digital 32
set(PICO_MODE 0)

#See https://github.com/pico-coder/sigrok-pico/pull/63/  
#setting this allows the DTR to be ignored.  In addition to this, the use of 
#tud_cdc_connected should be replaced with tud_ready.
add_definitions(-DPICO_STDIO_USB_CONNECTION_WITHOUT_DTR=1)
#add_definitions(-DPICO_USE_FASTEST_SUPPORTED_CLOCK=1)
add_definitions(-DSYS_CLK_MHZ=200)
add_definitions(-DPICO_MODE=${PICO_MODE})
pico_sdk_init()

set(EXE_NAME "pico_sdk_sigrok")

if(PICO_MODE EQUAL 0)
	list(APPEND EXE_NAME "_baseline")
elseif(PICO_MODE EQUAL 1)
	list(APPEND EXE_NAME "_dig26")
elseif(PICO_MODE EQUAL 2)
	list(APPEND EXE_NAME "_dig32")
else()
	message(FATAL_ERROR "invalid PICO_MODE value")
endif()

add_executable(pico_sdk_sigrok
  pico_sdk_sigrok.c
  sr_device.c
)

pico_enable_stdio_usb(pico_sdk_sigrok 1)
pico_enable_stdio_uart(pico_sdk_sigrok 0)

pico_add_extra_outputs(pico_sdk_sigrok)

target_link_libraries(
    pico_sdk_sigrok 
    pico_stdlib
    hardware_adc
    hardware_dma
    hardware_pio
    hardware_sync
#    hardware_sio
#    hardware_gpio
#    hardware_timer
    pico_multicore
    pico_stdio_usb
)
