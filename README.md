# ArmSmmu4K

ARM SMMU translation with 4 KiB pages.

used by the GPU and storage SMMU paths on sheng.

`ArmSmmu4KContextConfig.coherent_walk = 1` selects outer-shareable WBWA table
reads. The platform supplies CPU write-back table memory and publishes its
writes through the visibility callback before device access. A coherent walker
needs store ordering in that callback; TLB invalidation and synchronization
still follow the live-update API's rules.

`ArmSmmu4KLiveUpdateContextRanges` updates the supplied ranges after the caller
has audited the initial context and checked the new ranges against its existing
objects. It retains leaf counts in `ArmSmmu4KLiveState`, checks the changed
descriptors before and after writing them, publishes their table pages, and
synchronizes the ASID on unmap. The complete ownership and serialization
requirements are documented in `ArmSmmu4KLive.h`. Optional timing totals cover
successful calls and are read under the caller's context lock.

Live arenas accept `ArmSmmu4KTablePages`, which allocates intermediate tables
when a mapping first needs them and locates pages by physical address. A
detached process table retains the bank offset for ASID invalidation while
`ttbr0_switched` permits the GPU to install other process roots in that bank.
The caller releases an ASID only after GPU execution has switched away and its
remaining translations have been synchronized. Small unmaps invalidate their
pages; larger unmaps invalidate the ASID.

Build and interface details: [documentation](https://unixnu.com/reference/components). Component sources and dependencies: [catalog](https://unixnu.com/reference/components).
