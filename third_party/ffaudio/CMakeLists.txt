if(APPLE)
    set(BACKENDS
        ffaudio/coreaudio.c
    )
elseif(WIN32)
    set(BACKENDS
        ffaudio/dsound.c
        ffaudio/wasapi.c
    )
elseif(ANDROID)
    set(BACKENDS
        ffaudio/aaudio.c
    )
elseif(LINUX)
    if(USE_ALSA_BACKEND)
        cmake_pkg_config(IMPORT alsa REQUIRED)

        list(APPEND BACKENDS ffaudio/alsa.c)
    endif()

    if(USE_PULSE_BACKEND)
        cmake_pkg_config(IMPORT libpulse REQUIRED)

        list(APPEND BACKENDS ffaudio/pulse.c)
    endif()

    if(USE_JACK_BACKEND)
        cmake_pkg_config(IMPORT jack REQUIRED)

        list(APPEND BACKENDS ffaudio/jack.c)
    endif()
endif()

add_library(ffaudio STATIC
    # ffbase
    ffbase/args.h
    ffbase/atomic.h
    ffbase/base.h
    ffbase/chain.h
    ffbase/conf.h
    ffbase/conf.mk
    ffbase/cpuid.h
    ffbase/crc32.h
    ffbase/fntree.h
    ffbase/json.h
    ffbase/json-scheme.h
    ffbase/json-writer.h
    ffbase/list.h
    ffbase/lock.h
    ffbase/map.h
    ffbase/mem-print.h
    ffbase/murmurhash3.h
    ffbase/rbtree.h
    ffbase/ring.h
    ffbase/ringqueue.h
    ffbase/slice.h
    ffbase/sort.c
    ffbase/sort.h
    ffbase/sort-insertion.h
    ffbase/sort-merge.h
    ffbase/str-convert.h
    ffbase/stream.h
    ffbase/str-format.c
    ffbase/str-format.h
    ffbase/string.h
    ffbase/stringz.h
    ffbase/time.h
    ffbase/unicode.h
    ffbase/vector.h
    # ffaudio
    ${BACKENDS}
    ffaudio/audio.h
    ffaudio/pcm-convert.h
    ffaudio/pcm-gain.h
    ffaudio/pcm.h
    ffaudio/util.h
)

if(LINUX AND NOT ANDROID)
    if(USE_ALSA_BACKEND)
        target_link_libraries(ffaudio PUBLIC PkgConfig::alsa)
    endif()

    if(USE_PULSE_BACKEND)
        target_link_libraries(ffaudio PUBLIC PkgConfig::libpulse)
    endif()

    if(USE_JACK_BACKEND)
        target_link_libraries(ffaudio PUBLIC PkgConfig::jack)
    endif()
endif()

target_compile_definitions(ffaudio PRIVATE FFBASE_OPT_SIZE)

target_include_directories(ffaudio PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})