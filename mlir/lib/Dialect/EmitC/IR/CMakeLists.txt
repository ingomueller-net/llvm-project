include_guard(GLOBAL)

set(ABSL_PROPAGATE_CXX_STD ON)
set(CMAKE_INCLUDE_CURRENT_DIR OFF)

if(${LLVM_INCLUDE_TESTS})
  message(FATAL_ERROR "LLVM_INCLUDE_TESTS is enabled but breaks the build due "
                      "to a target name clash with 'abseil-cpp'. Please run "
                      "cmake again with '-DLLVM_INCLUDE_TESTS=OFF'.")
endif()

include(FetchContent)
FetchContent_Declare(GTest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG v1.14.0
        OVERRIDE_FIND_PACKAGE
)
FetchContent_Declare(Protobuf
        GIT_REPOSITORY https://github.com/protocolbuffers/protobuf.git
        GIT_TAG v28.2
        SYSTEM
        OVERRIDE_FIND_PACKAGE
)
set(protobuf_BUILD_TESTS OFF CACHE INTERNAL "")
# Disable warnings for dependency targets.
set(protobuf_BUILD_TESTS OFF CACHE INTERNAL "")
if(MSVC)
  add_compile_options("/W0")
else()
  add_compile_options("-w")
endif()
FetchContent_MakeAvailable(Protobuf GTest)


add_mlir_dialect_library(MLIREmitCDialect
  EmitC.cpp

  ADDITIONAL_HEADER_DIRS
  ${MLIR_MAIN_INCLUDE_DIR}/mlir/Dialect/EmitC

  DEPENDS
  MLIREmitCIncGen
  MLIREmitCAttributesIncGen

  LINK_LIBS PUBLIC
  MLIRCallInterfaces
  MLIRCastInterfaces
  MLIRControlFlowInterfaces
  MLIRFunctionInterfaces
  MLIRIR
  MLIRSideEffectInterfaces
  )

llvm_add_library(MLIREmitCDialect2
  EmitC.cpp

  OBJECT           # <-- problem goes away if this is commented out
  # SHARED STATIC  # <-- same problem if this is commented in
  ADDITIONAL_HEADER_DIRS
  ${MLIR_MAIN_INCLUDE_DIR}/mlir/Dialect/EmitC

  DEPENDS
  MLIREmitCIncGen
  MLIREmitCAttributesIncGen

  LINK_LIBS PUBLIC
  MLIRCallInterfaces
  MLIRCastInterfaces
  MLIRControlFlowInterfaces
  MLIRFunctionInterfaces
  MLIRIR
  MLIRSideEffectInterfaces
  protobuf::libprotobuf
  )
