cmake_minimum_required(VERSION 3.27)

project(lua
    VERSION 5.4.8
    LANGUAGES C
    DESCRIPTION "Lua embeddable scripting language"
)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
    message(FATAL_ERROR "In-source builds are not allowed. Please create a separate build directory.")
endif()

option(LUA_BUILD_SHARED "Build lua shared library" OFF)
option(LUA_BUILD_STATIC "Build lua static library" ON)
option(LUA_BUILD_COMPILER "Build luac compiler" OFF)
option(LUA_BUILD_INTERPRETER "Build lua interpreter" OFF)

set(LUA_CORE_SOURCES
    lapi.c
    lcode.c
    lctype.c
    ldebug.c
    ldo.c
    ldump.c
    lfunc.c
    lgc.c
    llex.c
    lmem.c
    lobject.c
    lopcodes.c
    lparser.c
    lstate.c
    lstring.c
    ltable.c
    ltm.c
    lundump.c
    lvm.c
    lzio.c
)

set(LUA_AUX_SOURCES
    lauxlib.c
)

set(LUA_LIB_SOURCES
    lbaselib.c
    lcorolib.c
    ldblib.c
    liolib.c
    lmathlib.c
    loadlib.c
    loslib.c
    lstrlib.c
    ltablib.c
    lutf8lib.c
    linit.c
)

set(LUA_ALL_SOURCES
    ${LUA_CORE_SOURCES}
    ${LUA_AUX_SOURCES}
    ${LUA_LIB_SOURCES}
)

if(LUA_BUILD_STATIC)
    add_library(lua_static STATIC ${LUA_ALL_SOURCES})
    add_library(lua::lua ALIAS lua_static)
    
    target_include_directories(lua_static
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:include>
    )
    
    set_target_properties(lua_static PROPERTIES
        OUTPUT_NAME lua
        POSITION_INDEPENDENT_CODE ON
        C_STANDARD 99
        C_STANDARD_REQUIRED ON
    )
    
    if(UNIX AND NOT APPLE AND NOT EMSCRIPTEN)
        target_compile_definitions(lua_static PUBLIC LUA_USE_LINUX)
        target_link_libraries(lua_static PUBLIC m dl)
    elseif(APPLE)
        target_compile_definitions(lua_static PUBLIC LUA_USE_MACOSX)
        target_link_libraries(lua_static PUBLIC m)
    elseif(WIN32)
        target_compile_definitions(lua_static PUBLIC LUA_USE_WINDOWS)
    elseif(EMSCRIPTEN)
        target_compile_definitions(lua_static PUBLIC LUA_USE_POSIX)
    else()
        target_compile_definitions(lua_static PUBLIC LUA_USE_POSIX)
        target_link_libraries(lua_static PUBLIC m)
    endif()
endif()

if(LUA_BUILD_SHARED)
    add_library(lua_shared SHARED ${LUA_ALL_SOURCES})
    add_library(lua::shared ALIAS lua_shared)
    
    target_include_directories(lua_shared
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            $<INSTALL_INTERFACE:include>
    )
    
    set_target_properties(lua_shared PROPERTIES
        OUTPUT_NAME lua
        C_STANDARD 99
        C_STANDARD_REQUIRED ON
    )
    
    if(UNIX AND NOT APPLE AND NOT EMSCRIPTEN)
        target_compile_definitions(lua_shared PUBLIC LUA_USE_LINUX)
        target_link_libraries(lua_shared PUBLIC m dl)
    elseif(APPLE)
        target_compile_definitions(lua_shared PUBLIC LUA_USE_MACOSX)
        target_link_libraries(lua_shared PUBLIC m)
    elseif(WIN32)
        target_compile_definitions(lua_shared PUBLIC LUA_USE_WINDOWS)
    elseif(EMSCRIPTEN)
        target_compile_definitions(lua_shared PUBLIC LUA_USE_POSIX)
    else()
        target_compile_definitions(lua_shared PUBLIC LUA_USE_POSIX)
        target_link_libraries(lua_shared PUBLIC m)
    endif()
endif()

if(LUA_BUILD_INTERPRETER)
    if(NOT LUA_BUILD_STATIC)
        message(FATAL_ERROR "LUA_BUILD_INTERPRETER requires LUA_BUILD_STATIC=ON")
    endif()
    
    add_executable(lua_interpreter lua.c)
    set_target_properties(lua_interpreter PROPERTIES OUTPUT_NAME lua)
    target_link_libraries(lua_interpreter PRIVATE lua_static)
    
    if(UNIX AND NOT APPLE)
        target_link_options(lua_interpreter PRIVATE -Wl,-E)
    endif()
endif()

if(CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
    include(GNUInstallDirs)
    
    if(LUA_BUILD_STATIC)
        install(TARGETS lua_static
            EXPORT lua-targets
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )
    endif()
    
    if(LUA_BUILD_SHARED)
        install(TARGETS lua_shared
            EXPORT lua-targets
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        )
    endif()
    
    install(FILES
        lua.h
        luaconf.h
        lualib.h
        lauxlib.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )
endif()