cmake_minimum_required(VERSION 3.2)
project(admesh C)

# Include necessary modules
include(GNUInstallDirs)
include_directories(${CMAKE_SOURCE_DIR}/src)

# Set installation paths and rpath
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR})

# Versioning
set(VERSION 1.0.0)
set(SOVERSION 1)

# Source files for the library
set(ADMESH_SOURCES
    src/connect.c
    src/normals.c
    src/shared.c
    src/stl_io.c
    src/stlinit.c
    src/util.c
)

# Header files for the library
set(ADMESH_HEADERS
    src/stl.h
)

# Define the executable
add_executable(admesh
    src/admesh.c
)

# Create shared library
add_library(libadmesh SHARED ${ADMESH_SOURCES})
set_target_properties(libadmesh PROPERTIES
    VERSION ${VERSION}
    SOVERSION ${SOVERSION}
    PREFIX ""
)

# Link the library with the executable
target_link_libraries(admesh libadmesh m)

# Set up the pkg-config file
set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_FULL_BINDIR})
set(libdir ${CMAKE_INSTALL_FULL_LIBDIR})
set(includedir ${CMAKE_INSTALL_FULL_INCLUDEDIR})
configure_file(${CMAKE_SOURCE_DIR}/libadmesh.pc.in ${CMAKE_BINARY_DIR}/libadmesh.pc)

# Install targets
install(TARGETS libadmesh EXPORT admesh DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(TARGETS admesh EXPORT admesh DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES ${ADMESH_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/admesh)
install(FILES ${CMAKE_BINARY_DIR}/libadmesh.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/)
install(FILES FindADMESH.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/admesh)
install(EXPORT admesh DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/admesh)
target_compile_definitions(libadmesh PUBLIC -DVERSION="${VERSION}")

# Enable testing
enable_testing()

# Define a macro to simplify adding tests
macro(add_admesh_test testname testfile)
    add_test(NAME ${testname}
             COMMAND ${CMAKE_BINARY_DIR}/admesh ${CMAKE_SOURCE_DIR}/examples/${testfile}.stl ${ARGN})
    add_test(NAME ${testname}-compare
             COMMAND ${CMAKE_COMMAND} -E compare_files ${CMAKE_SOURCE_DIR}/test/${testfile}/${testname}.stl ${CMAKE_BINARY_DIR}/${testname}.stl)
endmacro()

# Add tests for each file
foreach(testfile "block" "conveyor" "hourglass")
    MESSAGE("Test ${testfile}")

    add_admesh_test(${testfile}-basic ${testfile} -a ${CMAKE_BINARY_DIR}/basic.stl)
    add_admesh_test(${testfile}-x-rotate-30 ${testfile} --x-rotate=30 -a ${CMAKE_BINARY_DIR}/x-rotate-30.stl)
    add_admesh_test(${testfile}-y-rotate-30 ${testfile} --y-rotate=30 -a ${CMAKE_BINARY_DIR}/y-rotate-30.stl)
    add_admesh_test(${testfile}-z-rotate-30 ${testfile} --z-rotate=30 -a ${CMAKE_BINARY_DIR}/z-rotate-30.stl)
    add_admesh_test(${testfile}-x-y-z-rotate-30 ${testfile} --x-rotate=30 --y-rotate=30 --z-rotate=30 -a ${CMAKE_BINARY_DIR}/x-y-z-rotate-30.stl)
    add_admesh_test(${testfile}-xy-mirror ${testfile} --xy-mirror -a ${CMAKE_BINARY_DIR}/xy-mirror.stl)
    add_admesh_test(${testfile}-yz-mirror ${testfile} --yz-mirror -a ${CMAKE_BINARY_DIR}/yz-mirror.stl)
    add_admesh_test(${testfile}-xz-mirror ${testfile} --xz-mirror -a ${CMAKE_BINARY_DIR}/xz-mirror.stl)
    add_admesh_test(${testfile}-scale ${testfile} --scale 3 -a ${CMAKE_BINARY_DIR}/scale.stl)
    add_admesh_test(${testfile}-scale-xyz ${testfile} --scale-xyz=2,3,4 -a ${CMAKE_BINARY_DIR}/scale-xyz.stl)
    add_admesh_test(${testfile}-translate ${testfile} --translate=2,3,4 -a ${CMAKE_BINARY_DIR}/translate.stl)
    add_admesh_test(${testfile}-translate-rel ${testfile} --translate-rel=2,3,4 -a ${CMAKE_BINARY_DIR}/translate-rel.stl)
    add_admesh_test(${testfile}-stretch ${testfile} --stretch=::22,::3,::5 -a ${CMAKE_BINARY_DIR}/stretch.stl)
    add_admesh_test(${testfile}-merge ${testfile} --merge=${CMAKE_SOURCE_DIR}/test/${testfile}/stretch.stl -a ${CMAKE_BINARY_DIR}/merge.stl)
    add_admesh_test(${testfile}-nearby ${testfile}/nearby-bad -n -t 0.1 -a ${CMAKE_BINARY_DIR}/nearby.stl)
    add_admesh_test(${testfile}-remove-unconnected ${testfile}/remove-unconnected-bad --remove-unconnected -a ${CMAKE_BINARY_DIR}/remove-unconnected.stl)
    add_admesh_test(${testfile}-fill-holes ${testfile}/fill-holes-bad --fill-holes -a ${CMAKE_BINARY_DIR}/fill-holes.stl)
    add_admesh_test(${testfile}-normal-directions ${testfile}/normal-directions-bad --normal-directions -a ${CMAKE_BINARY_DIR}/normal-directions.stl)
    add_admesh_test(${testfile}-off-stl ${testfile} --write-off ${CMAKE_BINARY_DIR}/basic.off)
    add_admesh_test(${testfile}-dxf-stl ${testfile} --write-dxf ${CMAKE_BINARY_DIR}/basic.dxf)
    add_admesh_test(${testfile}-vrml-stl ${testfile} --write-vrml ${CMAKE_BINARY_DIR}/basic.vrml)
endforeach()
