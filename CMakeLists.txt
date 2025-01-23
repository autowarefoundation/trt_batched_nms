cmake_minimum_required(VERSION 3.17)
project(trt_batched_nms)

find_package(autoware_tensorrt_common)
if (NOT ${autoware_tensorrt_common_FOUND})
    message(WARNING "The autoware_tensorrt_common package is not found. Please check its dependencies.")
    return()
endif ()

find_package(autoware_cmake REQUIRED)
autoware_package()
add_compile_options(-Wno-deprecated-declarations)

include(CheckLanguage)
check_language(CUDA)
if (CMAKE_CUDA_COMPILER)
    enable_language(CUDA)
else ()
    message(WARNING "CUDA is not found. preprocess acceleration using CUDA will not be available.")
endif ()

find_library(NVINFER_LIB nvinfer PATHS ${TENSORRT_ROOT}/lib)
find_library(NVPLUGIN_LIB nvinfer_plugin PATHS ${TENSORRT_ROOT}/lib)

include_directories(
        include
)

cuda_add_library(${PROJECT_NAME}_plugin SHARED
        src/nms/batched_nms_kernel.cpp
        src/nms/kernel.cu
        src/nms/allClassNMS.cu
        src/nms/gatherNMSOutputs.cu
        src/nms/permuteData.cu
        src/nms/sortScoresPerClass.cu
        src/nms/sortScoresPerImage.cu
        src/trt_batched_nms.cpp
)

target_link_libraries(${PROJECT_NAME}_plugin ${CUDA_LIBRARIES} ${NVINFER_LIB} ${NVPLUGIN_LIB} ${OpenCV_LIBS} dl)

install(TARGETS ${PROJECT_NAME}_plugin
        LIBRARY DESTINATION lib
)

if (BUILD_TESTING)
    find_package(ament_lint_auto REQUIRED)
    ament_lint_auto_find_test_dependencies()
endif ()

ament_package()