CMAKE_MINIMUM_REQUIRED(VERSION 2.8.4)
SET(CMAKE_SYSTEM_NAME Generic)

#============================================================================================
PROJECT(avr)  # <-- ProjectName

SET(FILES main.c)		  	    # <-- Project Files

SET(DEVICE atmega16)			# <-- MCU
SET(FREQ 16000000)			    # <-- MCU-Frequenz

SET(AVRBIN
        /usr/local/Cellar/avr-gcc@9/9.3.0_3/bin)    # <-- Toolchain

SET(AVRGCC avr-gcc)             # <--     ""
SET(AVROBJCOPY avr-objcopy)		# <--     ""
SET(AVRSIZE avr-size)			# <--     ""
SET(AVRDUDE avrdude)			# <--     ""

SET(PROGRAMMER avrispmkii)		# <-- Programmer Type
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

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")

SET(OUTP ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/out_put)

#add_subdirectory(bin)


set(CMAKE_PREFIX_PATH /opt/local/avr)
set(CMAKE_EXECUTABLE_SUFFIX ".elf")

set_property(DIRECTORY PROPERTY ADDITIONAL_MAKE_CLEAN_FILES
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex"
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.eep"
        )

include_directories( /opt/local/avr/include)

SET(SOURCE_FILES ${FILES})

#============================================================================================

ADD_EXECUTABLE(${CMAKE_PROJECT_NAME} ${FILES})

file(MAKE_DIRECTORY ${OUTP})

ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRGCC} ARGS ${COMPILE_FLAGS} -c ${CMAKE_CURRENT_SOURCE_DIR}/${FILES} -o ${OUTP}/${CMAKE_PROJECT_NAME}.o)
ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRGCC} ARGS ${COMPILE_FLAGS} -o ${OUTP}/${CMAKE_PROJECT_NAME}.elf ${OUTP}/${CMAKE_PROJECT_NAME}.o)
ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVROBJCOPY} ARGS -O ihex ${OUTP}/${CMAKE_PROJECT_NAME}.elf ${OUTP}/${CMAKE_PROJECT_NAME}.hex)


ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVROBJCOPY} ARGS -O ihex ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.elf ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex)

if(EEPROM)
    ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVROBJCOPY} ARGS -O ihex -j .eeprom --set-section-flags=.eeprom="alloc,load"  --change-section-lma .eeprom=0 ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.elf ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.eep)
endif(EEPROM)

ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRSIZE} ARGS --target=ihex ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex)

if(FLASH)
    ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRDUDE} ARGS -p ${DEVICE} -c ${PROGRAMMER} -P ${PORT} -b ${BAUD} -U flash:w:${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex)
endif(FLASH)
