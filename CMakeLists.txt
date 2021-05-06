cmake_minimum_required(VERSION 3.10)
project(zetasql-sample-deps)

include(ExternalProject)

set(DEPS_PREFIX ${CMAKE_SOURCE_DIR}/usr)
set(DEPS_BUILD_DIR
    "${CMAKE_CURRENT_SOURCE_DIR}/deps"
    CACHE PATH "Dependencies build directory.")
set(DEPS_DOWNLOAD_DIR
    "${DEPS_BUILD_DIR}/downloads"
    CACHE PATH "Dependencies download directory.")

set(DEPS_CONFIG --prefix=${DEPS_PREFIX} --disable-shared --with-pic)

find_program(MAKE_EXE NAMES gmake nmake make)
ExternalProject_Add(
  protobuf
  URL https://github.com/protocolbuffers/protobuf/archive/v3.6.1.3.tar.gz
  URL_HASH
    SHA256=73fdad358857e120fd0fa19e071a96e15c0f23bb25f85d3f7009abfd4f264a2a
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/protobuf
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE True
  CONFIGURE_COMMAND ./autogen.sh && ./configure --prefix ${DEPS_PREFIX}
                    CXXFLAGS=-std=c++11
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install)

ExternalProject_Add(
  googletest
  URL https://github.com/google/googletest/archive//6f5fd0d7199b9a19faa9f499ecc266e6ae0329e7.tar.gz
  URL_HASH
    SHA256=51e6c4b4449aab8f31e69d0ff89565f49a1f3628a42e24f214e8b02b3526e3bc
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/googletest
  INSTALL_DIR ${DEPS_PREFIX}
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} -H${DEPS_BUILD_DIR}/src/googletest -B.
    -DCMAKE_INSTALL_PREFIX=${DEPS_PREFIX} -DCMAKE_INSTALL_LIBDIR=lib
    -DBUILD_SHARED_LIBS=OFF -Dgtest_build_tests=OFF -DCMAKE_CXX_FLAGS=-fPIC
  BUILD_COMMAND ${CMAKE_COMMAND} --build .
  INSTALL_COMMAND ${CMAKE_COMMAND} --build . --target install)

ExternalProject_Add(
  absl
  URL https://github.com/abseil/abseil-cpp/archive/a50ae369a30f99f79d7559002aba3413dac1bd48.tar.gz
  URL_HASH
    SHA256=be2a9d7ea7ee15f9317b57beff37e8ffb67418fb0df64592366b04c8618c2584
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/absl
  INSTALL_DIR ${DEPS_PREFIX}
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} -H${DEPS_BUILD_DIR}/src/absl -B. -DCMAKE_INSTALL_LIBDIR=lib
    -DCMAKE_INSTALL_PREFIX=${DEPS_PREFIX} -DCMAKE_BUILD_TYPE=Release
    -DCMAKE_CXX_STANDARD=17 -DABSL_USE_GOOGLETEST_HEAD=OFF -DABSL_RUN_TESTS=OFF
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  BUILD_COMMAND ${CMAKE_COMMAND} --build .
  INSTALL_COMMAND ${CMAKE_COMMAND} --build . --target install)

ExternalProject_Add(
  zetasql
  URL https://github.com/google/zetasql/archive/5ccb05880e72ab9ff75dd6b05d7b0acce53f1ea2.zip
  BUILD_IN_SOURCE True
  PREFIX ${DEPS_BUILD_DIR}
  CONFIGURE_COMMAND ""
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_COMMAND bazel build //zetasql/...
  INSTALL_COMMAND bash ${CMAKE_CURRENT_SOURCE_DIR}/../install_zetasql.sh)

ExternalProject_Add(
  googlelog
  URL https://github.com/google/glog/archive/refs/tags/v0.4.0.tar.gz
  URL_HASH
    SHA256=f28359aeba12f30d73d9e4711ef356dc842886968112162bc73002645139c39c
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/googlelog
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND ./autogen.sh && CXXFLAGS=-fPIC ./configure
                    --prefix="${DEPS_PREFIX}"
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install)

ExternalProject_Add(
  gflags
  URL https://github.com/gflags/gflags/archive/refs/tags/v2.2.0.tar.gz
  URL_HASH
    SHA256=466c36c6508a451734e4f4d76825cf9cd9b8716d2b70ef36479ae40f08271f88
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/gflags
  INSTALL_DIR ${DEPS_PREFIX}
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} -H${DEPS_BUILD_DIR}/src/gflags
    -DCMAKE_INSTALL_PREFIX="${DEPS_PREFIX}" -DGFLAGS_NAMESPACE=google
    -DCMAKE_CXX_FLAGS=-fPIC
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install)

