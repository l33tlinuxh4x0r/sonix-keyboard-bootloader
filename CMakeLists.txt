cmake_minimum_required(VERSION 3.2.0)

project(jumploader LANGUAGES C ASM)

function(add_bootloader device)
    string(TOUPPER ${device} device_upper)

    add_executable(${device}.elf
        src/main.c
        src/proxy.s
    )

    target_compile_options(${device}.elf PUBLIC
        -mthumb
        -mcpu=cortex-m0

        -flto
        ${OPT}

        -Wall
        -Wextra
    )

    target_include_directories(${device}.elf PUBLIC
        CMSIS
    )

    target_compile_definitions(${device}.elf PUBLIC
        TARGET_${device_upper}
    )

    target_link_libraries(${device}.elf PUBLIC
        -nostartfiles
        -nostdlib
        "-T ${PROJECT_SOURCE_DIR}/linker.x"
    )

    add_custom_target(jumploader-${device}.bin ALL
        COMMAND arm-none-eabi-objcopy -j .text -j .data -O binary $<TARGET_FILE:${device}.elf> ${CMAKE_BINARY_DIR}/jumploader-${device}.bin
        DEPENDS ${device}.elf
        BYPRODUCTS jumploader-${device}.bin
    )
endfunction(add_bootloader)

add_bootloader(generic)
add_bootloader(womier_k66)
add_bootloader(ganss_gk87pro)
add_bootloader(kemove_dk63)
add_bootloader(keychron_k6_rgb)
add_bootloader(redragon_k556)
