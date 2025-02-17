cmake_minimum_required(VERSION 3.0)

# Define the project name of these source code
project (gauss)

# We use a CMake module to fine the SystemC library. Define the module path
set (CMAKE_PREFIX_PATH $ENV{SYSTEMC_HOME})

# Find the package, which defines variable SYSTEMC_INCLUDE_DIRS
# and SYSTEMC_LIBRARIES.
find_package(SystemCLanguage CONFIG REQUIRED)

# Make Compiler option aligned to SystemC
set (CMAKE_CXX_STANDARD ${SystemC_CXX_STANDARD} CACHE STRING
     "C++ standard to build all targets. Supported values are 98, 11, and 14.")
set (CMAKE_CXX_STANDARD_REQUIRED ${SystemC_CXX_STANDARD_REQUIRED} CACHE BOOL
     "The with CMAKE_CXX_STANDARD selected C++ standard is a requirement.")

# Add optimization & warning flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -Wall")

# Set clock period = 1 ns
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D CLOCK_PERIOD=1.0")

# Define the executable name and its source code
file(GLOB SRC "*.cpp")
add_executable(gauss ${SRC})

# Define the used libraries of the executable, Equal to -l flags of g++
# In here, the libraries are given in apsolute path. hence no needs for -L
# flag
target_link_libraries(gauss SystemC::systemc)

# Add test gauss filter command
set(OUT_FILE "out.bmp")
add_custom_command(OUTPUT ${OUT_FILE}
	COMMAND ./gauss ${CMAKE_SOURCE_DIR}/lena_std_short.bmp ./${OUT_FILE}
	DEPENDS gauss
	WORKING_DIRECTORY ${CMAKE_BINARY_DIR}	
)
add_custom_target(run 
	DEPENDS ${OUT_FILE}
)

# Compare output & golden
add_custom_target(check
	COMMAND diff -s ${CMAKE_BINARY_DIR}/${OUT_FILE} golden/lena_std_short_gauss.bmp
	DEPENDS run
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}	
)
