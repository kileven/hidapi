cmake_minimum_required(VERSION 2.8)
project(hidapi)

find_package(Doxygen REQUIRED)
add_custom_target(hidapi_docs ALL
    ${DOXYGEN_EXECUTABLE}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/doxygen
)

if(NOT HAS_MZ_GLOBAL)
    include(global.cmake)
endif()

if(WIN32)
  set(SRC windows/hid.c)
elseif(APPLE)
  find_library(IOKit NAMES IOKit)
  find_library(CoreFoundation NAMES CoreFoundation)
  set(LIBS ${IOKit} ${CoreFoundation})
  set(SRC mac/hid.c)
else() # linux
  set(LIBS udev)
  set(SRC linux/hid.c)
endif()

set(HEADER hidapi/hidapi.h)

add_library(hidapi SHARED ${SRC} ${HEADER})
set_property(TARGET hidapi PROPERTY VERSION 0.7git)
set_property(TARGET hidapi PROPERTY SOVERSION 0.7git)
add_library(hidapi-static STATIC ${SRC} ${HEADER})
set_property(TARGET hidapi-static PROPERTY VERSION 0.7git)
set_property(TARGET hidapi-static PROPERTY SOVERSION 0.7git)
target_link_libraries(hidapi ${LIBS})
target_link_libraries(hidapi-static ${LIBS})
mz_target_props(hidapi)
mz_target_props(hidapi-static)

if(WIN32)
    install(TARGETS hidapi
                LIBRARY DESTINATION lib
                RUNTIME DESTINATION bin
                ARCHIVE DESTINATION lib)
    install(TARGETS hidapi-static
                LIBRARY DESTINATION lib
                RUNTIME DESTINATION bin
                ARCHIVE DESTINATION lib)
    install(DIRECTORY hidapi
                DESTINATION include
                PATTERN "*.svn" EXCLUDE)
    install(DIRECTORY doxygen/html
                DESTINATION doc
                PATTERN "*.svn" EXCLUDE)
else()
    install(TARGETS hidapi
                LIBRARY DESTINATION lib
                RUNTIME DESTINATION bin
                ARCHIVE DESTINATION lib)
    install(TARGETS hidapi-static
                LIBRARY DESTINATION lib
                RUNTIME DESTINATION bin
                ARCHIVE DESTINATION lib)
    install(DIRECTORY hidapi
                DESTINATION include
                PATTERN "*.svn" EXCLUDE)
    if(APPLE)
        install(DIRECTORY doxygen/html
                DESTINATION Library/Documentation/hidapi
                PATTERN "*.svn" EXCLUDE)
    else()
        install(DIRECTORY doxygen/html
                DESTINATION share/doc/hidapi-doc
                PATTERN "*.svn" EXCLUDE)
    endif()
endif()
