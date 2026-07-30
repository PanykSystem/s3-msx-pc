set(COMPONENT_SRCDIRS ".")
set(COMPONENT_ADD_INCLUDEDIRS ".")
register_component()

# LTO для main.c
component_compile_options(-flto -fno-fat-lto-objects)

rg_setup_compile_options(-O3)
