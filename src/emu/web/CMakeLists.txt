# ============================================================================
# EKA2L1 WebAssembly Frontend Build
# ============================================================================

# The web frontend executable
add_executable(eka2l1_wasm
    src/main.cpp
)

target_include_directories(eka2l1_wasm PUBLIC include)

# Link against all core emulator libraries
target_link_libraries(eka2l1_wasm PRIVATE
    common
    config
    cpu
    disasm
    dispatch
    drivers
    epoc
    epockern
    epocservs
    epocpkg
    epocloader
    epocio
    epocldd
    gdbstub
    bridge
    miniz
    FAT
    pugixml
    stb
    yaml-cpp
)

# Link Emscripten-provided libraries
if (EMSCRIPTEN)
    target_link_libraries(eka2l1_wasm PRIVATE
        sdl2
        gl
    )

    # Set Emscripten linker flags for WASM output
    set_target_properties(eka2l1_wasm PROPERTIES
        LINK_FLAGS "${EMSCRIPTEN_LINKER_FLAGS}"
        OUTPUT_NAME "eka2l1"
        SUFFIX ".js"
    )

    # Copy resources to build directory for Emscripten's file packaging
    add_custom_command(TARGET eka2l1_wasm POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:eka2l1_wasm>/resources"
        COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:eka2l1_wasm>/resources/upscale"
        COMMAND ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:eka2l1_wasm>/miscs"
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_SOURCE_DIR}/../drivers/resources/gl/" "$<TARGET_FILE_DIR:eka2l1_wasm>/resources/"
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_SOURCE_DIR}/../drivers/resources/upscale/" "$<TARGET_FILE_DIR:eka2l1_wasm>/resources/upscale/"
        COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_CURRENT_SOURCE_DIR}/../drivers/resources/defaultbank.hsb" "$<TARGET_FILE_DIR:eka2l1_wasm>/resources/"
        COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_CURRENT_SOURCE_DIR}/../drivers/resources/defaultbank.sf2" "$<TARGET_FILE_DIR:eka2l1_wasm>/resources/"
        COMMAND ${CMAKE_COMMAND} -E copy_directory "${PROJECT_SOURCE_DIR}/miscs/" "$<TARGET_FILE_DIR:eka2l1_wasm>/miscs/"
    )
endif()

# Set output directory
set_target_properties(eka2l1_wasm PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)