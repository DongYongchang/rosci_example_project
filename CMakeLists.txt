# use cmake 3 which supports targets
cmake_minimum_required(VERSION 3.1.3)

# ===== PROJECT CONFIG =====
project(example_project)

# ===== SETUP EXECUTABLE =====
# set where the executable is built
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_CURRENT_SOURCE_DIR}/bin")
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/install)
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CONFIGURATION_TYPES "Release")


#message(${jsoncpp_INCLUDES_DIR})
# ===== CREATE EXECUTABLE FROM SOURCE =====
# Include app and library headers
include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}/include
)

# Add source files
file(GLOB_RECURSE  SRC_FILES 
	${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc
)

file(GLOB_RECURSE  HEAD_FILES
	${CMAKE_CURRENT_SOURCE_DIR}/include/*.h
)

#add_library(${PROJECT_NAME}  ${NNG_LIBRARY} soem spdlog wpcap Packet jsoncpp winmm)
add_library(${PROJECT_NAME} STATIC ${SRC_FILES} )

target_include_directories(${PROJECT_NAME}
	PUBLIC ${CMAKE_PREFIX_PATH}/include
)

# ===== LINK DEPENDENCIES ======
target_link_directories(${PROJECT_NAME}
	PUBLIC ${CMAKE_PREFIX_PATH}/lib
)

target_link_libraries(${PROJECT_NAME}
)

# ===== LINK DEPENDENCIES ======
#### Head file install ####
install(DIRECTORY 
	${CMAKE_CURRENT_SOURCE_DIR}/include/
	DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/install/include
	FILES_MATCHING 
	PATTERN "*.h"
	PATTERN "*.hpp"
)

#### Cmake package install #### 
install(TARGETS ${PROJECT_NAME}
EXPORT "${PROJECT_NAME}-targets"
RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# copy the *-targets.cmake file to the CMAKE_INSTALL_PREFIX directory
install(EXPORT "${PROJECT_NAME}-targets"
    FILE "${PROJECT_NAME}-targets.cmake"
    DESTINATION "lib/cmake/${PROJECT_NAME}"
    )

# copy the *.-config file to the CMAKE_INSTALL_PREFIX directory. This will specify the dependencies.
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}-config.cmake.in" 
			 "${PROJECT_NAME}-config.cmake" @ONLY)
install(FILES 
	"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake"
    DESTINATION 
	"lib/cmake/${PROJECT_NAME}"
  )

# we supply a package wrapper ourselves: cmake/Modules/FindXXX.cmake
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/Modules")


# set cmake compile features and options, this replaces the CMAKE_CXX_FLAGS
# for c++ version: cxx_std_11, cxx_std_14, cxx_std_17
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_EXTENSIONS OFF)
#target_compile_options(${PROJECT_NAME} PRIVATE -Werror)
set(CMAKE_VERBOSE_MAKEFILE ON)

# visual studio 19
if (MSVC)
    add_definitions(/MP)        # build in parallel
    # disable the security warnings for fopen, scanf etc.
    add_definitions(/D"_CRT_SECURE_NO_WARNINGS" /D"_SCL_SECURE_NO_WARNINGS")
	add_compile_definitions(__STDC_LIMIT_MACROS)
endif()
