CMAKE_MINIMUM_REQUIRED(VERSION 2.8.4)
SET(CMAKE_SYSTEM_NAME Generic)

#============================================================================================
PROJECT(AVR)		  		# <-- Project Name

SET(FILES main.c)		  	# <-- Source File(s) (Seperated by Spaces)

SET(DEVICE atmega328p)			# <-- MCU.Go https://gcc.gnu.org/onlinedocs/gcc/AVR-Options.html to get a list of supported MCU
SET(FREQ 16000000)#Hz			# <-- MCU-frequency

SET(AVRBIN /usr/local/CrossPack-AVR/bin/)		# <-- Toolchain
SET(AVROBJCOPY avr-objcopy)		# <--     ""
SET(AVRSIZE avr-size)			# <--     ""
SET(AVRDUDE avrdude)			# <--     ""

SET(PROGRAMMER arduino)		# <-- Programmer Type. Go http://www.nongnu.org/avrdude/user-manual/avrdude_12.html for list of programmer Type
SET(PORT /dev/cu.usbmodem1411)		         	# <-- USB and interface
#In MacOS go to terminal and type "ls /dev/tty.*" (without quotes) For USB connection after connecting programmer
#In linux go to terminal and type "ls /dev/ttyACM*" (without quotes) if not found try "ls /dev/ttyACM*"(without quotes)
#Also a portfinder script is available https://startingelectronics.org/software/processing/find-arduino-port/
SET(BAUD 115200)	        	# <-- Baud Rate. Refer to datasheet

SET(EEPROM OFF)	     	        	# <-- Want to create an x.eep file? ON/OFF
SET(FLASH OFF)		        	# <-- Should be uploaded? ON/OFF

#============================================================================================

SET(CMAKE_C_COMPILER ${AVRBIN}/avr-gcc)
SET(CMAKE_CXX_COMPILER ${AVRBIN}/avr-g++)

SET(CMAKE_C_FLAGS  "-Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL -std=gnu99 -Wl,--gc-sections")
SET(CMAKE_CXX_FLAGS "-Os -mmcu=${DEVICE} -DF_CPU=${FREQ}UL -Wl,--gc-sections")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")
set(CMAKE_PREFIX_PATH /opt/local/avr)
set(CMAKE_EXECUTABLE_SUFFIX ".elf")

include_directories( /usr/local/usr/local/CrossPack-AVR/include)

SET(SOURCE_FILES ${FILES})

#============================================================================================

ADD_EXECUTABLE(${CMAKE_PROJECT_NAME} main.c)

ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRBIN}/${AVROBJCOPY} -O ihex -R .eeprom ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.elf ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex)

if(EEPROM)
    ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRBIN}/${AVROBJCOPY} -O ihex -j .eeprom --set-section-flags=.eeprom="alloc,load"  --change-section-lma .eeprom=0 ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.elf ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.eep)
endif(EEPROM)

ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRBIN}/${AVRSIZE} --target=ihex ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex)

if(FLASH)
    ADD_CUSTOM_COMMAND(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD COMMAND ${AVRBIN}/${AVRDUDE} -p${DEVICE} -c${PROGRAMMER} -P${PORT} -b${BAUD} -U flash:w:${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${CMAKE_PROJECT_NAME}.hex)
endif(FLASH)
