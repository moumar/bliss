cmake_minimum_required(VERSION 2.8)

project(bliss C)

find_package(PkgConfig REQUIRED)

pkg_check_modules(MULTIMEDIA REQUIRED libavformat libavutil libavcodec fftw3)

pkg_check_modules(RESAMPLE REQUIRED libavresample)

include_directories(${MULTIMEDIA_INCLUDE_DIRS} ${RESAMPLE_INCLUDE_DIRS} include/)
link_directories(${MULTIMEDIA_LIBRARY_DIRS} ${RESAMPLE_LIBRARY_DIRS})
add_definitions(${MULTIMEDIA_CFLAGS_OTHER} ${RESAMPLE_CFLAGS_OTHER})
add_definitions(-Wall -Wno-long-long -pedantic -std=c99)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")

#####################################
# Bliss target to build the library #
#####################################

add_library(bliss SHARED src/amplitude_sort.c src/decode_av.c src/tempo_atk_sort.c src/frequency_sort.c src/helpers.c src/analyze.c)

target_link_libraries(bliss
    ${MULTIMEDIA_LIBRARIES}
	${RESAMPLE_LIBRARIES}
    m)

#############################
# Examples building targets #
#############################
add_executable(analyze
    examples/analyze.c)
target_link_libraries(analyze bliss)

add_executable(distance
    examples/distance.c)
target_link_libraries(distance bliss)

add_executable(detect-gapless
	examples/detect-gapless.c)
target_link_libraries(detect-gapless bliss)

add_custom_target(examples
    DEPENDS analyze distance detect-gapless)


##############################
# Unittests building targets #
##############################
enable_testing()
add_executable(test_analyze
    tests/test_analyze.c)
target_link_libraries(test_analyze bliss)

add_test(ctest_build_test_analyze "${CMAKE_COMMAND}" --build ${CMAKE_BINARY_DIR} --target test_analyze)
add_test(ctest_run_test_analyze test_analyze)
set_tests_properties(ctest_run_test_analyze PROPERTIES DEPENDS ctest_build_test_analyze)


###################
# Install targets #
###################
install(TARGETS bliss DESTINATION /usr/lib)
install(FILES include/bliss.h DESTINATION /usr/include)

##############
# Generators #
##############

set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_NAME "bliss-music")
set(CPACK_DEBIAN_PACKAGE_VERSION "1.0")
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libavutil-dev, libavformat-dev, libavcodec-dev, libavresample-dev")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "polochonstreet@gmx.fr")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "A music analyzis library")
include(CPack)
