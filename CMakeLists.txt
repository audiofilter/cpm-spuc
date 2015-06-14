CMAKE_MINIMUM_REQUIRED(VERSION 2.8)


#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME spuc)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    message(STATUS "Cloning repo (https://github.com/iauns/cpm)")
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_QUIET ERROR_QUIET)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()


# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

# Include CPM modules or externals here (with CPM_AddModule).
CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# Spuc
#------------------------------------------------------------------------------
# 
set(SPUC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/spuc)
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${SPUC_DIR}
  GIT_REPOSITORY "https://github.com/audiofilter/spuclib.git"
  GIT_TAG 1.0.0
  USE_CACHING TRUE
  )

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
set(CMAKE_CXX_FLAGS " -std=c++11 ")
message(STATUS "Using c++11")
ENDIF()

execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion
                OUTPUT_VARIABLE GCC_VERSION)
if (GCC_VERSION VERSION_GREATER 4.8 OR GCC_VERSION VERSION_EQUAL 4.8)
    message(STATUS "GCC Version >=  4.8, using c++11/libstdc++")
	set(CMAKE_EXE_LINKER_FLAGS "-static-libstdc++")
	set(CMAKE_CXX_FLAGS "-Wall -std=c++0x ")
endif()

SET(TYPE_SRCS 
${SPUC_DIR}/spuc/bit.cpp
)

SET(FILT_SRCS  
${SPUC_DIR}/spuc/butterworth_iir.cpp
${SPUC_DIR}/spuc/butterworth_allpass.cpp
${SPUC_DIR}/spuc/chebyshev_iir.cpp
${SPUC_DIR}/spuc/elliptic_iir.cpp
${SPUC_DIR}/spuc/elliptic_allpass.cpp
${SPUC_DIR}/spuc/allpass_1aq1.cpp
${SPUC_DIR}/spuc/allpass_1aq2.cpp
${SPUC_DIR}/spuc/butterworth_fir.cpp
${SPUC_DIR}/spuc/create_remez_fir.cpp
${SPUC_DIR}/spuc/create_remez_lpfir.cpp
${SPUC_DIR}/spuc/farrow_upsampler.cpp
${SPUC_DIR}/spuc/find_roots.cpp
${SPUC_DIR}/spuc/fir_coeff.cpp
${SPUC_DIR}/spuc/fir_inv_dft.cpp
${SPUC_DIR}/spuc/gaussian_fir.cpp
${SPUC_DIR}/spuc/iir_coeff.cpp
${SPUC_DIR}/spuc/raised_cosine_imp.cpp
${SPUC_DIR}/spuc/root_raised_cosine_imp.cpp
${SPUC_DIR}/spuc/root_raised_cosine_quantized.cpp
${SPUC_DIR}/spuc/remez_fir.cpp
${SPUC_DIR}/spuc/spuc_math.cpp
${SPUC_DIR}/spuc/shelf_allpass1.cpp
${SPUC_DIR}/spuc/other_freq.cpp
${SPUC_DIR}/spuc/equalizer.cpp
${SPUC_DIR}/spuc/fast_equalizer.cpp
)

SET(SPUC_SRCS 
${SPUC_DIR}/spuc/bit_scrambler.cpp
${SPUC_DIR}/spuc/cfft.cpp
${SPUC_DIR}/spuc/idft.cpp
${SPUC_DIR}/spuc/fft.cpp
${SPUC_DIR}/spuc/ifft.cpp
${SPUC_DIR}/spuc/max_pn.cpp
${SPUC_DIR}/spuc/misc.cpp
${SPUC_DIR}/spuc/nco.cpp
${SPUC_DIR}/spuc/noise.cpp
${SPUC_DIR}/spuc/rv_stat.cpp
${SPUC_DIR}/spuc/sigma_delta.cpp
${SPUC_DIR}/spuc/timing_nco.cpp
${SPUC_DIR}/spuc/vco.cpp
${SPUC_DIR}/spuc/window.cpp
${SPUC_DIR}/spuc/utils.cpp
)


SET(COMM_SRCS 
${SPUC_DIR}/spuc/a_d.cpp
${SPUC_DIR}/spuc/agc.cpp
${SPUC_DIR}/spuc/bpsk_ber_test.cpp
${SPUC_DIR}/spuc/carrier_nco.cpp
${SPUC_DIR}/spuc/data_conv_encoder.cpp
${SPUC_DIR}/spuc/dqpsk.cpp
${SPUC_DIR}/spuc/dvb_conv_encoder.cpp
${SPUC_DIR}/spuc/fading_channel.cpp
${SPUC_DIR}/spuc/ofdm_data_encoder.cpp
${SPUC_DIR}/spuc/qam_conv_decoder.cpp
${SPUC_DIR}/spuc/qam_mod.cpp
${SPUC_DIR}/spuc/qam_soft_decision.cpp
${SPUC_DIR}/spuc/qpsk_ber_test.cpp
${SPUC_DIR}/spuc/qpsk_discriminators.cpp
${SPUC_DIR}/spuc/viterbi.cpp
)


include_directories(${SPUC_DIR})
#add_definitions("-fPIC")
add_library(${CPM_LIB_TARGET_NAME} STATIC ${SPUC_SRCS} ${COMM_SRCS} ${FILT_SRCS} ${TYPE_SRCS})
target_link_libraries(${CPM_LIB_TARGET_NAME} ${CPM_LIBRARIES})

CPM_ExportAdditionalIncludeDir("${SPUC_DIR}")

