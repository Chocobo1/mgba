find_program(FIXUP vita-elf-create)
find_program(OBJCOPY ${cross_prefix}objcopy)
find_file(NIDDB db.json PATHS ${VITASDK} ${VITASDK}/bin ${VITASDK}/share)
find_file(EXTRADB extra.json PATHS ${VITASDK}${VITASDK}/bin ${VITASDK}/share)
find_program(STRIP ${cross_prefix}strip)

file(GLOB OS_SRC ${CMAKE_SOURCE_DIR}/src/platform/psp2/psp2-*.c)
set(OS_SRC ${OS_SRC} PARENT_SCOPE)
source_group("PS Vita-specific code" FILES ${OS_SRC})

list(APPEND VFS_SRC ${CMAKE_CURRENT_SOURCE_DIR}/sce-vfs.c)
set(VFS_SRC ${VFS_SRC} PARENT_SCOPE)

set(OS_LIB -lvita2d -lSceCtrl_stub -lSceGxm_stub -lSceDisplay_stub -lSceAudio_stub -lSceMotion_stub -lScePower_stub -lSceTouch_stub -lSceCommonDialog_stub -lpng -lz -l${M_LIBRARY})
set(OBJCOPY_CMD ${OBJCOPY} -I binary -O elf32-littlearm -B arm)

list(APPEND GUI_SRC ${CMAKE_CURRENT_SOURCE_DIR}/gui-font.c)

set_source_files_properties(${CMAKE_CURRENT_BINARY_DIR}/font.o ${CMAKE_CURRENT_BINARY_DIR}/backdrop.o PROPERTIES GENERATED ON)
add_executable(${BINARY_NAME}.elf ${PLATFORM_SRC} ${GUI_SRC} ${CMAKE_CURRENT_BINARY_DIR}/font.o ${CMAKE_CURRENT_BINARY_DIR}/backdrop.o main.c)
set_target_properties(${BINARY_NAME}.elf PROPERTIES COMPILE_DEFINITIONS "${OS_DEFINES};${FEATURE_DEFINES};${FUNCTION_DEFINES}")
target_link_libraries(${BINARY_NAME}.elf ${BINARY_NAME} ${OS_LIB})

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/font.o
                   COMMAND ${OBJCOPY_CMD} font.png ${CMAKE_CURRENT_BINARY_DIR}/font.o
                   WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/res)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/backdrop.o
                   COMMAND ${OBJCOPY_CMD} backdrop.png ${CMAKE_CURRENT_BINARY_DIR}/backdrop.o
                   WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

add_custom_target(${BINARY_NAME}.velf ALL
                  ${STRIP} --strip-unneeded -go ${BINARY_NAME}-stripped.elf ${BINARY_NAME}.elf
                  COMMAND ${FIXUP} ${BINARY_NAME}-stripped.elf ${BINARY_NAME}.velf ${NIDDB} ${EXTRADB}
                  DEPENDS ${BINARY_NAME}.elf)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${BINARY_NAME}.velf DESTINATION . COMPONENT ${BINARY_NAME}-psp2)
