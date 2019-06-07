file( GLOB rg-files "rg/*.rg" )

set( midis "" )
set( oggs  "" )
set( tunes "" )

foreach( rg-file ${rg-files} )
    get_filename_component( stem "${rg-file}" NAME_WE )
    set( midis ${midis} ${CMAKE_CURRENT_SOURCE_DIR}/midi/${stem}.mid )
    set( oggs  ${oggs}  ${CMAKE_CURRENT_SOURCE_DIR}/ogg/${stem}.ogg )
    set( tunes ${tunes} ${stem} )
endforeach()

set( sf $ENV{HOME}/dev/sound/sf2/musyng-kite.sf2 )
if( NOT EXISTS ${sf} )
    set( sf /usr/share/sounds/sf2/FluidR3_GM.sf2 )
    if( NOT EXISTS ${sf} )
        message( FATAL_ERROR "Cannot find any soundfonts." )
    endif()
endif()
message( STATUS "Using sound font: ${sf}" )

set( sample-rate 44100 )

# Note: these commands operates in the source directory instead
# of the binary one. Also, the rg2midi command being run here ac-
# tually refers to a CMake target in the rg-2-midi submodule.
# That target will be added as a dependency automatically to any
# target using this command.
macro( create_custom_conversion_commands tune )
    set( rg   "rg/${tune}.rg" )
    set( midi "midi/${tune}.mid" )
    set( ogg  "ogg/${tune}.ogg" )
    # rg --> midi
    add_custom_command(
        OUTPUT  "${CMAKE_CURRENT_SOURCE_DIR}/${midi}"
        DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/${rg}"
        COMMAND rg2midi ${rg} ${midi}
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT "Exporting ${rg} to MIDI"
    )
    # midi --> ogg
    add_custom_command(
        OUTPUT  "${CMAKE_CURRENT_SOURCE_DIR}/${ogg}"
        DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/${midi}"
        COMMAND fluidsynth -nli -r ${sample-rate} -T oga
                -F    ${CMAKE_CURRENT_SOURCE_DIR}/${ogg}
                ${sf} ${CMAKE_CURRENT_SOURCE_DIR}/${midi}
                >/dev/null 2>&1
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT "Rendering ${midi} to OGG"
    )
endmacro()

foreach( tune ${tunes} )
    create_custom_conversion_commands( ${tune} )
endforeach()

# The `rn` target will depend on these, and these will in turn
# cause the custom commands to be run for each midi file, pos-
# sibly in parallel.
add_custom_target( midi-convert DEPENDS ${midis} )
add_custom_target( ogg-convert  DEPENDS ${oggs}  )
# Need this otherwise a midi-convert command may be run multiple
# times in parallel.
add_dependencies( ogg-convert midi-convert )
