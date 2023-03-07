# Set minimumum cmake version
cmake_minimum_required(VERSION 3.20.0)

# set project name 
project(F303Cmake VERSION 0.0.1 DESCRIPTION "Build stm32 project using cmake")

# Configure language options
enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED TRUE)
set(CMAKE_C_EXTENSIONS OFF)

# include file with includes and sources
include("ProjectFiles.cmake")

# set executable name
set(EXECUTABLE ${CMAKE_PROJECT_NAME}.elf)

# Add executable
add_executable(${EXECUTABLE} ${SRCS})

# Add includes
target_include_directories(${EXECUTABLE} PRIVATE ${INCS})

# add compiler options
target_compile_definitions(${EXECUTABLE} PRIVATE -DUSE_HAL_DRIVER 
-DSTM32F303xE)

target_compile_options(${EXECUTABLE} PRIVATE 
  -mcpu=cortex-m4
  -mthumb
  -mfpu=fpv4-sp-d16
  -mfloat-abi=hard

  -fdata-sections
  -ffunction-sections

  -Wall

  $<$<CONFIG:Debug>:-Og>
)

# add linker options
target_link_options(${EXECUTABLE} PRIVATE 
  -T${CMAKE_SOURCE_DIR}/STM32F303RETx_FLASH.ld
  -mcpu=cortex-m4
  -mthumb
  -mfpu=fpv4-sp-d16
  -mfloat-abi=hard
  -specs=nano.specs
  -lc
  -Wl,--print-memory-usage
  -lm
  -lnosys
  -Wl,-Map=${CMAKE_PROJECT_NAME}.map,--cref
  -Wl,--gc-sections
)


# Post build commands

# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-size ${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-objcopy -O ihex ${EXECUTABLE} ${CMAKE_PROJECT_NAME}.hex
        COMMAND arm-none-eabi-objcopy -O binary ${EXECUTABLE} ${CMAKE_PROJECT_NAME}.bin)