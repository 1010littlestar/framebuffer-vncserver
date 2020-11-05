PROJECT(framebuffer-vncserver)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)


FILE(GLOB SOURCES src/*.c)
ADD_EXECUTABLE(framebuffer-vncserver ${SOURCES})
INSTALL(TARGETS framebuffer-vncserver RUNTIME DESTINATION bin)

# LIBVNC
find_library(LIBVNC NAMES libvncserver vncserver)
target_link_libraries(framebuffer-vncserver ${LIBVNC})


MESSAGE( STATUS "LIBVNC:         " ${LIBVNC} )

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

#设置安装包的包名，打好的包将会是packagename-version-linux.deb，如果不设置，默认是工程名
set(CPACK_PACKAGE_NAME "framebuffer-vncserver")

#设置程序名，就是程序安装后的名字
set(CPACK_DEBIAN_PACKAGE_NAME "")

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
install(FILES ${CMAKE_SOURCE_DIR}/framebuffer-vncserver DESTINATION /usr/local/bin)

 #最后 别忘记最重要的  我们使用cpack工具来实现打包工作的
 include(CPack)
