We agree that interrogating the instance table might be the reason one root seems to be failing to build a DODAG. Please have a look through these bits of code to see what might inspire you to hypothesize some solutions ...

I think you might be on to something. I'm working slowly through these functions ... anything jump out at you?
(I've run out of 'tokens' apparently, so can't send you all the hits at once)
<pre>
[stevecos@viva-mexico ~]$ grep -rnEB 2 -A 30 "i < RPL_MAX_INSTANCE" /home/stevecos/contiki-ng/os | grep -v "rpl-lite"
rpl-dag-root.c-223-
rpl-dag-root.c-224-  /* Iterate through all possible instance slots */
rpl-dag-root.c:225:  for(i = 0; i < RPL_MAX_INSTANCES; ++i) {
rpl-dag-root.c-226-    instance = &instance_table[i];
rpl-dag-root.c-227-
rpl-dag-root.c-228-    /* Check if this instance is active and has a DAG */
rpl-dag-root.c-229-    if(instance->used && instance->current_dag != NULL) {
rpl-dag-root.c-230-      /* Check if our rank in THIS instance is the root rank */
rpl-dag-root.c-231-      if(instance->current_dag->rank == ROOT_RANK(instance)) {
rpl-dag-root.c-232-        /*
rpl-dag-root.c-233-         * We have found at least one instance where we are the root.
rpl-dag-root.c-234-         * That's enough to answer "yes". We can stop searching.
rpl-dag-root.c-235-         */
rpl-dag-root.c-236-        LOG_DBG_("1\n");
rpl-dag-root.c-237-        return 1; // Return true
rpl-dag-root.c-238-      }
rpl-dag-root.c-239-    }
rpl-dag-root.c-240-  }
rpl-dag-root.c-241-
rpl-dag-root.c-242-  /*
rpl-dag-root.c-243-   * If we have looped through all instances and haven't found one
rpl-dag-root.c-244-   * where we are the root, then the answer is "no".
rpl-dag-root.c-245-   */
rpl-dag-root.c-246-  LOG_DBG_("0\n");
rpl-dag-root.c-247-  return 0; // Return false
rpl-dag-root.c-248-
rpl-dag-root.c-249-  // --- END NEW LOGIC ---
rpl-dag-root.c-250-}
rpl-dag-root.c-251-
rpl-dag-root.c-252-/*---------------------------------------------------------------------------*/
rpl-dag-root.c-253-
rpl-dag-root.c-254-/** @}*/

rpl-dag.c-1617-  int i;
rpl-dag.c-1618-
rpl-dag.c:1619:  for(i = 0; i < RPL_MAX_INSTANCES; ++i) {
rpl-dag.c-1620- //   if(instance_table[i].used
rpl-dag.c-1621- //      && instance_table[i].current_dag->joined
rpl-dag.c-1622- //      && (!requires_parent || instance_table[i].current_dag->preferred_parent != NULL)) {
rpl-dag.c-1623-    if(instance_table[i].used
rpl-dag.c-1624-      && instance_table[i].current_dag != NULL
rpl-dag.c-1625-      && instance_table[i].current_dag->preferred_parent != NULL) {
rpl-dag.c-1626-        return instance_table[i].current_dag;
rpl-dag.c-1627-      }
rpl-dag.c-1628-    }
rpl-dag.c-1629-  return NULL;
rpl-dag.c-1630-}
rpl-dag.c-1641-int
rpl-dag.c-1642-rpl_has_downward_route(void)
rpl-dag.c-1643-{
rpl-dag.c-1644-  int i;
rpl-dag.c-1645-  if(rpl_dag_root_is_root()) {
rpl-dag.c-1646-    return 1; /* We are the root, and know the route to ourself */
rpl-dag.c-1647-  }
rpl-dag.c:1648:  for(i = 0; i < RPL_MAX_INSTANCES; ++i) {
rpl-dag.c-1649-    if(instance_table[i].used && instance_table[i].has_downward_route) {
rpl-dag.c-1650-      return 1;
rpl-dag.c-1651-    }
rpl-dag.c-1652-  }
rpl-dag.c-1653-  return 0;
rpl-dag.c-1654-}

