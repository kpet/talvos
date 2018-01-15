# Copyright (c) 2018 the Talvos developers. All rights reserved.
#
# This file is distributed under a three-clause BSD license. For full license
# terms please see the LICENSE file distributed with this source code.

# Check for SPIRV headers
set(SPIRV_INCLUDE_DIR "" CACHE STRING "Directory containing SPIR-V headers")
set(CMAKE_REQUIRED_INCLUDES "${SPIRV_INCLUDE_DIR}")
check_include_file_cxx("spirv/unified1/spirv.hpp" SPIRV_HEADER_FOUND)
if (NOT SPIRV_HEADER_FOUND)
  message(FATAL_ERROR "SPIR-V header required, try setting SPIRV_INCLUDE_DIR.")
endif()

# Check for SPIRV-Tools header
set(SPIRV_TOOLS_INCLUDE_DIR "" CACHE STRING
    "Directory containing SPIRV-Tools headers")
set(CMAKE_REQUIRED_INCLUDES "${SPIRV_TOOLS_INCLUDE_DIR}")
check_include_file_cxx("spirv-tools/libspirv.hpp" SPIRV_TOOLS_HEADER_FOUND)
if (NOT SPIRV_TOOLS_HEADER_FOUND)
  message(FATAL_ERROR
          "SPIRV-Tools header required, try setting SPIRV_TOOLS_INCLUDE_DIR.")
endif()

# Check for SPIRV-Tools library
set(SPIRV_TOOLS_LIBRARY_DIR "" CACHE STRING
    "Directory containing SPIRV-Tools libraries")
find_library(SPIRV_TOOLS_LIB SPIRV-Tools PATHS "${SPIRV_TOOLS_LIBRARY_DIR}")
message(STATUS "SPIRV-Tools library: ${SPIRV_TOOLS_LIB}")
if (NOT SPIRV_TOOLS_LIB)
  message(FATAL_ERROR
          "SPIRV-Tools library required, try setting SPIRV_TOOLS_LIBRARY_DIR.")
endif()

set(TALVOS_HEADERS
    ${CMAKE_SOURCE_DIR}/include/talvos/Module.h)
add_library(talvos SHARED Module.cpp ${TALVOS_HEADERS})
target_include_directories(talvos PRIVATE "${SPIRV_INCLUDE_DIR}")
target_include_directories(talvos PRIVATE "${SPIRV_TOOLS_INCLUDE_DIR}")
target_link_libraries(talvos "${SPIRV_TOOLS_LIB}")

install(TARGETS talvos DESTINATION lib)
install(FILES ${TALVOS_HEADERS} DESTINATION include/talvos)