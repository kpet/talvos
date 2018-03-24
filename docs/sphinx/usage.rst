Usage
=====
.. highlight:: bash

Vulkan runtime
--------------
Talvos provides an implementation of the Vulkan API which allows existing
Vulkan applications to make use of its dynamic analysis tools without requiring
source code modifications.
To use the Talvos Vulkan implementation, simply link the application against
the ``talvos-vulkan`` library (e.g. ``libtalvos-vulkan.so`` or
``talvos-vulkan.lib``) instead of the Vulkan loader library.
Alternatively on Unix systems, the ``LD_PRELOAD`` mechanism can be used to
force a Vulkan application to use Talvos without requiring it to be relinked:
::

  # Linux - assumes Talvos lib/ directory is on LD_LIBRARY_PATH
  LD_PRELOAD=libtalvos-vulkan.so ./app

  # macOS - assumes Talvos lib/ directory is on DYLD_LIBRARY_PATH
  DYLD_INSERT_LIBRARIES=libtalvos-vulkan.dylib DYLD_FORCE_FLAT_NAMESPACE=1 ./app


talvos-cmd
----------
In addition to the Vulkan runtime interface, a simple command-line tool is
provided which allows SPIR-V shaders to be executed without requiring any
Vulkan host code.
This involves writing a short command file that describes the way in which a
shader should be executed (:ref:`see here for full command file syntax
<talvos-cmd>`).
A Talvos command file can then be executed with this simple command:
::

  $ talvos-cmd foo.tcf


Errors
------
.. highlight:: none

If Talvos detects any errors while executing SPIR-V shaders, it will print
details to ``stderr``, including information about which entry point,
invocation, and SPIR-V instruction caused the problem.
For example, accessing device memory beyond the extent of a storage buffer
produces an error message like this:
::

  Invalid load of 4 bytes from address 0x200000000003c (Device scope)
      Entry point: %1 vecadd
      Invocation: Global(15,0,0) Local(0,0,0) Group(15,0,0)
        %29 = OpLoad %12 %28

If a device-scope variable is used without the application providing a
corresponding descriptor binding, the following error will be produced:
::

  Invalid base pointer for descriptor set entry (0,1)
      Entry point: %1 vecadd
      Invocation: Global(0,0,0) Local(0,0,0) Group(0,0,0)
        %28 = OpAccessChain %13 %10 %21 %25


Interactive SPIR-V execution
----------------------------
Talvos provides a simple interactive debugging interface that enables stepping
through the execution of a SPIR-V shader.
To enable the interactive debugger, set the environment variable
``TALVOS_INTERACTIVE=1``.
When a SPIR-V shader begins executing, Talvos will drop to an interactive
GDB-style prompt.
Type ``help`` to see a list of available commands.
Using ``step`` will advance the current invocation by a single SPIR-V
instruction, and ``print %<id>`` will display the value of the SPIR-V object
with the specified ID.
The ``switch`` command can be used to change to a different invocation.
An example interactive debugging session demonstrating these commands is shown
below:
::

  $ TALVOS_INTERACTIVE=1 talvos-cmd nbody.tcf
    %62 = OpAccessChain %21 %12 %31
  (talvos) step
    %63 = OpLoad %19 %62
  ...
  (talvos) step
    %71 = OpLoad %16 %70
  (talvos) step
    %72 = OpIEqual %29 %63 %31

  (talvos) print %16
    %16 = float32v4
  (talvos) print %71
    %71 = {86.52, 0, -94.33, 1}

  (talvos) switch 7 0 0
  Switched to invocation with global ID (7,0,0)
    %62 = OpAccessChain %21 %12 %31
  (talvos) step
    %63 = OpLoad %19 %62
  ...
    %72 = OpIEqual %29 %63 %31
  (talvos) step
    %73 = OpLogicalNot %29 %72

  (talvos) print %29
    %29 = bool
  (talvos) print %72
    %72 = false
  (talvos) continue

While running with ``continue``, the interactive debugger will automatically
break into a prompt whenever Talvos produces an error, allowing the user to
inspect the state of the current invocation:
::

  $ TALVOS_INTERACTIVE=1 talvos-cmd device-load-invalid.tcf
    %24 = OpAccessChain %19 %2 %21
  (talvos) continue

  Invalid load of 4 bytes from address 0x200000000003c (Device scope)
      Entry point: %1 vecadd
      Invocation: Global(15,0,0) Local(0,0,0) Group(15,0,0)
        %29 = OpLoad %12 %28

    %29 = OpLoad %12 %28
  (talvos) print %28
    %28 = 0x200000000003c
  (talvos) print %12
    %12 = int32