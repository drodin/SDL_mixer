cmake_minimum_required(VERSION 3.2)

option(MUSIC_CMD "Supports an external command for playing music" ON)
option(MUSIC_FLAC "Support loading FLAC music with libFLAC" ON)
# option(MUSIC_MID_FLUIDSYNTH "Support FluidSynth" OFF)
# option(MUSIC_MP3_MAD "Support loading MP3 music via MAD" OFF)
# option(MUSIC_MOD_MIKMOD "Support mikmod" OFF)
# option(MUSIC_MOD_MODPLUG "Support loading MOD music via modplug" OFF)
# option(MUSIC_MP3_MPG123 "Support loading MP3 music via MPG123" OFF)
option(MUSIC_MID_NATIVE "Support native midi" OFF)
option(MUSIC_OGG "Support loading OGG Vorbis music" ON)
option(MUSIC_OPUS "Support Opus" ON)
option(MUSIC_MID_TIMIDITY "Support TiMidity" ON)
option(MUSIC_WAV "Support streaming WAV files" ON)

include(cmake/HunterGate.cmake)
HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.266.tar.gz"
    SHA1 "7e6dbe7aafb88a7ca4d3f84040797dd52c468945"
)

project(SDL_mixer VERSION 2.0.4)

add_library(${PROJECT_NAME})

hunter_add_package(SDL2)
find_package(SDL2 CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE SDL2::SDL2)

set(SRC_FILES
    effect_position.c
    effect_stereoreverse.c
    effects_internal.c
    load_aiff.c
    load_voc.c
    mixer.c
    music.c
)

if(MUSIC_CMD AND NOT WIN32)
	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_CMD)
	list(APPEND SRC_FILES music_cmd.c)
endif()

if(MUSIC_FLAC)
	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_FLAC)
	list(APPEND SRC_FILES music_flac.c)
	hunter_add_package(FLAC)
	find_package(FLAC CONFIG REQUIRED)
	target_link_libraries(${PROJECT_NAME} PRIVATE FLAC::FLAC)
endif()

# if(MUSIC_MID_FLUIDSYNTH)
# 	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_MID_FLUIDSYNTH)
# 	list(APPEND SRC_FILES music_fluidsynth.c)
# endif()

# if(MUSIC_MP3_MAD)
# 	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_MP3_MAD)
# 	list(APPEND SRC_FILES music_mad.c)
# endif()

# if(MUSIC_MOD_MIKMOD)
# 	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_MOD_MIKMOD)
# 	list(APPEND SRC_FILES music_mikmod.c)
# endif()

# if(MUSIC_MOD_MODPLUG)
# 	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_MOD_MODPLUG)
# 	list(APPEND SRC_FILES music_modplug.c)
# endif()

# if(MUSIC_MP3_MPG123)
# 	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_MP3_MPG123)
# 	list(APPEND SRC_FILES music_mpg123.c)
# endif()

if(MUSIC_MID_NATIVE)
	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_MID_NATIVE)
	list(APPEND SRC_FILES
		music_nativemidi.c
		native_midi/native_midi_common.c
	)
	if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
		list(APPEND SRC_FILES native_midi/native_midi_macosx.c)
	endif()
	if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
		list(APPEND SRC_FILES native_midi/native_midi_win32.c)
	endif()
endif()

if(MUSIC_OGG)
	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_OGG)
	list(APPEND SRC_FILES music_ogg.c)
	hunter_add_package(vorbis)
	find_package(vorbis CONFIG REQUIRED)
	target_link_libraries(${PROJECT_NAME} PRIVATE vorbis::vorbisfile)
endif()

if(MUSIC_OPUS)
	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_OPUS)
	list(APPEND SRC_FILES music_opus.c)
	hunter_add_package(opusfile)
	find_package(opusfile CONFIG REQUIRED)
	target_link_libraries(${PROJECT_NAME} PRIVATE opusfile::opusfile)
endif()

if(MUSIC_MID_TIMIDITY)
	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_MID_TIMIDITY)
	list(APPEND SRC_FILES
		music_timidity.c
		timidity/common.c
		timidity/instrum.c
		timidity/mix.c
		timidity/output.c
		timidity/playmidi.c
		timidity/readmidi.c
		timidity/resample.c
		timidity/tables.c
		timidity/timidity.c
	)
endif()

if(MUSIC_WAV)
	target_compile_definitions(${PROJECT_NAME} PRIVATE MUSIC_WAV)
	list(APPEND SRC_FILES music_wav.c)
endif()


target_sources(${PROJECT_NAME} PRIVATE ${SRC_FILES})


include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

install(TARGETS ${PROJECT_NAME}
	EXPORT "${targets_export_name}"
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR}
	INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    FILES SDL_mixer.h
    DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}/SDL2
)

write_basic_package_version_file(
	"${version_config}" COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
	"cmake/template/Config.cmake.in"
	"${project_config}"
	INSTALL_DESTINATION "${config_install_dir}"
)

install(
	FILES "${project_config}" "${version_config}"
	DESTINATION "${config_install_dir}"
)

install(
	EXPORT "${targets_export_name}"
	NAMESPACE "${namespace}"
	DESTINATION "${config_install_dir}"
)
