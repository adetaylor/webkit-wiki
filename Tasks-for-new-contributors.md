Here are some proposals for tasks that new contributors might want to tackle. They are designed to be self-contained and asynchronous.

Please join the WebKit slack and ping the listed mentor before you start to avoid duplicating work, especially for hard tasks. This would also help you get feedback soon, so that you don't spend time writing code that can't be merged.

# JavaScriptCore (easy)

## Improve disassembly output

Mentor: [@justinmichaud](/justinmichaud) PST

We are missing SIMD arm instructions in our disassembly. This task involves adding those, and making the disassembler output more pretty. 

See `Source/JavaScriptCore/disassembler/ARM64/A64DOpcode.h`.

# JavaScriptCore (hard)

## Implement the Relaxed SIMD proposal (stage 3)

Mentor: [@justinmichaud](/justinmichaud) PST

This task involves adding a new JSC feature flag, implementing the listed instructions in the proposal on ARM and Intel in both WASM JIT tiers, and importing the spec tests / writing your own.
