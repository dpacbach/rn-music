set( TUNES
  fife-and-drum-1
)

if( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" )
    set( rg2midi_platform "linux64-mint-19.1-gcc7.3" )
elseif( ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin" )
    set( rg2midi_platform "darwin-osx-10.14-clang-4.2.1" )
else()
    message( FATAL "Unsupported system name: ${CMAKE_SYSTEM_NAME}" )
endif()

set( rg2m_bin "rg2midi/rg2midi.${rg2midi_platform}" )
message( STATUS "Using rg2midi binary: ${rg2m_bin}" )

# Note: this command operates in the source directory instead of
# the binary one.
macro( create_custom_conversion_command tune )
    set( rg   "rg/${tune}.rg" )
    set( midi "midi/${tune}.mid" )
    add_custom_command(
        OUTPUT  "${CMAKE_CURRENT_SOURCE_DIR}/${midi}"
        DEPENDS "${rg2m_bin}" "${CMAKE_CURRENT_SOURCE_DIR}/${rg}"
        COMMAND ${rg2m_bin} ${rg} ${midi}
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
