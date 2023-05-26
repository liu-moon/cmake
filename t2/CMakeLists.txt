PROJECT(HELLO CXX)
ADD_SUBDIRECTORY(src bin)

# 安装COPYRIGHT README到目录 share/doc/cmake/t2
INSTALL(FILES COPYRIGHT README DESTINATION share/doc/cmake/t2)

# 安装runhello.sh 到目录 bin
INSTALL(PROGRAMS runhello.sh DESTINATION bin)

# 安装 doc目录中的内容到目录 share/doc/cmake/t2
INSTALL(DIRECTORY doc/ DESTINATION share/doc/cmake/t2)