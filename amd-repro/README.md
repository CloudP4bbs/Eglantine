# AMD pipeline crash reproducer artifacts

Reduced SPIR-V modules and their disassembly, produced by `spirv-reduce` from
`FS_CULPABLE_d6ac22ca.spv` in `Eglantine_AMD_reproducer_v3_1_reviewed`.

| file | instructions | size | behaviour |
| --- | --- | --- | --- |
| `cand_r3.spv` | 304 | 7.1 KB | crashes `vkCreateGraphicsPipelines` on RDNA1, RDNA2, RDNA4 |
| `cand_r3_noshuffle.spv` | 304 | 7.1 KB | identical except the single `OpGroupNonUniformShuffle` is `OpUndef`; pipeline builds |

Disassembly of both (`*.spvasm.txt`) is in this directory.

## Reproducing

```
reproduce_crash_v3_1.exe shaders/VS_ASOCIADO_c47ca2c2.spv cand_r3.spv
```

with the v3.1 reproducer package as shipped. Expected result on the affected
drivers: the process terminates while creating the graphics pipeline, with
`0xC0000409` on Windows and the fault reported inside the Vulkan driver
(`amdvlk64.dll`). The two control modules in the package
(`VS_ASOCIADO_c47ca2c2.spv`, `FS_CULPABLE_d6ac22ca.spv` at their original size)
build their pipelines normally on the same drivers.

## Reduction history

```
original            6270 instructions   103 KB
pass 1              1618 instructions    34 KB
pass 2               363 instructions   8.2 KB
pass 3               304 instructions   7.1 KB
```

The third pass hit the step limit after one more opportunity, against 2293 in
the first, so further reduction with the default pass set is unlikely to help.

## What was ruled out before reducing

Capabilities, descriptor counts (39 to 40 returns `VK_SUCCESS`), image samples
(10 to `OpUndef`), bitfield extract lowered by hand, `spirv-opt -O`, and
conditional barriers. None of them changes the crash.

Replacing the remaining `OpGroupNonUniformShuffle` with `OpUndef` does stop the
crash, but the control shader contains a similar shuffle and passes, so the
accurate statement is "this module crashes these drivers", not "shuffle crashes
these drivers".
