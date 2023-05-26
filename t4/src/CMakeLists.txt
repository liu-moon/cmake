ADD_EXECUTABLE(main main.cpp)
INCLUDE_DIRECTORIES(/t2/usr/include/hello)
#TARGET_LINK_LIBRARIES(main /t2/usr/lib/libhello.so)
TARGET_LINK_LIBRARIES(main /t2/usr/lib/libhello.a)