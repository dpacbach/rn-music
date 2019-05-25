set( TUNES
  fife-and-drum-1
)

set( RG_BINARY "rg2midi/rg2midi.linux64-mint-19.1-gcc7.3" )

# Note: this command operates in the source directory instead of
# the binary one.
macro( create_custom_conversion_command tune )
    set( rg   "rg/${tune}.rg" )
    set( midi "midi/${tune}.mid" )
    add_custom_command(
        OUTPUT  "${CMAKE_CURRENT_SOURCE_DIR}/${midi}"
        DEPENDS "${RG_BINARY}" "${CMAKE_CURRENT_SOURCE_DIR}/${rg}"
        COMMAND ${RG_BINARY} ${rg} ${midi}
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT "Export ${tune}.rg to MIDI"
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
add_custom_target( midi_convert DEPENDS ${all_midi} )
