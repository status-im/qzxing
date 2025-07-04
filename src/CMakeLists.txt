cmake_minimum_required(VERSION 3.5)
project(QZXing)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS
    Core Gui Svg Quick QuickControls2)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Core REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Gui REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Multimedia)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED Svg Quick QuickControls2)

SET(BIGINT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/zxing/bigint)
SET(WIN32_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/zxing/win32/zxing)
SET(ZXING_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/zxing/zxing)

set(CMAKE_AUTOMOC ON)
set(CMAKE_WARN_ON)

set(SOURCES

    CameraImageWrapper.cpp
    CameraImageWrapper.h
    ImageHandler.cpp
    ImageHandler.h
    QZXing.cpp
    QZXing.h
    QZXing_global.h
    )

if(QZXING_MULTIMEDIA)

    if(${QT_VERSION_MAJOR} EQUAL 6)
        LIST(APPEND SOURCES QZXingFilterVideoSink.cpp QZXingFilterVideoSink.h)
    else()
        LIST(APPEND SOURCES QZXingFilter.cpp QZXingFilter.h)
    endif()
    add_definitions(-DQZXING_MULTIMEDIA)

    SET(QZXING_USE_QML ON)

endif(QZXING_MULTIMEDIA)

if(QZXING_USE_QML)
    LIST(APPEND SOURCES QZXingImageProvider.cpp QZXingImageProvider.h)
    add_definitions(-DQZXING_QML)
endif(QZXING_USE_QML)

if(QZXING_USE_ENCODER)
    add_definitions(-DENABLE_ENCODER_GENERIC -DENABLE_ENCODER_QR_CODE)
endif(QZXING_USE_ENCODER)

add_library(qzxing "" ${SOURCES})

if(WIN32)
    add_subdirectory(zxing/win32)
    if(MSVC)
        add_definitions(-D__STDC_LIMIT_MACROS)
    endif(MSVC)
else()
    add_definitions(-DNO_ICONV)
endif(WIN32)

if(VS_WINRT_COMPONENT)
    add_definitions(-DNO_ICONV)
endif(VS_WINRT_COMPONENT)

add_subdirectory(zxing/bigint)

add_subdirectory(zxing/zxing)

target_link_libraries(qzxing Qt${QT_VERSION_MAJOR}::Core Qt${QT_VERSION_MAJOR}::Gui)

if(QZXING_MULTIMEDIA)
    target_link_libraries(qzxing Qt${QT_VERSION_MAJOR}::Multimedia)
    target_compile_definitions(qzxing PUBLIC -DQZXING_MULTIMEDIA)
endif(QZXING_MULTIMEDIA)

if(QZXING_USE_QML)
    target_link_libraries(qzxing
        Qt${QT_VERSION_MAJOR}::Svg
        Qt${QT_VERSION_MAJOR}::Quick
        Qt${QT_VERSION_MAJOR}::QuickControls2)
    target_compile_definitions(qzxing PUBLIC -DQZXING_QML)
endif(QZXING_USE_QML)

if(QZXING_USE_ENCODER)
    target_compile_definitions(qzxing PUBLIC -DENABLE_ENCODER_GENERIC -DENABLE_ENCODER_QR_CODE)
endif(QZXING_USE_ENCODER)

if(QZXING_USE_DECODER_QR_CODE)
    target_compile_definitions(qzxing PRIVATE -DENABLE_DECODER_QR_CODE)
endif()

if(QZXING_USE_DECODER_1D_BARCODES)
    target_compile_definitions(qzxing PRIVATE -DENABLE_DECODER_1D_BARCODES)
endif()

if(QZXING_USE_DECODER_DATA_MATRIX)
    target_compile_definitions(qzxing PRIVATE -DENABLE_DECODER_DATA_MATRIX)
endif()

if(QZXING_USE_DECODER_AZTEC)
    target_compile_definitions(qzxing PRIVATE -DENABLE_DECODER_AZTEC)
endif()

if(QZXING_USE_DECODER_PDF17)
    target_compile_definitions(qzxing PRIVATE -DENABLE_DECODER_PDF17)
endif()

if(QZXING_USE_DECODER_1D_BARCODES)
    target_compile_definitions(qzxing PRIVATE -DENABLE_DECODER_1D_BARCODES)
endif()


# Change Global Definitions depending on how you want to use the library
target_compile_definitions(qzxing PUBLIC DISABLE_LIBRARY_FEATURES)

# Target includes
target_include_directories(qzxing
  PRIVATE
        .
        zxing
        zxing/win32/zxing
        zxing/zxing
        zxing/bigint
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

