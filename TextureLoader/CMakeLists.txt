cmake_minimum_required (VERSION 3.10)

project(Diligent-TextureLoader CXX)

set(INCLUDE 
    include/dxgiformat.h
    include/pch.h
    include/TextureLoaderImpl.hpp
)

set(INTERFACE
    interface/JPEGCodec.h
    interface/PNGCodec.h
    interface/SGILoader.h
    interface/BCTools.h
    interface/Image.h
    interface/TextureLoader.h
    interface/TextureUtilities.h
)

set(SOURCE 
    src/BCTools.cpp
    src/DDSLoader.cpp
    src/JPEGCodec.c
    src/Image.cpp
    src/KTXLoader.cpp
    src/SGILoader.cpp
    src/PNGCodec.c
    src/STBImpl.cpp
    src/TextureLoaderImpl.cpp
    src/TextureUtilities.cpp
)

add_library(Diligent-TextureLoader STATIC ${SOURCE} ${INCLUDE} ${INTERFACE})
set_common_target_properties(Diligent-TextureLoader)

set_property(SOURCE src/PNGCodec.c src/Image.cpp
APPEND PROPERTY INCLUDE_DIRECTORIES
    "${CMAKE_CURRENT_SOURCE_DIR}/../ThirdParty/libpng" # png_static target does not define any public include directories
    "${CMAKE_CURRENT_BINARY_DIR}/../ThirdParty/libpng" # pnglibconf.h is generated in the binary directory
)

get_target_relative_dir(Diligent-TextureLoader TEXTURE_LOADER_DIR)

target_include_directories(Diligent-TextureLoader
PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/interface>
    $<INSTALL_INTERFACE:include/${TEXTURE_LOADER_DIR}/interface>
PRIVATE
    include
)

source_group("source" FILES ${SOURCE})
source_group("include" FILES ${INCLUDE})
source_group("interface" FILES ${INTERFACE})

if ((${DILIGENT_USE_VCPKG}))
    set(IMG_LIBS ${JPEG_LIBRARIES} PNG::PNG TIFF::TIFF ZLIB::ZLIB)
else()
    set(IMG_LIBS PNG::PNG TIFF::TIFF ZLIB::ZLIB)
endif()

target_link_libraries(Diligent-TextureLoader 
PRIVATE 
    Diligent-BuildSettings 
    Diligent-Common 
    Diligent-PlatformInterface 
    Diligent-GraphicsEngineInterface 
    Diligent-GraphicsAccessories
    Diligent-GraphicsTools
    ${IMG_LIBS} 
)

if (NOT DILIGENT_EXTERNAL_LIBJPEG)
    target_link_libraries(Diligent-TextureLoader
    PRIVATE
        JPEG::JPEG
    )
else()
    target_link_libraries(Diligent-TextureLoader
    PRIVATE
        ${DILIGENT_EXTERNAL_LIBJPEG}
    )
endif()

set_target_properties(Diligent-TextureLoader PROPERTIES
    FOLDER DiligentTools
)

if(DILIGENT_INSTALL_TOOLS)
    install_tools_lib(Diligent-TextureLoader)
endif()

if ((${DILIGENT_USE_VCPKG}))
    install_tools_lib_and_export(Diligent-TextureLoader)
endif()
