if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sps_vars.f90")
  message(FATAL_ERROR "The source code in the FSPS submodule was not found."
    "Please run 'git submodule update --init' to initialize the submodule.")
endif()

# Generate the f2py wrappers
set(F2PY_SOURCES
  "${CMAKE_CURRENT_BINARY_DIR}/_fspsmodule.c"
  "${CMAKE_CURRENT_BINARY_DIR}/_fsps-f2pywrappers2.f90"
)
add_custom_command(
  OUTPUT ${F2PY_SOURCES}
  DEPENDS fsps.f90
  VERBATIM
  COMMAND "${Python_EXECUTABLE}" -m numpy.f2py
  "${CMAKE_CURRENT_SOURCE_DIR}/fsps.f90" -m _fsps --lower
  --build-dir "${CMAKE_CURRENT_BINARY_DIR}")

# List out the explicit FSPS sources; we don't use a glob here since
# the 'src' directory includes some executables that we don't want to
# include here.
set(
  FSPS_SOURCES
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sps_vars.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sps_utils.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/add_agb_dust.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/add_bs.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/add_dust.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/add_nebular.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/add_remnants.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/add_xrb.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/agn_dust.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/attn_curve.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/compsp.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/csp_gen.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/funcint.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/get_lumdist.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/get_tuniv.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/getindx.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/getmags.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/getspec.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/igm_absorb.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/imf.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/imf_weight.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/intsfwght.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/linterp.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/linterparr.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/locate.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/mod_gb.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/mod_hb.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/pz_convol.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sbf.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/setup_tabular_sfh.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sfh_weight.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sfhinfo.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sfhlimit.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sfhstat.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/smoothspec.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/spec_bin.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/sps_setup.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/ssp_gen.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/tsum.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/vacairconv.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/write_isochrone.f90"
  "${CMAKE_CURRENT_SOURCE_DIR}/libfsps/src/ztinterp.f90"
)

if(WIN32)
  message(STATUS "Fixing Windows path separators...")
  
  # 修复 FSPS_SOURCES 列表中的所有路径
  set(FSPS_SOURCES_FIXED "")
  foreach(src ${FSPS_SOURCES})
    file(TO_CMAKE_PATH "${src}" src_fixed)
    list(APPEND FSPS_SOURCES_FIXED "${src_fixed}")
  endforeach()
  set(FSPS_SOURCES ${FSPS_SOURCES_FIXED})
  
  # 修复 F2PY 生成的文件路径
  if(DEFINED F2PY_MODULE_C)
    file(TO_CMAKE_PATH "${F2PY_MODULE_C}" F2PY_MODULE_C)
  endif()
  
  if(DEFINED F2PY_WRAPPER_F90)
    file(TO_CMAKE_PATH "${F2PY_WRAPPER_F90}" F2PY_WRAPPER_F90)
  endif()
  
  if(DEFINED FORTRANOBJECT_C)
    file(TO_CMAKE_PATH "${FORTRANOBJECT_C}" FORTRANOBJECT_C)
  endif()
  
  message(STATUS "Path fixing completed")
endif()

# Define the Python library
if(WIN32)
  message(STATUS "Using Windows-specific library creation...")
  
  # 收集所有源文件
  set(ALL_SOURCES
    ${FSPS_SOURCES}
    ${F2PY_MODULE_C}
    ${F2PY_WRAPPER_F90}
    ${FORTRANOBJECT_C}
  )
  
  # 使用 add_library 替代 python_add_library
  add_library(_fsps MODULE ${ALL_SOURCES})
  
  # 设置模块属性
  set_target_properties(_fsps PROPERTIES
    PREFIX ""
    SUFFIX ".pyd"
    OUTPUT_NAME "_fsps"
  )
  
  # 链接 Python 库
  target_link_libraries(_fsps PRIVATE Python::Module Python::NumPy)
  
  # 安装目标
  install(TARGETS _fsps LIBRARY DESTINATION ${SKBUILD_PROJECT_NAME})
  
else()
  # 非 Windows 系统使用原来的方式
  python_add_library(_fsps MODULE
    ${FSPS_SOURCES}
    ${F2PY_MODULE_C}
    ${F2PY_WRAPPER_F90}
    ${FORTRANOBJECT_C}
    WITH_SOABI
  )
endif()
target_link_libraries(_fsps PUBLIC Python::NumPy)
target_include_directories(_fsps PUBLIC "${F2PY_INCLUDE_DIR}")
install(TARGETS _fsps DESTINATION ${SKBUILD_PROJECT_NAME})
