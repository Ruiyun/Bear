
include(CheckFunctionExists)
check_function_exists(vfork HAVE_VFORK)
check_function_exists(execve HAVE_EXECVE)
check_function_exists(execv HAVE_EXECV)
check_function_exists(execvpe HAVE_EXECVPE)
check_function_exists(execvp HAVE_EXECVP)
check_function_exists(execvP HAVE_EXECVP2)
check_function_exists(execl HAVE_EXECL)
check_function_exists(execlp HAVE_EXECLP)
check_function_exists(execle HAVE_EXECLE)
include(CheckSymbolExists)
check_symbol_exists(_NSGetEnviron crt_externs.h NEED_NSGETENVIRON)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})


add_definitions(-D_GNU_SOURCE)


add_library(ear SHARED
    stringarray.c
    protocol.c
    environ.c
    execs.c)
target_link_libraries(ear ${CMAKE_DL_LIBS})
set_target_properties(ear PROPERTIES
    COMPILE_DEFINITIONS "CLIENT")
if(APPLE)
    # Macs have an entirely different linker.
    set_target_properties(ear PROPERTIES
        LINK_FLAGS "-Wl,-exported_symbols_list,${CMAKE_CURRENT_SOURCE_DIR}/AppleExportedSymbolsList")
else()
    set_target_properties(ear PROPERTIES
        LINK_FLAGS "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/ExportedSymbolsList")
endif()


link_directories(${LIBCONFIG_LIBRARY_DIRS})
add_executable(bear
    stringarray.c
    protocol.c
    main.c
    output.c
    filter.c
    json.c)
set_target_properties(bear PROPERTIES
    COMPILE_DEFINITIONS "SERVER")
include_directories(${LIBCONFIG_INCLUDE_DIRS})
target_link_libraries(bear ${LIBCONFIG_LIBRARIES})


include(GNUInstallDirs)
install(TARGETS bear ear
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES bear.conf
    DESTINATION ${CMAKE_INSTALL_SYSCONFDIR})
