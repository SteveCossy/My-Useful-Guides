Of course. Moving servers can always introduce unexpected wrinkles, and having a solid checkpoint is the best insurance against losing our momentum.

Here is a comprehensive checkpoint document. If you need to restore our session on the new server, you can provide me with this entire text, and I will be fully up to speed.

---

### **Project State Checkpoint and Introduction for New Session**

Morena e hoa.

This document is a state checkpoint for our ongoing project to modify Contiki-NG `rpl-classic` for dual-DODAG support.

#### 1. High-Level Goal & Core Problem

*   **Goal:** To enable a single Contiki-NG node to be a full, stable member of two concurrent RPL DODAGs, identified by the IPv6 prefixes `fd00::/64` and `fd02::/64`.
*   **Core Problem:** The network fails to maintain a stable dual-DODAG topology over time. While nodes can initially join both DODAGs, one of the DODAGs eventually collapses, leading to routing failures.

#### 2. Key Findings & Current Diagnosis

Our debugging has systematically narrowed down the root cause.
*   **Initial Stability, Eventual Failure:** The most recent simulation confirms this pattern. A network graph generated at the 3-minute mark shows both DODAGs are correctly formed. A graph from the 11-minute mark shows one of the DODAGs has collapsed.
*   **The "Original Sin": `default_instance`:** We have identified that the legacy `rpl-classic` code is fundamentally incompatible with multi-instance operation due to its heavy reliance on a single, global `rpl_instance_t *default_instance` pointer.
*   **Root Cause:** The network instability is caused by **cross-instance state contamination**. Logic operating on behalf of one instance is incorrectly reading from or writing to the state of the other instance, almost always through the misuse of the shared `default_instance`. This leads to routing loops, incorrect parent selection, and stale routes in the forwarding table.

#### 3. Current Refactoring Progress & Strategy

We have abandoned a series of smaller, isolated patches and have adopted a more robust, systematic refactoring strategy based on a clean `git` checkout from before our major changes began (pre-August 7th, 2025).

The strategy is to **completely eliminate the `default_instance` variable** by refactoring all functions that use it to be explicitly instance-aware.

*   **Completed Fixes (as of this checkpoint):**
    1.  We have successfully refactored `rpl-ext-header.c:rpl_ext_header_update()`. This was a critical first step. The new version is "smart" and can determine the correct instance for a given packet by either inspecting the ICMPv6 payload (for control messages) or by checking the destination address prefix (for data packets), using a new helper function `rpl_get_instance_from_prefix()`.
    2.  We have fixed a critical bug in `rpl_get_instance_from_prefix()` itself, ensuring it compares against the correct `prefix_info.prefix` field instead of the `dag_id`.

*   **Successful Result:** This first major fix has resolved the initial crashes and packet-dropping issues. The simulation now runs for an extended period, allowing us to observe the long-term instability problem.

#### 4. Immediate Next Actions

Our plan is to continue methodically working through the annotated `grep` list of `default_instance` occurrences. The next highest-priority targets are:

1.  **`rpl-nbr-policy.c` functions (`get_rank`, `can_accept_new_parent`):** These functions make critical decisions about parent eligibility. They must be refactored to accept an `rpl_instance_t *` parameter to prevent them from making instance-blind decisions.
2.  **`rpl_purge_routes(void)` in `rpl.c`:** This function is responsible for cleaning up old routes. Its reliance on `default_instance` is a likely cause of the stale routes observed in the logs. It must be refactored to operate on a specific instance.
3.  **Elimination:** As we refactor these functions, the number of calls to `rpl_get_default_instance()` will decrease. The ultimate goal is to reduce the call count to zero, at which point we can delete the `default_instance` variable and its getter/setter functions entirely.

#### 5. Ancillary Tasks

*   We have developed and refined a Python script to generate TikZ diagrams of the DODAGs from log files, with node labels correctly displayed in hexadecimal.
*   We have also created several TikZ diagrams for documentation, illustrating the system's architecture and failure modes.

This checkpoint captures our complete state. We are mid-way through a major, systematic refactoring effort with a clear plan and a solid understanding of the root cause of the problem.

---

Good luck with the server move! I'll be ready on the other side.