ExternalProject_Add(
  zlib
  URL https://github.com/madler/zlib/archive/v1.2.11.tar.gz
  URL_HASH
    SHA256=629380c90a77b964d896ed37163f5c3a34f6e6d897311f1df2a7016355c45eff
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/zlib
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  PATCH_COMMAND
    sed -i '/CFLAGS="${CFLAGS--O3}"/c\  CFLAGS="${CFLAGS--O3} -fPIC"' configure
  CONFIGURE_COMMAND ./configure --static --prefix="${DEPS_PREFIX}"
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install)

ExternalProject_Add(
  snappy
  URL https://src.fedoraproject.org/lookaside/pkgs/snappy/snappy-1.1.1.tar.gz/8887e3b7253b22a31f5486bca3cbc1c2/snappy-1.1.1.tar.gz
  URL_HASH
    SHA256=d79f04a41b0b513a014042a4365ec999a1ad5575e10d5e5578720010cb62fab3
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/snappy
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND ./configure ${DEPS_CONFIG}
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install)

ExternalProject_Add(
  unwind
  URL https://github.com/libunwind/libunwind/archive/refs/tags/v1.1.tar.gz
  URL_HASH
    SHA256=2cd6bfa8d1862bcc0b3b1f70f49213c67dec162dc4b0f4e7bb96a60aeb8a0345
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/libunwind
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND ./configure --prefix="${DEPS_PREFIX}"
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install)

ExternalProject_Add(
  gperftools
  URL https://github.com/gperftools/gperftools/releases/download/gperftools-2.5/gperftools-2.5.tar.gz
  URL_HASH
    SHA256=6fa2748f1acdf44d750253e160cf6e2e72571329b42e563b455bde09e9e85173
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/gperftools
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND
    ./configure --enable-cpu-profiler --enable-heap-checker
    --enable-heap-profiler --enable-static --prefix="${DEPS_PREFIX}"
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install)

ExternalProject_Add(
  leveldb
  URL https://github.com/google/leveldb/archive/refs/tags/v1.20.tar.gz
  URL_HASH
    SHA256=f5abe8b5b209c2f36560b75f32ce61412f39a2922f7045ae764a2c23335b6664
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/leveldb
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  PATCH_COMMAND sed -i 's/^OPT ?= -O2 -DNDEBUG/OPT ?= -O2 -DNDEBUG -fPIC/'
                Makefile
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND cp -rf include/* "${DEPS_PREFIX}/include" && cp
                  out-static/libleveldb.a "${DEPS_PREFIX}/lib")

ExternalProject_Add(
  openssl
  URL https://github.com/openssl/openssl/archive/OpenSSL_1_1_0.zip
  URL_HASH
    SHA256=2150919847ae685b58ae26e737f7e3aff82a934d793c3aee5a01cfc9ef55c121
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/openssl
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND ./config --prefix="${DEPS_PREFIX}"
                    --openssldir="${DEPS_PREFIX}"
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install
  COMMAND rm -rf "${DEPS_PREFIX}"/lib/libssl.so*
  COMMAND rm -rf "${DEPS_PREFIX}"/lib/libcrypto.so*)

ExternalProject_Add(
  incubator-brpc
  URL https://github.com/4paradigm/incubator-brpc/archive/4f69bc0c04abc0734962722ba43aecb4dd7a5dea.zip
  URL_HASH
    SHA256=06b26bd153599055ec9a2670bb5cfaf758be65a5c3a1b346550b623607e3b29c
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/incubator-brpc
  INSTALL_DIR ${DEPS_PREFIX}
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND
    sh config_brpc.sh --with-glog --headers="${DEPS_PREFIX}/include"
    --libs="${DEPS_PREFIX}/lib"
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND cp -rf output/include/* "${DEPS_PREFIX}/include/"
  COMMAND cp libbrpc.a "${DEPS_PREFIX}/lib")

ExternalProject_Add(
  zookeeper
  URL https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
  URL_HASH
    SHA256=b14f7a0fece8bd34c7fffa46039e563ac5367607c612517aa7bd37306afbd1cd
  PREFIX ${DEPS_BUILD_DIR}
  DOWNLOAD_DIR ${DEPS_DOWNLOAD_DIR}/zookeeper
  INSTALL_DIR ${DEPS_PREFIX}
  BINARY_DIR ${DEPS_BUILD_DIR}/src/zookeeper/zookeeper-client/zookeeper-client-c
  BUILD_IN_SOURCE TRUE
  CONFIGURE_COMMAND autoreconf -if
    COMMAND ./configure --prefix="${DEPS_PREFIX}"
  BUILD_COMMAND ${MAKE_EXE}
  INSTALL_COMMAND ${MAKE_EXE} install
)
