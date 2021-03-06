# Copyright (c) 2018 the Talvos developers. All rights reserved.
#
# This file is distributed under a three-clause BSD license. For full license
# terms please see the LICENSE file distributed with this source code.

foreach(test
  errors/device-load-invalid
  errors/device-store-invalid
  errors/invocation-load-invalid
  errors/invocation-store-invalid
  errors/missing-ds-entry
  errors/workgroup-load-invalid
  errors/workgroup-store-invalid
  misc/jacobi
  misc/multiple-entry-points
  misc/nbody
  misc/reduce
  misc/vecadd
  misc/vecadd_binary
  spirv/bitcast
  spirv/composite-extract
  spirv/constant-composite
  spirv/function-call
  spirv/group-builtins
  spirv/phi-swap
  spirv/simple-branch
  spirv/simple-loop
  spirv/spec-constants
  spirv/spec-constant-workgroupsize
  spirv/struct-offset
  spirv/test-bitwise-instructions
  spirv/test-fp-arithmetic
  spirv/test-fp-comparisons
  spirv/test-integer-arithmetic
  spirv/test-integer-comparisons
  spirv/test-logical-instructions
  spirv/test-op-helpers
  talvos-cmd/binfile
  talvos-cmd/binfile-too-short
  talvos-cmd/dispatch-without-module
  talvos-cmd/duplicate-allocation-name
  talvos-cmd/empty
  talvos-cmd/endloop-without-loop
  talvos-cmd/invalid-resource-name
  talvos-cmd/invalid-spec-id
  talvos-cmd/invalid-vector-suffix
  talvos-cmd/loop-count-zero
  talvos-cmd/missing-binfile
  talvos-cmd/parse-failure
  talvos-cmd/unexpected-eof
  talvos-cmd/unterminated-loop
  talvos-cmd/wrong-specialize-size
)
  add_test(
    NAME ${test}
    COMMAND
    ${PYTHON_EXECUTABLE} ${CMAKE_SOURCE_DIR}/test/run-test.py
    $<TARGET_FILE:talvos-cmd>
    ${CMAKE_CURRENT_SOURCE_DIR}/${test}.tcf
  )
endforeach(${test})

add_subdirectory(interactive)
add_subdirectory(plugins)
add_subdirectory(runtime)