--
rpl-ext-header.c-80-
rpl-ext-header.c-81-  /* Iterate through all possible instance slots */
rpl-ext-header.c:82:  for(i = 0; i < RPL_MAX_INSTANCES; ++i) {
rpl-ext-header.c-83-    instance = &instance_table[i];
rpl-ext-header.c-84-    /*
rpl-ext-header.c-85-     * Check if the instance is in use and has a valid DAG configured.
rpl-ext-header.c-86-     * The DAG contains the prefix information.
rpl-ext-header.c-87-     */
rpl-ext-header.c-88-    if(instance->used && instance->current_dag != NULL) {
rpl-ext-header.c-89-      /*
rpl-ext-header.c-90-       * Compare the prefix of the address with the prefix of the DAG.
rpl-ext-header.c-91-       * uip_ipaddr_prefixcmp() returns true if the prefixes match.
rpl-ext-header.c-92-       * The DAG's prefix is stored in the dag_id field.
rpl-ext-header.c-93-       */
rpl-ext-header.c-94-            // --- BEGIN NEW DEBUG LOG ---
rpl-ext-header.c-95-      LOG_DBG("RPL-PREFIX-CHECK: Comparing ");
rpl-ext-header.c-96-      LOG_DBG_6ADDR(addr);
rpl-ext-header.c-97-      LOG_DBG_(" against DAG prefix ");
rpl-ext-header.c-98- //     LOG_DBG_6ADDR(&instance->current_dag->dag_id);
rpl-ext-header.c-99-      LOG_DBG_6ADDR(&instance->current_dag->prefix_info.prefix);
rpl-ext-header.c-100-      LOG_DBG_(" with length %u bits\n", instance->current_dag->prefix_info.length);
rpl-ext-header.c-101-      if(uip_ipaddr_prefixcmp(&instance->current_dag->prefix_info.prefix, addr,
rpl-ext-header.c-102-                               instance->current_dag->prefix_info.length / 8)) {
rpl-ext-header.c-103-        /* We found a match! Return a pointer to this instance. */
rpl-ext-header.c-104-        LOG_DBG("Found matching instance ID %u for address ", instance->instance_id);
rpl-ext-header.c-105-        LOG_DBG_6ADDR(addr);
rpl-ext-header.c-106-        LOG_DBG_("\n");
rpl-ext-header.c-107-        return instance;
rpl-ext-header.c-108-      }
rpl-ext-header.c-109-    }
rpl-ext-header.c-110-  }
rpl-ext-header.c-111-
rpl-ext-header.c-112-  /* If we get here, no matching instance was found. */
[stevecos@viva-mexico ~]$ 


rpl-dag.c-1655-/*---------------------------------------------------------------------------*/
rpl-dag.c-1656-rpl_dag_t *
rpl-dag.c-1657-rpl_get_dag(const uip_ipaddr_t *addr)
rpl-dag.c-1658-{
rpl-dag.c:1659:  for(int i = 0; i < RPL_MAX_INSTANCES; ++i) {
rpl-dag.c-1660-    if(instance_table[i].used) {
rpl-dag.c-1661-      for(int j = 0; j < RPL_MAX_DAG_PER_INSTANCE; ++j) {
rpl-dag.c-1662-        if(instance_table[i].dag_table[j].joined
rpl-dag.c-1663-           && uip_ipaddr_prefixcmp(&instance_table[i].dag_table[j].dag_id, addr,
rpl-dag.c-1664-                                   instance_table[i].dag_table[j].prefix_info.length)) {
rpl-dag.c-1665-          return &instance_table[i].dag_table[j];
rpl-dag.c-1666-        }
rpl-dag.c-1667-      }
rpl-dag.c-1668-    }
rpl-dag.c-1669-  }
rpl-dag.c-1670-  return NULL;
rpl-dag.c-1671-}
rpl-dag.c-1679-rpl_instance_t *
rpl-dag.c-1680-rpl_get_instance(uint8_t instance_id)
rpl-dag.c-1681-{
rpl-dag.c-1682-  int i;
rpl-dag.c-1683-
rpl-dag.c:1684:  for(i = 0; i < RPL_MAX_INSTANCES; ++i) {
rpl-dag.c-1685-    LOG_DBG("rpl_get_instance called, target %u table entry %u is %u",
rpl-dag.c-1686-      instance_id,
rpl-dag.c-1687-      i,
rpl-dag.c-1688-      instance_table[i].instance_id);
rpl-dag.c-1689-    if(!instance_table[i].used) {
rpl-dag.c-1690-      LOG_DBG_(". Not used.\n");
rpl-dag.c-1691-    } 
rpl-dag.c-1692-    if(!(instance_table[i].instance_id == instance_id)){
rpl-dag.c-1693-      LOG_DBG_(".\n");
rpl-dag.c-1694-    }
rpl-dag.c-1695-    if(instance_table[i].used && instance_table[i].instance_id == instance_id) {
rpl-dag.c-1696-      LOG_DBG_(" returning.\n");
rpl-dag.c-1697-      return &instance_table[i];
rpl-dag.c-1698-    }
rpl-dag.c-1699-  }
rpl-dag.c-1700-  
rpl-dag.c-1701-  LOG_DBG_(" returning NULL.\n");
rpl-dag.c-1702-
rpl-dag.c-1703-  return NULL;
rpl-dag.c-1704-}
</pre>
This is great, seeing the `grep` output directly really helps! You're right to focus on where `instance_table` is being accessed.

