CMAKE_MINIMUM_REQUIRED(VERSION 2.8.4)
SET(CMAKE_SYSTEM_NAME Generic)
#============================================================================================
PROJECT(KeyPad)                    # <-- ProjectName

SET(FILES main.c)		  	    # <-- Project Files

SET(DEVICE ATmega16)			# <-- MCU
SET(FREQ 8000000)			    # <-- MCU-Frequenz
set(FUSE 0x04)                  # <-- fuse bits

SET(AVRBIN
        /usr/local/Cellar/avr-gcc@9/9.3.0_3/bin)    # <-- Toolchain

SET(AVRGCC avr-gcc)             # <--     ""
SET(AVROBJCOPY avr-objcopy)		# <--     ""
SET(AVRSIZE avr-size)			# <--     ""
SET(AVRDUDE avrdude)            # <--     ""
SET(AVRSTRIP avr-strip)         # <--     ""
SET(PROGRAMMER usbasp)		    # <-- Programmer Type
SET(PORT usb)		         	# <-- Port Type/name   ex: /dev/cu.00000000
SET(BAUD 57600)	        	    # <-- Baudrate

SET(COMPILE_FLAGS -Wall -Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL)   # <--    "c flags"


SET(EEPROM yes)	     	        # <-- set yes if you want .eep file !
SET(FLASH no)		        	# <-- this command is for program hardware !!! set it no if you dont wanna program dvice !

#============================================================================================

SET(CMAKE_C_COMPILER ${AVRBIN}/avr-gcc)
SET(CMAKE_CXX_COMPILER ${AVRBIN}/avr-g++)

SET(CMAKE_C_FLAGS  "-Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL -std=gnu99 -Wl,--gc-sections")
SET(CMAKE_CXX_FLAGS "-Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL -Wl,--gc-sections")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin/out_put")

SET(OUTP ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

set(CMAKE_PREFIX_PATH /opt/local/avr)
set(CMAKE_EXECUTABLE_SUFFIX ".elf")



include_directories( /opt/local/avr/include)

SET(SOURCE_FILES ${FILES})
#============================================================================================
FILE(MAKE_DIRECTORY ${OUTP})

ADD_EXECUTABLE(${CMAKE_PROJECT_NAME} ${FILES} )

#
add_custom_target(strip ALL     ${AVRSTRIP} "${OUTP}/${PROJECT_NAME}.elf" DEPENDS ${PROJECT_NAME})
add_custom_target(hex   ALL     ${AVROBJCOPY} -R .eeprom -O ihex "${OUTP}/${PROJECT_NAME}.elf" "${OUTP}/${PROJECT_NAME}.hex" DEPENDS strip)
add_custom_target(eeprom        ${AVROBJCOPY} -j .eeprom --change-section-lma .eeprom=0 -O ihex "${OUTP}/${PROJECT_NAME}.elf" "${OUTP}/${PROJECT_NAME}.eep" DEPENDS strip)
add_custom_target(flash         ${AVRDUDE} -c ${PROGRAMMER} -p ${DEVICE} -P ${PORT} -b ${BAUD} -U lfuse:w:${FUSE}:m -U flash:w:${OUTP}/${PROJECT_NAME}.hex DEPENDS hex)
#

set_directory_properties(PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex"
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.eep"
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.lst"
        )
