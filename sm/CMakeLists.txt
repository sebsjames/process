#
# This file controls the installation of the header files.
#

# Header installation
install(
  FILES
  process

  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/sm
)

# Install the EXPORT so that maths has its own .cmake file and find_package(process) should work
install(FILES process-config.cmake DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/process)
