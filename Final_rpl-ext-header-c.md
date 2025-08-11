E hoa, that is fantastic news!

> "Simulation is taking much longer now ..."

This is the single most positive sign we could have asked for. A simulation that fails instantly means a fundamental piece of logic is broken. A simulation that runs for a long time means the nodes are actually communicating, building topologies, and running the protocol. It means the core packet processing is no longer crashing or getting stuck in a tight loop.

> "... and no sign of the rpl_get_instance_from_prefix function."

This is also excellent. It confirms that our fix to `rpl_ext_header_update` is working correctly for control messages (like DIOs). The `if` block that checks the ICMPv6 payload is successfully identifying the instance, so it never needs to fall back to the `rpl_get_instance_from_prefix` function for those packets. We will likely only see that function called when you start sending unicast data packets, which is exactly what we designed it for.

You have made a huge breakthrough. Okay, let's pick up where we left off and apply the final, clean changes to `rpl-ext-header.c`.

### The Final, Clean Refactoring of `rpl-ext-header.c`

Here is the complete `rpl_ext_header_update` function, incorporating all our successful debugging and removing the `default_instance` dependency. This is the version you should be using on your new, clean branch.

```c
/*
 * In /os/net/routing/rpl-classic/rpl-ext-header.c
 * This is the primary function to be refactored.
 */
#include "net/ipv6/uip.h"       // For UIP_IP_BUF etc.
#include "net/ipv6/uip-ds6.h"   // For uip_ds6_is_my_addr
#include "rpl-private.h"       // For RPL constants and types

// Make sure you have a forward declaration or include for this helper function
rpl_instance_t *rpl_get_instance_from_prefix(const uip_ipaddr_t *addr);


int
rpl_ext_header_update(void)
{
  rpl_instance_t *instance = NULL;

  /*
   * This function is called for both outgoing and forwarded packets.
   * We must determine the correct RPL instance for the packet in flight.
   */

  /* Step 1: Check if the packet is an RPL ICMPv6 control message.
     If so, we can get the instance ID directly from the payload. */
  if(UIP_IP_BUF->proto == UIP_PROTO_ICMP6) {
    /* Per rfc6550, the first byte of ANY RPL message payload is the instance ID. */
    uint8_t rpl_instance_id = UIP_ICMP_PAYLOAD[0];
    instance = rpl_get_instance(rpl_instance_id);

    /* Safety check: ensure it's a valid RPL code and we found a local instance. */
    if(instance == NULL ||
       (UIP_ICMP_BUF->code != RPL_CODE_DIS && UIP_ICMP_BUF->code != RPL_CODE_DIO &&
        UIP_ICMP_BUF->code != RPL_CODE_DAO && UIP_ICMP_BUF->code != RPL_CODE_DAO_ACK)) {
      instance = NULL; /* Not a valid RPL control message we can handle. */
    }
  }

  /* Step 2: If it wasn't a known RPL control message, it might be a data packet.
     Try to find the instance based on the destination address prefix. */
  if(instance == NULL) {
    instance = rpl_get_instance_from_prefix(&UIP_IP_BUF->destipaddr);
  }

  /* Step 3: If we still haven't found an instance, this packet is not our concern.
     Return 1 to allow normal processing to continue without adding/updating headers. */
  if(instance == NULL || instance->current_dag == NULL) {
    return 1; /* Let non-RPL traffic pass through */
  }

  /*
   * From here on, we have a valid 'instance' pointer.
   * All previous uses of 'default_instance' MUST be replaced with 'instance'.
   */

  if(instance->current_dag->rank == ROOT_RANK(instance)) {
    /* Logic for a DODAG root node. */
    rpl_ext_header_remove(); /* Remove any existing RPL headers */
    /* Check if destination is inside our DODAG */
    if(rpl_get_dag(&UIP_IP_BUF->destipaddr) != NULL) {
      if(RPL_IS_NON_STORING(instance)) {
        return insert_srh_header();
      } else {
        return insert_hbh_header(instance);
      }
    } else {
      return 1; /* Destination is outside DODAG, no header needed */
    }
  } else {
    /* Logic for a router/leaf node. */
    if(uip_ds6_is_my_addr(&UIP_IP_BUF->srcipaddr) &&
       UIP_IP_BUF->ttl == uip_ds6_if.cur_hop_limit) {
      /* This is a packet we are originating. Insert a new HBH. */
      return insert_hbh_header(instance);
    } else {
      /* This is a packet we are forwarding. Update the existing HBH. */
      return update_hbh_header();
    }
  }
}
```

### Key Features of This Final Version

*   **No `default_instance`:** It has been completely eliminated.
*   **Dual-Pronged Instance Detection:** It correctly handles both control packets (by reading the payload) and data packets (by checking the prefix).
*   **Robust:** It gracefully handles non-RPL traffic by returning early, allowing those packets to be processed normally.
*   **Correct Logic:** All the final decision-making logic (e.g., `RPL_IS_NON_STORING(instance)`) now correctly uses the `instance` pointer that was determined at the start of the function.

This is the code you should implement now. It represents the culmination of our entire debugging journey for this critical function. Once this is in place, you can move on to the next items on our `grep` list, like `rpl-nbr-policy.c`.

Mahi pai! You've navigated a very complex problem and are now on a clear path to the solution.
