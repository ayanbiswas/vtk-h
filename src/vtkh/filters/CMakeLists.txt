#==============================================================================
# See License.txt
#==============================================================================

set(Headers
  Filter.hpp
  Clip.hpp
  MarchingCubes.hpp
  Threshold.hpp
  WaveletCompressor.hpp
  )

set(Sources
  Clip.cpp
  MarchingCubes.cpp
  Threshold.cpp
  WaveletCompressor.cpp
  )
#
# add_library(vtkh_filters ${Sources} ${Headers})
#
# target_include_directories(vtkh_filters
#   PUBLIC
#     $<BUILD_INTERFACE:${VTKh_SOURCE_DIR}>
#     $<BUILD_INTERFACE:${VTKh_BINARY_DIR}>
#     $<INSTALL_INTERFACE:${VTKh_INSTALL_INCLUDE_DIR}>
#   )
#
# target_link_libraries(vtkh_filters
#   PUBLIC vtkh_core
#   PRIVATE vtkh_dependencies_private)

set(vtkh_filters_deps vtkh_core  )

if(CUDA_FOUND)
    # triggers cuda compile
    list(APPEND vtkh_filters_deps cuda)
endif()


blt_add_library(
  NAME vtkh_filters
  SOURCES ${Sources}
  HEADERS ${Headers}
  DEPENDS_ON ${vtkh_filters_deps}
  )

  ########
  # TODO: We should have a macro for install targets + headers
  ########


# Install libraries
install(TARGETS vtkh_filters
  EXPORT ${VTKh_EXPORT_NAME}
  ARCHIVE DESTINATION ${VTKh_INSTALL_LIB_DIR}
  LIBRARY DESTINATION ${VTKh_INSTALL_LIB_DIR}
  RUNTIME DESTINATION ${VTKh_INSTALL_BIN_DIR}
  )

# Install headers
install(FILES ${Headers}
  DESTINATION ${VTKh_INSTALL_INCLUDE_DIR}/vtkh/filters)

if (MPI_FOUND)

    set(vtkh_filters_par_deps vtkh_core_par vtkh_utils )

    # necessary b/c of how find_cuda works
    if(CUDA_FOUND)
         set(CUDA_NVCC_FLAGS_PREV ${CUDA_NVCC_FLAGS})
         set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -D PARALLEL")
         # triggers cuda compile
         list(APPEND vtkh_filters_par_deps cuda)
    endif()
    
    blt_add_library(
      NAME vtkh_filters_par
      SOURCES ${Sources}
      HEADERS ${Headers}
      DEPENDS_ON ${vtkh_filters_par_deps}
      )

    if(CUDA_FOUND)
         SET(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS_PREV})
    else()
         blt_add_target_compile_flags(TO vtkh_filters_par FLAGS "-D PARALLEL")
    endif()
    


  # target_include_directories(vtkh_filters_par
  #   PUBLIC
  #     $<BUILD_INTERFACE:${VTKh_SOURCE_DIR}>
  #     $<BUILD_INTERFACE:${VTKh_BINARY_DIR}>
  #     $<INSTALL_INTERFACE:${VTKh_INSTALL_INCLUDE_DIR}>
  #   )
  # target_link_libraries(vtkh_filters_par
  #   PUBLIC vtkh_core_par
  #   PRIVATE vtkh_dependencies_private)


  ########
  # TODO: We should have a macro for install targets + headers
  ########


  # Install libraries
  install(TARGETS vtkh_filters_par
    EXPORT ${VTKh_EXPORT_NAME}
    ARCHIVE DESTINATION ${VTKh_INSTALL_LIB_DIR}
    LIBRARY DESTINATION ${VTKh_INSTALL_LIB_DIR}
    RUNTIME DESTINATION ${VTKh_INSTALL_BIN_DIR}
    )
endif()