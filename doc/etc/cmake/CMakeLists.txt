cmake_minimum_required(VERSION 2.8)

# 设置项目名称
project(logic_service)

# 设置C++标准
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORT_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORT_CXX0X)
if(COMPILER_SUPPORT_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -pthread -m64 -DSUS_LINUX -g")
#elseif(COMPILER_SUPPORT_CXX0X)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "the compiler ${CMAKE_CXX_COMPILER} not support C++11.")
endif()

# 路径相关定义
## 公共组件库的路径
set(FUTU_BASE_LIB_PATH /usr/local/fsi_libs/)
set(FUTU_SRPC_PATH ${FUTU_BASE_LIB_PATH}/srpc_lib)
#set(FUTU_SVR_BASE_PATH ${FUTU_BASE_LIB_PATH}/svr_lib/proto3)
set(FUTU_SVR_BASE_PATH ${FUTU_BASE_LIB_PATH}/svr_lib/)
set(FUTU_FTRACE_PATH ${FUTU_SVR_BASE_PATH}/ftrace)
set(FUTU_CMLB_PATH ${FUTU_BASE_LIB_PATH}/cmlb_lib)
set(FUTU_ULS_PATH ${FUTU_BASE_LIB_PATH}/uls_lib)
set(FUTU_IM_BASE_PATH ${FUTU_BASE_LIB_PATH}/commlib/baselib)
set(FUT_3PARTY_PATH ${FUTU_SVR_BASE_PATH}/3rd_party)

## fsi lib 选择；svr_lib是master分支则采用 fsi 目录下的；否则采用 fsi2 目录下的（原因暂时未知）
#set(FUTU_FSI_PATH ${FUTU_SVR_BASE_PATH}/fsi)
set(FUTU_FSI_PATH ${FUTU_SVR_BASE_PATH}/fsi2)
set(FUT_FSI_LIB_PATH ${FUTU_FSI_PATH}/debug)
#set(FUT_FSI_LIB_PATH ${FUTU_FSI_PATH}/release)

## ftrace 库debug与release分支的选择
#set(FUTU_FTRACE_LIB_PATH ${FUTU_FTRACE_PATH}/debug)
set(FUTU_FTRACE_LIB_PATH ${FUTU_FTRACE_PATH}/release)

# 头文件所在目录
## fsi 相关头文件
include_directories(${FUTU_FSI_PATH}/include/ut ${FUTU_FSI_PATH}/include/nw ${FUTU_FSI_PATH}/include/oi ${FUTU_FSI_PATH}/include/cmn ${FUTU_FSI_PATH}/include/cmlb)
include_directories(${FUTU_FTRACE_PATH}/include/ftrace ${FUTU_FTRACE_PATH}/include/ftrace_queue ${FUTU_FTRACE_PATH}/include/ftrace_report_api)

## 3 party 相关头文件
### mysql include
include_directories(${FUT_3PARTY_PATH}/mariadb/include)
### hiredis
include_directories(${FUT_3PARTY_PATH}/hiredis/include)
### libev
include_directories(${FUT_3PARTY_PATH}/libev/include)
include_directories(${FUT_3PARTY_PATH}/jsoncpp/include)

## srpc 相关头文件
include_directories(${FUTU_SRPC_PATH}/include)
include_directories(${FUTU_SRPC_PATH}/include/spp_incl)
include_directories(${FUTU_SRPC_PATH}/include/rpc_incl)
include_directories(${FUTU_SRPC_PATH}/include/sync_incl)
include_directories(${FUTU_SRPC_PATH}/pb_lib/include)

## cares 库相关的头文件路径（mt_mysql 库依赖于该库）
include_directories(${FUT_3PARTY_PATH}/c-ares/include)
## uls
include_directories(${FUTU_ULS_PATH})
## cmlb
include_directories(${FUTU_CMLB_PATH})

## 自己项目中的头文件引入
include_directories(${PROJECT_SOURCE_DIR})
include_directories(${PROJECT_SOURCE_DIR}/proto)
include_directories(${PROJECT_SOURCE_DIR}/storage_client)
include_directories(${PROJECT_SOURCE_DIR}/src)

# 静态库目录
## mysql lib
link_directories(${FUT_3PARTY_PATH}/mariadb/lib64)
## hiredis
link_directories(${FUT_3PARTY_PATH}/hiredis/lib64)
## libev
link_directories(${FUT_3PARTY_PATH}/libev/lib64)
## misc
link_directories(${FUT_3PARTY_PATH}/misc/lib64)
link_directories(${FUT_3PARTY_PATH}/jsoncpp/lib64)

## srpc 相关lib
link_directories(${FUTU_SRPC_PATH}/lib)
link_directories(${FUTU_SRPC_PATH}/pb_lib/lib)

## libcares.a 所在目录
link_directories(${FUT_3PARTY_PATH}/c-ares/lib)
## liblogsys_client.a
link_directories(${FUTU_ULS_PATH})
## cmlb
link_directories(${FUTU_CMLB_PATH})
## fsi2
link_directories(${FUTU_FSI_PATH}/release)
## ftrace
link_directories(${FUTU_FTRACE_LIB_PATH})
## commlib
link_directories(${FUTU_BASE_LIB_PATH}/commlib/baselib)

## 练习项目中将一些公共代码封装起来的 libmain.a 库
link_directories(${PROJECT_SOURCE_DIR}/../../lib)

# 添加源文件
## 添加 .cpp 文件
aux_source_directory(${PROJECT_SOURCE_DIR} MAIN_SRC_DIR)
aux_source_directory(${PROJECT_SOURCE_DIR}/proto SUB_PROTO_DIR)
aux_source_directory(${PROJECT_SOURCE_DIR}/storage_client SUB_ST_CLIENT_DIR)
aux_source_directory(${PROJECT_SOURCE_DIR}/src SUB_SRC_DIR)

#########################################################################
# 编译 .proto 文件

## 获取 .proto 文件的编译器
find_program(
    PROTOC_CXX
    protoc
    HINTS /usr/local/fsi_libs/srpc_lib/pb_lib/bin/
    DOC "Protobuf Compiler (protoc)"
    NO_DEFAULT_PATH
    REQUIRED
)

## 找到所有要编译的 .proto 文件
file (GLOB PROTO_SOURCE_FILES
    "${PROJECT_SOURCE_DIR}/proto/*.proto"
)
set(PROTO_PATH "${PROJECT_SOURCE_DIR}/proto")
set(PROTO_CXX_OUT "${PROJECT_SOURCE_DIR}/proto")

#file(MAKE_DIRECTORY ${PROTO_CXX_OUT})

## 获取需要生成的文件
foreach(input_proto ${PROTO_SOURCE_FILES})
    get_filename_component(DIR ${input_proto} DIRECTORY)
    get_filename_component(FILE_NAME ${input_proto} NAME_WE)

    set(OUTPUT_CXX_HEADER   "${PROTO_CXX_OUT}/${FILE_NAME}.pb.h")
    set(OUTPUT_CXX_SOURCE   "${PROTO_CXX_OUT}/${FILE_NAME}.pb.cc")
    list(APPEND OUTPUT_SOURCES_CXX
        ${OUTPUT_CXX_HEADER} ${OUTPUT_CXX_SOURCE})
endforeach()

## 编译 .proto文件
add_custom_command(
    OUTPUT  ${OUTPUT_SOURCES_CXX}
    COMMAND ${PROTOC_CXX} --cpp_out=${PROTO_CXX_OUT} --proto_path=${PROTO_PATH} --proto_path=${FUTU_SRPC_PATH}/pb_lib/include --proto_path=${FUTU_SRPC_PATH}/comm ${PROTO_SOURCE_FILES}
    DEPENDS ${PROTO_SOURCE_FILES}
    WORKING_DIRECTORY ${PROTO_PATH}
    COMMENT "Generate Cpp Protobuf Source Files"
)

# 连接成动态库
## 设置动态库输出路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/../spp_frame/client/bin/)
add_library(logic_service SHARED ${MAIN_SRC_DIR} ${SUB_PROTO_DIR} ${SUB_ST_CLIENT_DIR} ${SUB_SRC_DIR} ${OUTPUT_SOURCES_CXX})
#add_executable(libstorage_service ${MAIN_SRC_DIR} ${SUB_SRC_DIR})

#########################################################################
# 添加静态库链接
# remark: 对于 libpthread.a 无法通过 target_link_libraries(day2_test libpthread.a) 进行链接，会报skipping incompatible /usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../libpthread.a when searching for -lpthread 的错误
## 链接 srpc、mt 库
target_link_libraries(logic_service PUBLIC librpc_comm.a libmt_async.a libmt.a)
## 链接 fsi 相关的库
target_link_libraries(logic_service PUBLIC liboi.a libcmlb.a libcmn.a libnw.a libut.a)
## 链接 cmlb 相关库
target_link_libraries(logic_service PUBLIC libcomm_albs_v2_client_api.a)
## 链接 ftrace 相关库（一定要在 rpc_comm 之后；protobuf 之前）
target_link_libraries(logic_service PUBLIC libftrace.a libftrace_queue.a libftrace_report_api.a)
## 链接 protobuf 相关库
target_link_libraries(logic_service PUBLIC libprotobuf.a)
## 链接 3rd-party 相关库
target_link_libraries(logic_service PUBLIC libjsoncpp.a libhiredis.a libmysqlclient_r.a)
## 链接 usl 相关库
target_link_libraries(logic_service PUBLIC liblogsys_client.a)
## 链接其他依赖的静态库
### note: 不是22年版本的公共库-fsi_lib，无需连接 libcares.a
#target_link_libraries(redis_test PUBLIC libcomm_oi.a libcurl.a libidn.a libev.a libcares.a)
target_link_libraries(logic_service PUBLIC libcomm_oi.a libcurl.a libidn.a libev.a)


# 添加动态库链接
target_link_libraries(logic_service PUBLIC -lssl -lcrypto -lz -ldl -lrt)
