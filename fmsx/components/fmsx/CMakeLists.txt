set(COMPONENT_SRCS
    # ESP32 port
    msxfix.c

    # EMULib core (без NetPlay, IPS, Unix)
    src/EMULib/AY8910.c
    src/EMULib/Console.c
    src/EMULib/EMULib.c
    src/EMULib/FDIDisk.c
    src/EMULib/Floppy.c
    src/EMULib/Hunt.c
    src/EMULib/I8255.c
    src/EMULib/Image.c
    src/EMULib/MCF.c
    src/EMULib/Record.c
    src/EMULib/SCC.c
    src/EMULib/SHA1.c
    src/EMULib/Sound.c
    src/EMULib/Touch.c
    src/EMULib/WD1793.c
    src/EMULib/YM2413.c

    # EMULib/emu - Okazaki audio chips (все нужны)
    src/EMULib/emu/emu2149.c
    src/EMULib/emu/emu2212.c
    src/EMULib/emu/emu2413.c
    src/EMULib/emu/ym2151.c

    # fMSX
    src/fMSX/fMSX.c
    src/fMSX/I8251.c
    src/fMSX/Menu.c
    src/fMSX/MSX.c
    src/fMSX/Patch.c
    src/fMSX/tinf_zlib.c
    src/fMSX/V9938.c

    # Z80 (без ConDebug.c, Debug.c)
    src/Z80/Z80.c
)

set(COMPONENT_ADD_INCLUDEDIRS ". src/EMULib src/EMULib/emu src/fMSX src/Z80")
set(COMPONENT_REQUIRES "retro-go")
register_component()

# ===== LTO только для эмулятора fmsx =====
component_compile_options(-flto -fno-fat-lto-objects)
# ==========================================

# === Общие флаги ===
rg_setup_compile_options(
    -Wno-error=misleading-indentation
    -Wno-error=format-overflow
    -Wno-implicit-fallthrough
    -Wno-format-overflow
    -Wno-stringop-truncation
    -Wno-unused-but-set-variable
    -Wno-unused-variable
    -DBPS16 -DUNIX -DLSB_FIRST
    -DNARROW
    -DZLIB
    -fomit-frame-pointer
    -fno-strict-aliasing
    -mlongcalls
    -mtarget-align
    -fshort-enums
    -ffunction-sections
    -fdata-sections
)

# === msxfix.h для всех C-файлов ===
file(GLOB_RECURSE c_sources "src/*.c")
set_source_files_properties(${c_sources} PROPERTIES COMPILE_FLAGS "-include msxfix.h")

# === Отключаем -Werror для Okazaki кода ===
file(GLOB okazaki_sources "src/EMULib/emu/*.c")
set_source_files_properties(${okazaki_sources}
    PROPERTIES COMPILE_FLAGS
    "-O2 -include msxfix.h -Wno-error -Wno-implicit-fallthrough -Wno-parentheses"
#    "-O2 -include msxfix.h -Wno-error -Wno-implicit-fallthrough -Wno-parentheses -ffast-math"
)

# === Z80.c: 32KB I-cache ===
set_source_files_properties(
    "${CMAKE_CURRENT_SOURCE_DIR}/src/Z80/Z80.c"
    PROPERTIES COMPILE_FLAGS "\
        -O3 \
        -fomit-frame-pointer \
        -fno-strict-aliasing \
        -fwrapv \
        -frename-registers \
        -fsched-interblock \
        -fshrink-wrap \
        -include msxfix.h"
)

# === MSX.c: ===
set_source_files_properties(
    "${CMAKE_CURRENT_SOURCE_DIR}/src/fMSX/MSX.c"
    PROPERTIES COMPILE_FLAGS "-O2 -fomit-frame-pointer -fshrink-wrap -fshort-enums -include msxfix.h -Wno-error=misleading-indentation"
)

# === V9938.c: ===
set_source_files_properties(
    "${CMAKE_CURRENT_SOURCE_DIR}/src/fMSX/V9938.c"
    PROPERTIES COMPILE_FLAGS "-O2 -fomit-frame-pointer -fshrink-wrap -fshort-enums -include msxfix.h"
)

target_link_options(${COMPONENT_TARGET} PRIVATE
    -Wl,--gc-sections
    -Wl,--strip-all
 #   -Wl, --section-start=.iram1.text=0x4038C000
)