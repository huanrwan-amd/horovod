message(STATUS "Building Horovod for ROCm")

# Set HIP_PATH if not already defined
if (NOT DEFINED HIP_PATH)
  if (DEFINED ENV{HIP_PATH})
    set(HIP_PATH $ENV{HIP_PATH} CACHE PATH "Path where HIP is installed")
  else()
    set(HIP_PATH "/opt/rocm" CACHE PATH "Path where HIP is set by default")
  endif()
endif()

# Set ROCM_PATH, default to /opt/rocm
if (NOT DEFINED ROCM_PATH)
  set(ROCM_PATH "/opt/rocm")
endif()

# Add ROCm-related paths
list(APPEND CMAKE_PREFIX_PATH "${HIP_PATH}/hip" "${HIP_PATH}")
set(HIP_CLANG_PATH "${ROCM_PATH}/llvm/bin")
set(CMAKE_MODULE_PATH "${HIP_PATH}/cmake" ${CMAKE_MODULE_PATH})

# Set HIP compiler flags
set(HIP_HIPCC_FLAGS -D__HIP_PLATFORM_HIPCC__=1 -fPIC)
list(APPEND HIP_HIPCC_FLAGS_RELEASE -O3 -fPIC)
list(APPEND HIP_HIPCC_FLAGS_DEBUG -G -fPIC)
list(APPEND HIP_HIPCC_FLAGS -D_GLIBCXX_USE_CXX11_ABI=1 -fPIC)

# Find HIP package
find_package(HIP QUIET REQUIRED)

# Compile HIP kernels into static libraries
# Library with CXX11 ABI
add_library(horovod_hip_kernels STATIC hip_kernels.cu)
target_compile_options(horovod_hip_kernels PRIVATE ${HIP_HIPCC_FLAGS})
target_compile_definitions(horovod_hip_kernels PRIVATE _GLIBCXX_USE_CXX11_ABI=1)

# Library compatible with older ABI
hip_add_library(compatible_horovod_hip_kernels STATIC hip_kernels.cu)
target_compile_options(compatible_horovod_hip_kernels PRIVATE ${HIP_HIPCC_FLAGS})
target_compile_definitions(compatible_horovod_hip_kernels PRIVATE _GLIBCXX_USE_CXX11_ABI=0)
