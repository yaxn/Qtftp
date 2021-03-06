project(qtftp_lib)

# GNUInstallDirs sets variable CMAKE_INSTALL_LIBDIR that we use to install library in correct lib dir
include("GNUInstallDirs")


set( QTFTP_INCLUDE_FILES include/abstractsocket.h
                         include/udpsocket.h
                         include/udpsocketfactory.h
                         include/session.h
                         include/readsession.h
                         include/tftpserver.h
                         include/tftp_error.h
                         include/tftp_utils.h
                         include/tftp_constants.h
)

set( QTFTP_SOURCE_FILES src/udpsocket.cpp
                        src/udpsocketfactory.cpp
                        src/session.cpp
                        src/readsession.cpp
                        src/tftpserver.cpp
                        src/abstractsocket.cpp
                        src/tftp_utils.cpp
)

#because the include files are in a different directory than the .cpp files we have to include them
#as source files for our library, to let cmake automoc process them.
add_library(Qtftp STATIC ${QTFTP_SOURCE_FILES} ${QTFTP_INCLUDE_FILES} )
set_property(TARGET Qtftp PROPERTY VERSION ${QTFTP_VERSION})
set_property(TARGET Qtftp PROPERTY SOVERSION ${QTFTP_VERSION_MAJOR})
set_property(TARGET Qtftp PROPERTY DEBUG_POSTFIX d)
target_include_directories(Qtftp PUBLIC $<INSTALL_INTERFACE:include> $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
target_link_libraries(Qtftp 
    PUBLIC 
        Qt5::Network
    PRIVATE
        $<$<PLATFORM_ID:Windows>:Ws2_32.lib>
)

#if (NOT MSVC)
#    #below is a dummy target to let the header files show up in QtCreator. It can safely be removed if needed.
#    #add_custom_target(qtftpLibHeaderDummy SOURCES ${QTFTP_INCLUDE_FILES} )
#endif()

target_compile_features( Qtftp
    PUBLIC
        cxx_override
    
    PRIVATE
        cxx_auto_type
        cxx_constexpr
        cxx_deleted_functions
        cxx_lambdas
        cxx_noexcept
        cxx_strong_enums
        cxx_uniform_initialization
        cxx_user_literals
        cxx_raw_string_literals
        cxx_std_14
)

#When compiling with MSVC suppress suggestion to use checked iterators, because they are a Microsoft extension
#also suppress suggestion to use strcpy_s because it is Microsoft specific
target_compile_definitions( Qtftp
    PRIVATE $<$<CXX_COMPILER_ID:MSVC>: _SCL_SECURE_NO_WARNINGS _CRT_SECURE_NO_WARNINGS>
)

install(TARGETS Qtftp EXPORT QtftpTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)
install(FILES ${QTFTP_INCLUDE_FILES} DESTINATION include)

#And some boiler plate code to generate the cmake module files
install(EXPORT QtftpTargets
    FILE QtftpTargets.cmake
    NAMESPACE Qtftp::
    DESTINATION lib/cmake/Qtftp
)
include(CMakePackageConfigHelpers)
write_basic_package_version_file("QtftpConfigVersion.cmake"
    VERSION ${QTFTP_VERSION}
    COMPATIBILITY SameMajorVersion
)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/QtftpConfig.cmake" "
include(CMakeFindDependencyMacro)
find_dependency(Qt5Core ${QT_MIN_VERSION})
find_dependency(Qt5Network ${QT_MIN_VERSION})
include(\"\${CMAKE_CURRENT_LIST_DIR}/QtftpTargets.cmake\")
")

install(FILES  
    ${CMAKE_CURRENT_BINARY_DIR}/QtftpConfigVersion.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/QtftpConfig.cmake
    DESTINATION lib/cmake/Qtftp
)


