# Generating tracing primitives
set ( GEN_INC_DIR ${PROJECT_BINARY_DIR}/inc )
set ( GEN_SRC_DIR ${PROJECT_BINARY_DIR}/src )
execute_process ( COMMAND sh -xc "mkdir -p ${GEN_INC_DIR}" )
execute_process ( COMMAND sh -xc "mkdir -p ${GEN_SRC_DIR}" )
execute_process ( COMMAND sh -xc "${ROOT_DIR}/script/hsaap.py ${PROJECT_BINARY_DIR} ${HSA_RUNTIME_INC_PATH}" )
execute_process ( COMMAND sh -xc "${ROOT_DIR}/script/kfdap.py ${PROJECT_BINARY_DIR} ${HSA_KMT_INC_PATH}" )
execute_process ( COMMAND sh -xc "${CMAKE_C_COMPILER} -E ${HSA_KMT_INC_PATH}/hsakmttypes.h > ${GEN_INC_DIR}/hsakmttypes_pp.h" )
execute_process ( COMMAND sh -xc "${ROOT_DIR}/script/gen_ostream_ops.py -in ${GEN_INC_DIR}/hsakmttypes_pp.h -out ${GEN_INC_DIR}/kfd_ostream_ops.h" )
execute_process ( COMMAND sh -xc "${CMAKE_C_COMPILER} -E ${HIP_PATH}/include/hip/hip_runtime_api.h ${HIP_DEFINES} -I${HIP_PATH}/include -I${ROCM_ROOT_DIR}/hsa/include > ${GEN_INC_DIR}/hip_runtime_api_pp.h" )
execute_process ( COMMAND sh -xc "${ROOT_DIR}/script/gen_ostream_ops.py -in ${GEN_INC_DIR}/hip_runtime_api_pp.h -out ${GEN_INC_DIR}/hip_ostream_ops.h" )
execute_process ( COMMAND sh -xc "${CMAKE_C_COMPILER} -E ${HSA_RUNTIME_INC_PATH}/hsa.h > ${GEN_INC_DIR}/hsa_pp.h" )
execute_process ( COMMAND sh -xc "${ROOT_DIR}/script/gen_ostream_ops.py -in ${GEN_INC_DIR}/hsa_pp.h -out ${GEN_INC_DIR}/hsa_ostream_ops.h" )

# Build dynamic Library object
set ( TARGET_LIB ${TARGET_NAME} )
set ( LIB_SRC
  ${LIB_DIR}/core/roctracer.cpp
  ${LIB_DIR}/proxy/proxy_queue.cpp
  ${LIB_DIR}/proxy/simple_proxy_queue.cpp
  ${LIB_DIR}/proxy/intercept_queue.cpp
  ${LIB_DIR}/util/hsa_rsrc_factory.cpp
)
add_library ( ${TARGET_LIB} ${LIBRARY_TYPE} ${LIB_SRC} )
target_include_directories ( ${TARGET_LIB} PRIVATE ${LIB_DIR} ${ROOT_DIR} ${ROOT_DIR}/inc ${HSA_RUNTIME_INC_PATH} ${HSA_RUNTIME_HSA_INC_PATH} ${HIP_INC_DIR} ${HCC_INC_DIR} ${HSA_KMT_INC_PATH} ${GEN_INC_DIR} )
target_link_libraries( ${TARGET_LIB} PRIVATE ${HSA_RUNTIME_LIB} c stdc++ )

# Build KFD/Thunk tracing library
set ( KFD_LIB "kfdwrapper64" )
set ( KFD_LIB_SRC ${GEN_SRC_DIR}/kfd_wrapper.cpp)
add_library ( ${KFD_LIB} SHARED ${KFD_LIB_SRC} )
target_include_directories ( ${KFD_LIB} PRIVATE ${LIB_DIR} ${ROOT_DIR} ${ROOT_DIR}/inc ${HSA_RUNTIME_INC_PATH} ${HSA_RUNTIME_HSA_INC_PATH} ${HSA_KMT_INC_PATH} ${GEN_INC_DIR} )
target_link_libraries( ${KFD_LIB} PRIVATE c stdc++ )

# Build ROCTX tracing library
set ( ROCTX_LIB "roctx64" )
set ( ROCTX_LIB_SRC
  ${LIB_DIR}/roctx/roctx.cpp
  ${LIB_DIR}/roctx/roctx_intercept.cpp
)
add_library ( ${ROCTX_LIB} SHARED ${ROCTX_LIB_SRC} )
target_include_directories ( ${ROCTX_LIB} PRIVATE ${LIB_DIR} ${ROOT_DIR} ${ROOT_DIR}/inc ${HSA_RUNTIME_INC_PATH} ${HSA_RUNTIME_HSA_INC_PATH} ${GEN_INC_DIR} )
target_link_libraries( ${ROCTX_LIB} PRIVATE c stdc++ )
