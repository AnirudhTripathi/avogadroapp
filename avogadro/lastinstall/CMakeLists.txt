if((APPLE OR WIN32) AND NOT ${CMAKE_VERSION} VERSION_LESS 2.8.8)
  set(pfx "")
  if(NOT APPLE)
    set(pfx "bin/")
  endif()

  set(sfx "")
  if(APPLE)
    set(sfx ".app")
  elseif(WIN32)
    set(sfx ".exe")
  endif()

  get_target_property(output_name avogadro OUTPUT_NAME)
  if(output_name)
    set(exe "${pfx}${output_name}${sfx}")
  else()
    set(exe "${pfx}avogadro${sfx}")
  endif()

  set(dirs "")
  if(CMAKE_PREFIX_PATH)
    foreach(dir ${CMAKE_PREFIX_PATH})
      list(APPEND dirs "${dir}/bin" "${dir}/lib")
    endforeach()
  endif()

  if(USE_VTK)
    # The VTK dlls are not in the CMAKE_PREFIX_PATH when built against
    # a "build tree" of VTK:
    get_property(loc TARGET VTK::ChartsCore PROPERTY LOCATION_RELEASE)
    get_filename_component(vtk_dlls_dir "${loc}" PATH)
    set(dirs ${dirs} "${vtk_dlls_dir}")
  endif()

  set(plugins "")
  foreach(plugin ${AvogadroLibs_PLUGINS})
    get_property(location TARGET ${plugin} PROPERTY LOCATION)
    list(APPEND plugins ${location})
  endforeach()
  # make sure to include the core image formats
  # likely only be required on Mac / Windows since Qt package on Unix provides
  if(APPLE OR WIN32)
    # JPG BMP, WebP, GIF plus default of PNG, etc.
    foreach(qt_plugin Qt5::QJpegPlugin Qt5::QWbmpPlugin Qt5::QWebpPlugin Qt5::QGifPlugin)
      get_property(location TARGET ${qt_plugin} PROPERTY LOCATION_RELEASE)
      list(APPEND plugins ${location})
    endforeach()
  endif()

  # Find and fixup the Open Babel plugins (if installed)
  find_program(OBABEL_EXE obabel)
  if(INSTALL_BUNDLE_FILES AND OBABEL_EXE AND APPLE)
    get_filename_component(BABEL_DIR "${OBABEL_EXE}" PATH)
    file(GLOB BABEL_PLUGINS
      RELATIVE ${BABEL_DIR}/../lib/openbabel/
      ${BABEL_DIR}/../lib/openbabel/*.so)
    foreach(plugin ${BABEL_PLUGINS})
      list(APPEND ob_plugins ${INSTALL_LIBRARY_DIR}/openbabel/${plugin})
    endforeach()
  endif()
  set(CMAKE_INSTALL_UCRT_LIBRARIES TRUE)
  include(InstallRequiredSystemLibraries)
  include(DeployQt5)
  install_qt5_executable(${exe} "${plugins}" "${ob_plugins}" "${dirs}" "")
endif()
