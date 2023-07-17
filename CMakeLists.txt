cmake_minimum_required(VERSION 3.5)
project(leveldb-mcpe)

set(CMAKE_CXX_FLAGS_RELEASE "-O2")

set(SOURCES
    ./db/builder.cc
    ./db/c.cc
    ./db/dbformat.cc
    ./db/db_impl.cc
    ./db/db_iter.cc
    ./db/dumpfile.cc
    ./db/filename.cc
    ./db/log_reader.cc
    ./db/log_writer.cc
    ./db/memtable.cc
    ./db/repair.cc
    ./db/snappy_compressor.cc
    ./db/table_cache.cc
    ./db/version_edit.cc
    ./db/version_set.cc
    ./db/write_batch.cc
    ./db/zlib_compressor.cc
    ./db/zopfli_compressor.cc
    ./db/zstd_compressor.cc
    ./table/block_builder.cc
    ./table/block.cc
    ./table/filter_block.cc
    ./table/format.cc
    ./table/iterator.cc
    ./table/merger.cc
    ./table/table_builder.cc
    ./table/table.cc
    ./table/two_level_iterator.cc
    ./util/arena.cc
    ./util/bloom.cc
    ./util/cache.cc
    ./util/coding.cc
    ./util/comparator.cc
    ./util/crc32c.cc
    ./util/env_boost.cc
    ./util/env.cc
    ./util/env_posix.cc
    ./util/env_win.cc
    ./util/filter_policy.cc
    ./util/hash.cc
    ./util/histogram.cc
    ./util/logging.cc
    ./util/options.cc
    ./util/status.cc
    ./util/win_logger.cc
    ./port/port_posix.cc
    ./port/port_posix_sse.cc
    )

set(MEMENV_SOURCES
    ./helpers/memenv/memenv.cc
)

include_directories(
    ./db
    ./table
    ./util
    ./port
    ./helpers/memenv
)

set(OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/output")
set(LIBRARY_OUTPUT_DIRECTORY ${OUTPUT_DIR})

set(PLATFORMS "ALL" "WIN32" "LINUX" "FREEBSD" "MACOS" "ANDROID" "IOS")

if (PLATFORMS STREQUAL "ALL")

elseif (PLATFORMS STREQUAL "WIN32")
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        include(CheckCXXCompilerFlag)
        check_cxx_compiler_flag("-mavx" COMPILER_SUPPORTS_AVX)

        if (COMPILER_SUPPORTS_AVX)
            set(COMPILE_OPTIONS "-mavx")
            set(TARGET_SUFFIX "-win32-amd64.dll")
        else()
            set(COMPILE_OPTIONS "")
            set(TARGET_SUFFIX "-win32-amd64-no_avx.dll")
        endif()
    endif()
elseif (PLATFORMS STREQUAL "LINUX")
    if(CMAKE_SIZEOF_VOID_P EQUAL 4)
        set(TARGET_SUFFIX "-linux-i686.so")
    elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)
        include(CheckCXXCompilerFlag)
        check_cxx_compiler_flag("-mavx" COMPILER_SUPPORTS_AVX)

        if (COMPILER_SUPPORTS_AVX)
            set(COMPILE_OPTIONS "-mavx")
            set(TARGET_SUFFIX "-linux-amd64.so")
        else()
            set(COMPILE_OPTIONS "")
            set(TARGET_SUFFIX "-linux-amd64-no_avx.so")
        endif()
    endif()
elseif (PLATFORMS STREQUAL "FREEBSD")
    if(CMAKE_SIZEOF_VOID_P EQUAL 4)
        set(TARGET_SUFFIX "-freebsd-i686.so")
    elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)
        include(CheckCXXCompilerFlag)
        check_cxx_compiler_flag("-mavx" COMPILER_SUPPORTS_AVX)

        if (COMPILER_SUPPORTS_AVX)
            set(COMPILE_OPTIONS "-mavx")
            set(TARGET_SUFFIX "-freebsd-amd64.so")
        else()
            set(COMPILE_OPTIONS "")
            set(TARGET_SUFFIX "-freebsd-amd64-no_avx.so")
        endif()
    endif()
elseif (PLATFORMS STREQUAL "MACOS")
    if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
        include(CheckCXXCompilerFlag)
        check_cxx_compiler_flag("-mavx" COMPILER_SUPPORTS_AVX)

        if (COMPILER_SUPPORTS_AVX)
            set(COMPILE_OPTIONS "-mavx")
            set(TARGET_SUFFIX "-macos-x86_64.dylib")
        else()
            set(COMPILE_OPTIONS "")
            set(TARGET_SUFFIX "-macos-x86_64-no_avx.dylib")
        endif()
    elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "arm64")
        # Apple Silicon
        set(TARGET_SUFFIX "-macos-arm64.dylib")
    endif ()
elseif (PLATFORMS STREQUAL "ANDROID")
    if(${ANDROID_ABI} STREQUAL "armeabi-v7a")
        set(TARGET_SUFFIX "-android-armeabi-v7a.so")
    elseif(${ANDROID_ABI} STREQUAL "arm64-v8a")
        set(TARGET_SUFFIX "-android-aarch64.so")
    elseif(${ANDROID_ABI} STREQUAL "x86")
        set(TARGET_SUFFIX "-android-x86.so")
    elseif (${ANDROID_ABI} STREQUAL "x86_64")
        set(TARGET_SUFFIX "-android-x86_64.so")
    else()
        message("Unsupported processor architecture, please submit a pull request or modify the CMakeList.txt file by yourself.")
    endif()
elseif (PLATFORMS STREQUAL "IOS")
    # Only for Apple Silicon.
    set(TARGET_SUFFIX "-ios.dylib")
else ()

endif ()


add_library(${PROJECT_NAME} SHARED ${SOURCES} ${MEMENV_SOURCES})
target_compile_options(${PROJECT_NAME} PRIVATE ${COMPILE_OPTIONS})
# set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "" ${COMPILE_OPTIONS} SUFFIX ${TARGET_SUFFIX} OUTPUT_NAME ${PROJECT_NAME})