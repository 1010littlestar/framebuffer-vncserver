PROJECT(framebuffer-vncserver)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

FILE(GLOB SOURCES src/*.c)
ADD_EXECUTABLE(${PROJECT_NAME} ${SOURCES})
INSTALL(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)



EXECUTE_PROCESS( COMMAND uname -m COMMAND tr -d '\n' OUTPUT_VARIABLE ARCHITECTURE )
message( STATUS "Architecture: ${ARCHITECTURE}" )
if( ${ARCHITECTURE} STREQUAL "aarch64" )
    # LIBVNC
    find_library(LIBVNC NAMES libvncserver vncserver)
    target_link_libraries(framebuffer-vncserver ${LIBVNC})
    MESSAGE( STATUS "LIBVNC:         " ${LIBVNC} )
else()
    # 交叉编译
    set(CMAKE_SYSTEM_NAME Linux)
    set(TOOLCHAIN_PATH /opt/compiler)
    set(CMAKE_C_COMPILER ${TOOLCHAIN_PATH}/bin/aarch64-linux-gnu-gcc)
    set(LIB_DIRS "${TOOLCHAIN_PATH}/luoxiang/lib/aarch64-linux-gnu/"
    "${TOOLCHAIN_PATH}/luoxiang/usr/lib/aarch64-linux-gnu/")

    target_link_directories(${PROJECT_NAME} PRIVATE ${LIB_DIRS})
    target_link_libraries(${PROJECT_NAME} vncserver)

    # sysroot location
    set(MYSYSROOT ${TOOLCHAIN_PATH}/luoxiang)
    # compiler/linker flags
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} --sysroot=${MYSYSROOT}" CACHE INTERNAL "" FORCE)
    set(CMAKE_C_LINK_FLAGS "${CMAKE_C_LINK_FLAGS} --sysroot=${MYSYSROOT}" CACHE INTERNAL "" FORCE)

    # cmake built-in settings to use find_xxx() functions
    set(CMAKE_FIND_ROOT_PATH "${MYSYSROOT}")
    set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
    set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
endif()




#准备的版本设置
set(_VERSION_MAJOR 1)
set(_VERSION_MINOR 0)
set(_VERSION_PATCH 0)

#说明要生成的是deb包

set(CPACK_GENERATOR "DEB")

############下面是设置debian/control文件中的内容

#设置版本信息
set(CPACK_PACKAGE_VERSION_MAJOR "${_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${_VERSION_PATCH}")

#设置架构
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "arm64")

#设置依赖
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libvncserver1")

#设置section
set(CPACK_DEBIAN_PACKAGE_SECTION "net")

#设置priority
set(CPACK_DEBIAN_PACKAGE_PRIORITY "Optional")

#设置description
set(CPACK_PACKAGE_DESCRIPTION "luoxiang box vnc server")

#设置联系方式
set(CPACK_PACKAGE_CONTACT "qiaosx@botech.com.cn")

#设置维护人
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "qiaosx@botech.com.cn")

##############debian/control设置完成了，下面设置控制脚本，动态库，changelog等
install(FILES ${CMAKE_SOURCE_DIR}/framebuffer-vncserver.service DESTINATION /lib/systemd/system)
install(FILES ${PROJECT_BINARY_DIR}/framebuffer-vncserver DESTINATION /usr/local/bin)

 #最后 别忘记最重要的  我们使用cpack工具来实现打包工作的
 include(CPack)
