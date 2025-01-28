INCLUDE ("${CMAKE_SOURCE_DIR}/cmake-common/sac2c-variables.cmake")

# C files
SET (C_DEPS_SRC
    src/PGM/array2pgm.c
    src/PGM/pgm2array.c
    src/PPM/array2ppm.c
    src/PPM/ppm2array.c
)

# SaC files
SET (SAC_SRC
    PGM.sac
    PPM.sac
)

# For every C source, compile an object file maintaining the right
# location in the binary directory so that SaC can pick it up
FOREACH (name ${C_DEPS_SRC})
    SET (src "${CMAKE_CURRENT_SOURCE_DIR}/${name}")

    GET_FILENAME_COMPONENT (dir ${name} DIRECTORY)

    GET_FILENAME_COMPONENT (dst ${name} NAME_WE)
    SET (dst "${CMAKE_CURRENT_BINARY_DIR}/${dir}/${dst}${OBJEXT}")

    # Put the object file in the same location as the source file
    FILE (MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/${dir}")

    ADD_CUSTOM_COMMAND (
        OUTPUT "${dst}"
        IMPLICIT_DEPENDS C "${src}"
        MAIN_DEPENDENCY "${src}"
        COMMAND ${SAC2C} -v0 -noprelude -cc ccmod -o "${dst}" "${src}"
        COMMENT "Generating ${dst} from ${src}")
ENDFOREACH (name)

# Make a directory for sac2c output
FILE (MAKE_DIRECTORY "${DLL_BUILD_DIR}/${TARGET_ENV}/${SBI}")

# For every SaC file, compile Tree and Mod files.
