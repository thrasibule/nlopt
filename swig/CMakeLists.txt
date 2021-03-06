option (BUILD_PYTHON "build python bindings" ON)

find_package (PythonInterp)
find_package (PythonLibs)

if (NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/nlopt-python.cpp)
  find_package (SWIG)
endif ()

if (BUILD_PYTHON AND PYTHONINTERP_FOUND AND PYTHONLIBS_FOUND AND (SWIG_FOUND OR (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/nlopt-python.cpp)) AND BUILD_SHARED_LIBS)

  include_directories (${CMAKE_SOURCE_DIR}/api)
  include_directories (${PYTHON_INCLUDE_DIRS})

  if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/nlopt-python.cpp)
    set (SWIG_MODULE_nlopt_REAL_NAME _nlopt)
    add_library (${SWIG_MODULE_nlopt_REAL_NAME} MODULE nlopt-python.cpp)
    target_link_libraries(${SWIG_MODULE_nlopt_REAL_NAME} nlopt)
    target_link_libraries(${SWIG_MODULE_nlopt_REAL_NAME} ${PYTHON_LIBRARIES})
    set_target_properties (${SWIG_MODULE_nlopt_REAL_NAME} PROPERTIES PREFIX "")
    if (NOT CMAKE_SOURCE_DIR MATCHES CMAKE_BINARY_DIR)
      execute_process (COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/nlopt.py ${CMAKE_CURRENT_BINARY_DIR}/nlopt.py)
    endif ()
  else ()
    include (UseSWIG)
    set_source_files_properties (nlopt.i PROPERTIES CPLUSPLUS ON)
    set (SWIG_MODULE_nlopt_EXTRA_DEPS nlopt-python.i numpy.i ${CMAKE_SOURCE_DIR}/api/nlopt.hpp)

    swig_add_module (nlopt python nlopt.i)
    swig_link_libraries (nlopt nlopt)
    swig_link_libraries (nlopt ${PYTHON_LIBRARIES})
  endif ()
  execute_process ( COMMAND ${PYTHON_EXECUTABLE} -c "from distutils import sysconfig; print(sysconfig.get_python_lib(plat_specific=True, prefix='${CMAKE_INSTALL_PREFIX}'))"
                    OUTPUT_VARIABLE _ABS_PYTHON_MODULE_PATH
                    OUTPUT_STRIP_TRAILING_WHITESPACE )

  get_filename_component (_ABS_PYTHON_MODULE_PATH ${_ABS_PYTHON_MODULE_PATH} ABSOLUTE)
  file (RELATIVE_PATH _REL_PYTHON_MODULE_PATH ${CMAKE_INSTALL_PREFIX} ${_ABS_PYTHON_MODULE_PATH})

  set (PYTHON_MODULE_PATH ${_REL_PYTHON_MODULE_PATH})

  install ( FILES ${CMAKE_CURRENT_BINARY_DIR}/nlopt.py DESTINATION ${PYTHON_MODULE_PATH} )
  install ( TARGETS ${SWIG_MODULE_nlopt_REAL_NAME} LIBRARY DESTINATION ${PYTHON_MODULE_PATH} )
endif ()
