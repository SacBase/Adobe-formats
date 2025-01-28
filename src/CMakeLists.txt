INCLUDE ("${CMAKE_SOURCE_DIR}/cmake-common/sac2c-variables.cmake")
INCLUDE ("${CMAKE_SOURCE_DIR}/cmake-common/resolve-sac2c-dependencies.cmake")

# C files relative to this CMakeLists.txt.
SET (C_DEPS_SRC
    src/PGM/array2pgm.c
    src/PGM/pgm2array.c
    src/PPM/array2ppm.c
    src/PPM/ppm2array.c
)

# SaC files relative to this CMakeLists.txt.
SET (SAC_SRC
    PGM.sac
    PPM.sac
)

# For every C source, compile an object file maintaining the right
# location in the binary dir so that SaC files can pick it up.
FOREACH(name ${C_DEPS_SRC})
    SET (src "${CMAKE_CURRENT_SOURCE_DIR}/${name}")

    GET_FILENAME_COMPONENT (dir ${name} DIRECTORY)

    GET_FILENAME_COMPONENT (dst ${name} NAME_WE)
    SET (dst "${CMAKE_CURRENT_BINARY_DIR}/${dir}/${dst}${OBJEXT}")

    # Make sure that we put the object file in the same
    # location where the source file was.
    FILE (MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/${dir}")

    ADD_CUSTOM_COMMAND (
        OUTPUT "${dst}"
        IMPLICIT_DEPENDS C "${src}"
        MAIN_DEPENDENCY "${src}"
        COMMAND ${SAC2C} -v0 -noprelude -cc ccmod -o "${dst}" "${src}"
        COMMENT "Generating ${dst}")
ENDFOREACH(name)

# Make a directory for sac2c output
FILE (MAKE_DIRECTORY "${DLL_BUILD_DIR}/${TARGET_ENV}/${SBI}")

# For every SaC file, compile Tree and Mod files.
FOREACH (name ${SAC_SRC})
    SET (src "${CMAKE_CURRENT_SOURCE_DIR}/${name}")
    # sac2c requires computes objectfiles relatively to the working directory
    # of the call to sac2c.
    GET_FILENAME_COMPONENT (dir "${CMAKE_CURRENT_BINARY_DIR}/${name}" DIRECTORY)
    GET_FILENAME_COMPONENT (dst ${name} NAME_WE)

    SET (mod "${DLL_BUILD_DIR}/${TARGET_ENV}/${SBI}/lib${dst}Mod${VARIANT}${MODEXT}")
    SET (tree "${DLL_BUILD_DIR}/tree/${TARGET_ENV}/${SBI}/lib${dst}Tree${VARIANT}${TREE_DLLEXT}")

    RESOLVE_SAC_DEPS_AS_TARGETS ("${name}" "<TARGET>-module-<NAME>" target_list objs_list source_list)

    MESSAGE (STATUS "Computing dependencies for `${name}'")
    SET (src_deps)
    FOREACH (src_name ${source_list})
        MESSAGE (DEBUG "Searching for ${src_name} in source path list...")
        MATCH_NAME_IN_LIST ("${ALL_SAC_SRC}" "${src_name}" src_path)
        IF (src_path)
            MESSAGE (DEBUG "   found ${src_path}")
            LIST (APPEND src_deps "${CMAKE_CURRENT_SOURCE_DIR}/${src_path}")
        ELSE ()
            MESSAGE (FATAL_ERROR "Unable to find path to ${src_name} in sources!")
        ENDIF ()
    ENDFOREACH ()

    LIST (APPEND deps_list ${src_deps} ${objs_list} ${target_list})
    UNSET (src_deps)
    UNSET (objs_list)
    UNSET (source_list)
    UNSET (target_list)
    MESSAGE (DEBUG "For ${name} the computed dependences are: ${deps_list}")

    # Make sure that we have a directory we are changing to.
    FILE (MAKE_DIRECTORY "${dir}")

    ADD_CUSTOM_TARGET (${TARGET}-module-${dst} DEPENDS "${mod}" "${tree}")

    ADD_DEPENDENCIES (${TARGET}-all-modules ${TARGET}-module-${dst})

    ADD_CUSTOM_COMMAND (
        OUTPUT "${mod}" "${tree}"
        COMMAND ${SAC2C} -v0 -o ${DLL_BUILD_DIR} "${src}"
        WORKING_DIRECTORY "${dir}"
        MAIN_DEPENDENCY "${src}"
        DEPENDS ${deps_list}
        COMMENT "Building module `${dst}' for target `${TARGET}'")

    # Install compiled Tree/Mod parts of the compiled module
    # to the corresponding locations.
    INSTALL (
        FILES "${mod}"
        DESTINATION ${_install_mod_dir}/${TARGET_ENV}/${SBI}
        COMPONENT modules)
    INSTALL (
        FILES "${tree}"
        DESTINATION ${_install_tree_dir}/tree/${TARGET_ENV}/${SBI}
        COMPONENT trees)

    UNSET (deps_list)
ENDFOREACH (name)
