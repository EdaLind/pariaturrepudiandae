cmake_minimum_required(VERSION 3.9.0)

set (PACKAGE_NAME "react-native-quick-crypto")
set (BUILD_DIR ${CMAKE_SOURCE_DIR}/build)
file (GLOB LIBFBJNI_INCLUDES "${BUILD_DIR}/fbjni-*-headers.jar/")
file (GLOB LIBRN_SO "${BUILD_DIR}/react-native-0*/jni/${ANDROID_ABI}")
set (CMAKE_CXX_STANDARD 17)
# TODO(osp) remove before release
set (CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g")
set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")

include_directories(
        ../cpp
        "${NODE_MODULES_DIR}/react-native/React"
        "${NODE_MODULES_DIR}/react-native/React/Base"
        "${NODE_MODULES_DIR}/react-native/ReactAndroid/src/main/jni"
        "${NODE_MODULES_DIR}/react-native/ReactAndroid/src/main/java/com/facebook/react/turbomodule/core/jni"
        "${NODE_MODULES_DIR}/react-native/ReactCommon"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/callinvoker"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/jsi"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/turbomodule/core"
        "${NODE_MODULES_DIR}/react-native/ReactCommon/react/nativemodule/core"
        "${LIBFBJNI_INCLUDES}"
)

if(${REACT_NATIVE_VERSION} LESS 66)
        set (
                INCLUDE_JSI_CPP
                "${NODE_MODULES_DIR}/react-native/ReactCommon/jsi/jsi/jsi.cpp"
        )
endif()

add_library(reactnativequickcrypto  # <-- Library name
        SHARED
        "src/main/cpp/cpp-adapter.cpp"
        "../cpp/MGLQuickCryptoHostObject.cpp"
        "../cpp/JSIUtils/MGLTypedArray.cpp"
        "../cpp/Utils/MGLDispatchQueue.cpp"
        "../cpp/JSIUtils/MGLThreadAwareHostObject.cpp"
        "../cpp/JSIUtils/MGLSmartHostObject.cpp"
        "../cpp/HMAC/MGLHmacInstaller.cpp"
        "../cpp/HMAC/MGLHmacHostObject.cpp"
        "../cpp/fastpbkdf2/MGLPbkdf2HostObject.cpp"
        "../cpp/fastpbkdf2/fastpbkdf2.c"
        "../cpp/Random/MGLRandomHostObject.cpp"
        "../cpp/Hash/MGLHashInstaller.cpp"
        "../cpp/Hash/MGLHashHostObject.cpp"
        "../cpp/Cipher/MGLCipherHostObject.cpp"
        "../cpp/Cipher/MGLCreateCipherInstaller.cpp"
        "../cpp/Cipher/MGLCreateDecipherInstaller.cpp"
        ${INCLUDE_JSI_CPP} # only on older RN versions
)

set_target_properties(
        reactnativequickcrypto PROPERTIES
        CXX_STANDARD 17
        CXX_EXTENSIONS OFF
        POSITION_INDEPENDENT_CODE ON
)

file (GLOB LIBRN_DIR "${PREBUILT_DIR}/${ANDROID_ABI}")

find_library(
        log-lib
        log
)
find_library(
        REACT_NATIVE_JNI_LIB
        reactnativejni
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)
find_library(
        FBJNI_LIB
        fbjni
        PATHS ${LIBRN_SO}
        NO_CMAKE_FIND_ROOT_PATH
)
find_library(
        TURBOMODULES_LIB
        turbomodulejsijni
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)
find_library(
        REACT_LIB
        react_nativemodule_core
        PATHS ${LIBRN_DIR}
        NO_CMAKE_FIND_ROOT_PATH
)

if(${REACT_NATIVE_VERSION} LESS 66)
        # JSI lib didn't exist on RN 0.65 and before. Simply omit it.
        set (JSI_LIB "")
else()
        # RN 0.66 distributes libjsi.so, can be used instead of compiling jsi.cpp manually.
        find_library(
                JSI_LIB
                jsi
                PATHS ${LIBRN_DIR}
                NO_CMAKE_FIND_ROOT_PATH
        )
endif()

find_package(openssl REQUIRED CONFIG)

target_link_libraries(
        reactnativequickcrypto
        ${TURBOMODULES_LIB}
        ${FBJNI_LIB}
        ${log-lib}
        ${JSI_LIB}
        ${REACT_NATIVE_JNI_LIB}
        ${REACT_LIB}
        android
        openssl::crypto
)
