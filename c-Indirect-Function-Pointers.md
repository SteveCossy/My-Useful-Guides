**indirect calls via function pointers.**

Let's break down how this works.

### The Mechanism: A Generic "Routing Driver" Interface

The core IPv6 stack in Contiki-NG is designed to be **routing-protocol-agnostic**. It doesn't know or care whether you are using RPL Classic, RPL Lite, or some future protocol. To achieve this, it defines a generic "contract" or "interface" for what a routing protocol must be able to do.

This contract is the `struct routing_driver`. It's a structure made almost entirely of **function pointers**.

Conceptually, the generic `routing.h` file defines this structure:
```c
// In a file like os/net/routing/routing.h (conceptual)
struct routing_driver {
  char *name;
  void (*init)(void);
  // ... many other function pointers ...
  void (*ext_header_remove)(void);
  void (*ext_header_update)(void); // <-- This is the function pointer
  // ... more function pointers ...
};
```

### Step 1: Registration (The "Plug-in" Step)

Your `rpl.c` file contains the `rpl_classic_driver` struct. This is the **concrete implementation** of that generic contract. You are providing the actual functions that fulfill the roles defined in the interface.

During the system's initialization, Contiki-NG sets a global pointer to point to your driver. There is a global variable, often called `ROUTING_DRIVER`, that is set like this:
`ROUTING_DRIVER = &rpl_classic_driver;`

Now, the entire operating system knows that "RPL Classic" is the active routing protocol.

### Step 2: The Indirect Call (The Hidden Call Site)

The core IPv6 stack, likely in a file like `os/net/ipv6/uip6.c`, is responsible for parsing incoming packets. When it parses a packet and finds a Hop-by-Hop extension header that needs to be processed, it does **not** call `rpl_ext_header_update()` directly. It has no knowledge of that function's name.

Instead, it does this:
```c
// Inside a function in uip6.c, for example

// 'ROUTING_DRIVER' is the global pointer to the active routing driver.
if(ROUTING_DRIVER.ext_header_update != NULL) {
  ROUTING_DRIVER.ext_header_update();
}
```
This is the **indirect call**. The code calls the function pointed to by the `ext_header_update` field of whatever the active `ROUTING_DRIVER` is. Since we've registered `rpl_classic_driver`, this call is transparently redirected to your `rpl_ext_header_update` function.

### How to Find the Call Site

Your `grep` for the function name failed because the name doesn't exist in the calling code. To find where it's being called from, you need to search for the **field name from the struct**.

**Try this `grep` command:**

```bash
grep -rn "ext_header_update" /home/stevecos/contiki-ng/os/
```
This will search for any use of the member `ext_header_update`. You will likely find the call inside `os/net/ipv6/uip6.c` or `os/net/tcpip.c`, which are the main engines of the IPv6 stack.

**The Packet's Journey (Simplified):**
1.  A packet arrives at the radio driver.
2.  It's passed up to `tcpip_input()`.
3.  `tcpip_input()` identifies it as an IPv6 packet and passes it to `uip6_input()`.
4.  `uip6_input()` parses the main IPv6 header. It sees the "Next Header" field points to a Hop-by-Hop Options header.
5.  It enters a loop or calls a function to process extension headers.
6.  Inside that logic, it finds the line `ROUTING_DRIVER.ext_header_update();`, which executes your function.
7.  After your function returns, the IPv6 stack continues processing the rest of the packet, eventually passing it up to the ICMPv6 layer, which is when `dio_input` is finally called.

This mechanism is what makes the system modular. You could write a completely new routing protocol, fill in the `struct routing_driver` with your new functions, and plug it in without ever touching a line of the core `uip6.c` code. It's a very powerful and elegant design.
