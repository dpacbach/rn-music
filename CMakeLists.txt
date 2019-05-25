set( TUNES
  fife-and-drum-1
)

# Note: this command operates in the source directory instead of
# the binary one. Also, the command being run here (`rg2midi`)
# actually refers to a CMake target in the rg-2-midi submodule.
# That target will be added as a dependency automatically to any
# target using this command.
macro( create_custom_conversion_command tune )
    set( rg   "rg/${tune}.rg" )
    set( midi "midi/${tune}.mid" )
    add_custom_command(
        OUTPUT  "${CMAKE_CURRENT_SOURCE_DIR}/${midi}"
        DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/${rg}"
        COMMAND rg2midi ${rg} ${midi}
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT "Exporting ${rg} to MIDI"
    )
endmacro()

set( all_midi "" )

foreach( tune ${TUNES} )
    create_custom_conversion_command( ${tune} )
    set( all_midi ${all_midi} ${CMAKE_CURRENT_SOURCE_DIR}/midi/${tune}.mid )
endforeach()

# The `rn` target will depend on this one, and this one in turn
# will cause a custom command to be run for each midi file, pos-
# sibly in parallel.
add_custom_target( midi-convert DEPENDS ${all_midi} )

# === Subdirectories ==============================================

# Add only the src folder because the top-level CMake file is
# only for standalone builds of that project.
add_subdirectory( rg-2-midi/src )
