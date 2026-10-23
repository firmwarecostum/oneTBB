# Copyright (c) 2020-2023 Intel Corporation
# Copyright (c) 2026 UXL Foundation Contributors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set(PY_EXECUTABLE "")
if(CMAKE_VERSION VERSION_LESS 3.12)
  find_package(PythonInterp 3.5 REQUIRED)
  set(PY_EXECUTABLE "${PYTHON_EXECUTABLE}")
else()
  find_package(Python3 3.5 REQUIRED COMPONENTS Interpreter)
  set(PY_EXECUTABLE "${Python3_EXECUTABLE}")
endif()

set(PYTHON_BUILD_WORK_DIR python_build)

configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/setup.py.in  ${CMAKE_CURRENT_SOURCE_DIR}/setup.py)

add_custom_target(
    python_copy
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/tbb ${PYTHON_BUILD_WORK_DIR}/tbb
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/TBB.py ${PYTHON_BUILD_WORK_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/setup.py ${PYTHON_BUILD_WORK_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/pyproject.toml ${PYTHON_BUILD_WORK_DIR}
) 

# Python build requires path to TBB headers
get_target_property(TBB_INCLUDES tbb INTERFACE_INCLUDE_DIRECTORIES)
foreach(dir ${TBB_INCLUDES})
    if (${dir} MATCHES "<BUILD_INTERFACE:" OR TBB_FOUND)
        set(TBB4PY_INCLUDE_STRING "${dir} ${TBB4PY_INCLUDE_STRING}")
    endif()
endforeach()
string(STRIP "${TBB4PY_INCLUDE_STRING}" TBB4PY_INCLUDE_STRING)

add_custom_target(
    python_build
    ALL
    DEPENDS tbb python_copy
    COMMAND
    ${CMAKE_COMMAND} -E env
        TBB_INCLUDEDIRS="${TBB4PY_INCLUDE_STRING}"
        TBB_LIBDIRS=$<TARGET_FILE_DIR:TBB::tbb>
    CC="aarch64-openwrt-linux-gnu-gcc" CCSHARED="aarch64-openwrt-linux-gnu-gcc -DPIC -fPIC" CXX="aarch64-openwrt-linux-gnu-g++" LD="aarch64-openwrt-linux-gnu-gcc" LDSHARED="aarch64-openwrt-linux-gnu-gcc -shared" CFLAGS="-Os -pipe -mcpu=generic -fno-caller-saves -fno-plt -fhonour-copts -ffile-prefix-map=/home/runner/work/mosdns/mosdns/sdk/build_dir/target-aarch64_generic_glibc/oneTBB-2023.1.0=oneTBB-2023.1.0 -Wl,-z,max-page-size=4096 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -Wl,-z,pack-relative-relocs" CPPFLAGS="-I/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/usr/include -I/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/include -I/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc/usr/include/python3.14" LDFLAGS="-L/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/usr/lib -L/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/lib -fuse-ld=bfd -zmax-page-size=4096 -znow -zrelro -zpack-relative-relocs -lpython3.14" _PYTHON_HOST_PLATFORM="linux-aarch64" _PYTHON_SYSCONFIGDATA_NAME="_sysconfigdata__linux_aarch64-linux-gnu" PYTHONPATH="/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc/usr/lib/python3.14:/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc//usr/lib/python3.14/site-packages:/home/runner/work/mosdns/mosdns/sdk/build_dir/target-aarch64_generic_glibc/oneTBB-2023.1.0/ipkg-install//usr/lib/python3.14/site-packages" PYTHONDONTWRITEBYTECODE=1 _python_sysroot="/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc" _python_prefix="/usr" _python_exec_prefix="/usr" ${PY_EXECUTABLE} -m pip install
        --no-build-isolation
        --prefix build
        ./${PYTHON_BUILD_WORK_DIR}
    COMMENT "Build and install to work directory the oneTBB Python module"
)

add_test(NAME python_test
         COMMAND ${CMAKE_COMMAND}
                 -DTBB_BINARIES_PATH=$<TARGET_FILE_DIR:TBB::tbb>
                 -DPYTHON_MODULE_BUILD_PATH=${PYTHON_BUILD_WORK_DIR}/build
                 -DPY_EXECUTABLE=CC="aarch64-openwrt-linux-gnu-gcc" CCSHARED="aarch64-openwrt-linux-gnu-gcc -DPIC -fPIC" CXX="aarch64-openwrt-linux-gnu-g++" LD="aarch64-openwrt-linux-gnu-gcc" LDSHARED="aarch64-openwrt-linux-gnu-gcc -shared" CFLAGS="-Os -pipe -mcpu=generic -fno-caller-saves -fno-plt -fhonour-copts -ffile-prefix-map=/home/runner/work/mosdns/mosdns/sdk/build_dir/target-aarch64_generic_glibc/oneTBB-2023.1.0=oneTBB-2023.1.0 -Wl,-z,max-page-size=4096 -Wformat -Werror=format-security -fstack-protector -D_FORTIFY_SOURCE=1 -Wl,-z,now -Wl,-z,relro -Wl,-z,pack-relative-relocs" CPPFLAGS="-I/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/usr/include -I/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/include -I/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc/usr/include/python3.14" LDFLAGS="-L/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/usr/lib -L/home/runner/work/mosdns/mosdns/sdk/staging_dir/toolchain-aarch64_generic_gcc-14.3.0_glibc/lib -fuse-ld=bfd -zmax-page-size=4096 -znow -zrelro -zpack-relative-relocs -lpython3.14" _PYTHON_HOST_PLATFORM="linux-aarch64" _PYTHON_SYSCONFIGDATA_NAME="_sysconfigdata__linux_aarch64-linux-gnu" PYTHONPATH="/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc/usr/lib/python3.14:/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc//usr/lib/python3.14/site-packages:/home/runner/work/mosdns/mosdns/sdk/build_dir/target-aarch64_generic_glibc/oneTBB-2023.1.0/ipkg-install//usr/lib/python3.14/site-packages" PYTHONDONTWRITEBYTECODE=1 _python_sysroot="/home/runner/work/mosdns/mosdns/sdk/staging_dir/target-aarch64_generic_glibc" _python_prefix="/usr" _python_exec_prefix="/usr" ${PY_EXECUTABLE}
                 -P ${PROJECT_SOURCE_DIR}/cmake/python/test_launcher.cmake)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/build/
        DESTINATION .
        COMPONENT tbb4py)

if (UNIX AND NOT APPLE)
    add_subdirectory(rml)
    add_dependencies(python_build irml)
endif()
