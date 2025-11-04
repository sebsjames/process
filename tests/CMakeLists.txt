# CTests

# All #includes in test programs have to be #include <sm/header>
include_directories(BEFORE ${PROJECT_SOURCE_DIR})

# Test sm::process class
if(APPLE)
  message("-- NB: Omitting testProcess.cpp on Mac for now, as it doesn't work.")
else(APPLE)
  add_executable(testProcess testProcess.cpp)
  add_test(testProcess testProcess)
endif(APPLE)