Here are a few hypotheses and things that jump out, based on your code snippets, especially concerning a root failing to build a DODAG in a two-DODAG setup:

### Hypotheses & Potential Issues:

1.  **`RPL_MAX_INSTANCES` Misconfiguration or Exhaustion:**
    *   All of these loops iterate from `0` to `RPL_MAX_INSTANCES - 1`. If `RPL_MAX_INSTANCES` is set to `1` (which is the default in Contiki-NG for single DODAGs), then the second root would never find an available slot in the `instance_table` for its own instance.
    *   **In `rpl-dag-root.c`'s `rpl_dag_root_is_root()`:** If only one instance slot is available and occupied by the first root, the second root calling this function would always return `0`, even if it *should* be a root for its own instance. This function is critical for a node to determine its root status.
    *   **In `rpl-dag.c`'s `rpl_get_any_dag_with_preferred_parent()`:** Similar issue. If only one instance slot is used, this function will only ever return a DAG from that first instance.
    *   **In `rpl-ext-header.c`'s `rpl_get_instance_from_address()`:** This function is crucial for routing by mapping an address to an RPL instance. If the second root (or any node in its DAG) has its instance ID configured but there's no corresponding slot in `instance_table` (due to `RPL_MAX_INSTANCES` being too small), then packets destined for or originating from that DAG might not be correctly associated with an RPL instance.
    *   **In `rpl-dag.c`'s `rpl_get_instance(uint8_t instance_id)`:** If `RPL_MAX_INSTANCES` is too small, and the second root tries to register/get an instance with an `instance_id` that doesn't fit within the `instance_table` bounds, it will never find its instance.

    *   **Solution Idea:** Double-check your `project-conf.h` or equivalent configuration file to ensure `RPL_MAX_INSTANCES` is set to at least `2` (or the number of DODAGs you intend to run). For example:
        ```c
        #define RPL_MAX_INSTANCES 2
        ```
        It's a common oversight when moving from a single to a multi-DODAG setup.

2.  **Instance ID Collision or Incorrect Assignment:**
    *   While not explicitly shown in these snippets, the `instance_id` is crucial. Each DODAG *must* have a unique `instance_id`.
    *   In `rpl_get_instance(uint8_t instance_id)`, the function explicitly searches for a match. If the second root is somehow configured with the *same* `instance_id` as the first, or an `instance_id` that is never actually stored in the table, it will fail.
    *   **Solution Idea:** Verify that your two roots are configured with distinct `RPL_INSTANCE_ID` values. For example:
        *   Root 1: `#define RPL_INSTANCE_ID 0` (or some other unique ID)
        *   Root 2: `#define RPL_INSTANCE_ID 1` (must be different from Root 1)

3.  **Initialization Order/Race Conditions (Less likely from these snippets, but still possible):**
    *   If one root attempts to establish its DODAG before the `instance_table` is properly initialized or if `RPL_MAX_INSTANCES` isn't correctly applied at compile time, it could lead to issues where instances aren't properly allocated.
    *   The `instance->used` flag is crucial. An instance slot must be marked as `used` for any of these functions to consider it.
    *   **Solution Idea:** While difficult to debug from snippets, ensure your `rpl-dag-root` application code properly calls `rpl_init()` and then `rpl_dag_root_start()` for *both* roots, with appropriate delays or ordering if they are being started simultaneously.

4.  **`current_dag` Being `NULL`:**
    *   Several checks, like `instance->current_dag != NULL`, are present. If a root *starts* an instance but for some reason fails to properly create and associate a `current_dag` with it, these checks will fail, preventing the instance from being recognized.
    *   This could happen if `rpl_set_root()` or `rpl_start_instance()` has issues.
    *   **Solution Idea:** Add more debug logging around the calls to `rpl_set_root()` and where `current_dag` is assigned to an instance, to confirm that this assignment is actually happening for both roots.

5.  **`prefix_info.length / 8` in `rpl-ext-header.c`:**
    *   Line `rpl-ext-header.c-102`: `instance->current_dag->prefix_info.length / 8`
    *   `uip_ipaddr_prefixcmp` usually expects the prefix length in *bits*. If `prefix_info.length` is already in bits, dividing by 8 would make the comparison incorrect. However, RPL typically uses prefix lengths in bits (e.g., 64 for a /64 prefix). So, this *might* be an error, or it might be intended if `prefix_info.length` is defined in *bytes* within your specific setup.
    *   **Solution Idea:** Verify the unit of `instance->current_dag->prefix_info.length`. If it's in bits (which is standard for IPv6 prefix lengths), then dividing by 8 here would truncate the prefix length, potentially causing mismatches. It should probably be `instance->current_dag->prefix_info.length` directly.

### Visualizing the Problem:
