cmake_minimum_required(VERSION 3.31)

project(mercury
	VERSION 1.99
	LANGUAGES C)

# Options
option(USE_ALSA_BACKEND "Use the alsa audio backend for ffaudio" ON)
option(USE_PULSE_BACKEND "Use the pulseaudio audio backend for ffaudio" OFF)
option(USE_JACK_BACKEND "Use the jack audio backend for ffaudio" OFF)

if(NOT (USE_ALSA_BACKEND OR USE_PULSE_BACKEND OR USE_JACK_BACKEND))
	message(FATAL_ERROR "You need to choose at least one of the following audio backends: alsa, pulse or jack.")
endif()

set(SOURCES
	src/main.c
	src/audioio/audioio.c
	src/audioio/audioio.h
	src/audioio/resampler.c
	src/audioio/resampler.h
	src/audioio/std.h
	src/common/cfg_utils.c
	src/common/cfg_utils.h
	src/common/chan.c
	src/common/chan.h
	src/common/crc6.c
	src/common/crc6.h
	src/common/defines_modem.h
	src/common/hermes_log.c
	src/common/hermes_log.h
	src/common/os_interop.c
	src/common/os_interop.h
	src/common/queue.c
	src/common/queue.h
	src/common/ring_buffer_posix.c
	src/common/ring_buffer_posix.h
	src/common/shm_posix.c
	src/common/shm_posix.h
	src/common/watterson.c
	src/common/watterson.h
	src/data_interfaces/net.c
	src/data_interfaces/net.h
	src/data_interfaces/tcp_interfaces.c
	src/data_interfaces/tcp_interfaces.h
	src/datalink_arq/arith.c
	src/datalink_arq/arq.c
	src/datalink_arq/arq_channels.c
	src/datalink_arq/arq_channels.h
	src/datalink_arq/arq_events.h
	src/datalink_arq/arq_fsm.c
	src/datalink_arq/arq_fsm.h
	src/datalink_arq/arq.h
	src/datalink_arq/arq_modem.c
	src/datalink_arq/arq_modem.h
	src/datalink_arq/arq_protocol.c
	src/datalink_arq/arq_protocol.h
	src/datalink_arq/arq_timing.c
	src/datalink_arq/arq_timing.h
	src/datalink_broadcast/broadcast.c
	src/datalink_broadcast/broadcast.h
	src/datalink_broadcast/kiss.c
	src/datalink_broadcast/kiss.h
	src/gui_interface/ui_communication.c
	src/gui_interface/ui_communication.h
	src/gui_interface/websocket/mercury_websocket.c
	src/gui_interface/websocket/mercury_websocket.h
	src/modem/framer.c
	src/modem/framer.h
	src/modem/modem.c
	src/modem/modem.h
	src/radio_io/radio_cmds.h
	src/radio_io/radio_io.c
	src/radio_io/radio_io.h
	src/radio_io/rigctl_parse.c
	src/radio_io/rigctl_parse.h
	src/radio_io/sbitx_io.c
	src/radio_io/sbitx_io.h
	src/radio_io/shm_utils.c
	src/radio_io/shm_utils.h
	src/utils/broadcast_diag_rx.c
	src/utils/broadcast_diag_tx.c
	src/utils/broadcast_test.c
	src/utils/loopsim/noisebridge.c
	src/utils/watterson_test.c
)

# Prepare third party libraries

# Prepare ffaudio
add_subdirectory(third_party/ffaudio)

# Prepare freedv
add_subdirectory(third_party/freedv)

# Prepare iniparser
add_subdirectory(third_party/iniparser)

# Prepare moongoose
add_subdirectory(third_party/moongoose)

# Prepare hamlib for Windows
if(WIN32)
	add_subdirectory(third_party/hamlib-w64)
endif()

# Create the main target
add_executable(${PROJECT_NAME} ${SOURCES})

# Link libraries

target_link_libraries(${PROJECT_NAME} PRIVATE ffaudio freedv iniparser mongoose)

# Link to libm if using Linux for including <math.h>
if(LINUX)
	target_link_libraries(${PROJECT_NAME} PRIVATE m)
endif()

# Link to ws2_32 if using Windows for including <winsock2.h>
# Link to local vendored hamlib
if(WIN32)
	target_link_libraries(${PROJECT_NAME} PRIVATE ws2_32)
	target_link_libraries(${PROJECT_NAME} PRIVATE hamlib-w64)
endif()

# Install rules
if(LINUX)
	include(GNUInstallDirs)

	install(TARGETS ${PROJECT_NAME}
		DESTINATION ${CMAKE_INSTALL_BINDIR}
	)

	install(FILES ${PROJECT_SOURCE_DIR}/resources/mercury.1
		DESTINATION ${CMAKE_INSTALL_MANDIR}/man1
	)

	install(FILES ${PROJECT_SOURCE_DIR}/resources/icon-512.svg
		DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/icons/hicolor/scalable/apps
		RENAME ${PROJECT_NAME}.svg
	)
endif()