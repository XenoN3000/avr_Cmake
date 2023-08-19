CMAKE_MINIMUM_REQUIRED(VERSION 2.8.4)
#============================================================================================
PROJECT(KeyPad C CXX)                    # <-- ProjectName

SET(CMAKE_TOOLCHAIN_FILE AVRBIN)



SET(FILES main.c)		  	    # <-- Project Files

SET(DEVICE ATmega16)			# <-- MCU
SET(FREQ 8000000)			    # <-- MCU-Frequenz
set(FUSE 0x04)                  # <-- fuse bits



SET(AVRBIN
        "/usr/local/Cellar/avr-gcc@9/9.3.0_3/bin/")    # <-- Toolchain


SET(AVRCPP ${AVRBIN}avr-g++)             # <--     ""
SET(AVRGCC ${AVRBIN}avr-gcc)             # <--     ""
SET(AVROBJCOPY avr-objcopy)		# <--     ""
SET(AVRSIZE avr-size)			# <--     ""
SET(AVRDUDE avrdude)            # <--     ""
SET(AVRSTRIP avr-strip)         # <--     ""
SET(PROGRAMMER usbasp)		    # <-- Programmer Type
SET(PORT usb)		         	# <-- Port Type/name   ex: /dev/cu.00000000
SET(BAUD 57600)	        	    # <-- Baudrate



enable_language(CXX)
enable_language(C)




SET(COMPILE_FLAGS -Wall -Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL)   # <--    "c flags"


SET(EEPROM yes)	     	        # <-- set yes if you want .eep file !
SET(FLASH no)		        	# <-- this command is for program hardware !!! set it no if you dont wanna program dvice !

#============================================================================================

SET(CMAKE_C_COMPILER ${AVRGCC})
SET(CMAKE_ASM_COMPILER ${AVRGCC})
SET(CMAKE_CXX_COMPILER ${AVRCPP})

SET(CMAKE_C_FLAGS  "-Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL -std=gnu99 -Wl,--gc-sections")
SET(CMAKE_CXX_FLAGS "-Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL -Wl,--gc-sections")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin/out_put")
SET(CMAKE_BUILD_ "${CMAKE_CURRENT_SOURCE_DIR}/build")

SET(OUTP ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

set(CMAKE_CONFIGURATION_TYPES "AVR;Flash" CACHE STRING "" FORCE )

set(CMAKE_PREFIX_PATH /opt/local/avr)
set(CMAKE_EXECUTABLE_SUFFIX ".elf")



include_directories( /opt/local/avr/include)

SET(SOURCE_FILES ${FILES})
#============================================================================================
FILE(MAKE_DIRECTORY ${OUTP})


ADD_EXECUTABLE(${CMAKE_PROJECT_NAME} ${FILES} )

#
add_custom_target(strip ALL     ${AVRSTRIP} "${OUTP}/${CMAKE_PROJECT_NAME}.elf" DEPENDS ${CMAKE_PROJECT_NAME})
add_custom_target(hex   ALL     ${AVROBJCOPY} -R .eeprom -O ihex "${OUTP}/${CMAKE_PROJECT_NAME}.elf" "${OUTP}/${PROJECT_NAME}.hex" DEPENDS strip)
add_custom_target(eeprom        ${AVROBJCOPY} -j .eeprom --change-section-lma .eeprom=0 -O ihex "${OUTP}/${PROJECT_NAME}.elf" "${OUTP}/${CMAKE_PROJECT_NAME}.eep" DEPENDS strip)
add_custom_target(flash         ${AVRDUDE} -c ${PROGRAMMER} -p ${DEVICE} -P ${PORT} -b ${BAUD} -U lfuse:w:${FUSE}:m -U flash:w:${OUTP}/${CMAKE_PROJECT_NAME}.hex DEPENDS hex)
#


if (CMAKE_BUILD_TYPE MATCHES Flash)
    ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRDUDE} ARGS -p ${DEVICE} -c ${PROGRAMMER} -P ${PORT} -b ${BAUD} -U lfuse:w:${FUSE}:m -U flash:w:${OUTP}/${CMAKE_PROJECT_NAME}.hex)
endif ()

set_directory_properties(PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
        "${OUTP}/${CMAKE_PROJECT_NAME}.hex"
        "${OUTP}/${CMAKE_PROJECT_NAME}.eep"
        "${OUTP}/${CMAKE_PROJECT_NAME}.lst"
        )
