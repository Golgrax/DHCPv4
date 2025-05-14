# DHCPv4

### DHCPv4 CONCEPTS 

*   **Purpose:** Automates the assignment of IPv4 addresses, subnet masks, default gateways, DNS server addresses, and other network parameters to client devices.
*   **Lease Process:** Involves a client requesting an IP (DHCPDISCOVER), a server offering one (DHCPOFFER), the client formally requesting the offered IP (DHCPREQUEST), and the server acknowledging the lease (DHCPACK). This is the DORA process.
*   **Renewal:** Clients attempt to renew their leases before expiration.
*   **Benefits:** Simplifies network administration, reduces IP address conflicts, and efficiently manages IP address allocation.
*   **Cisco Routers:** Can act as both DHCPv4 servers (to provide IPs) and DHCPv4 clients (to receive an IP, e.g., from an ISP).

---

### CONFIGURE DHCPv4 SERVER (Full Topology from PDF Page 12/15)

This section will guide you through setting up R1 as a DHCP server, specifically for `LAN-POOL-1` (serving the 192.168.10.0/24 network where PC1 resides), within the context of the full two-LAN topology shown in the PDF. We will also set up the DNS server.

**Packet Tracer Topology Setup:**

1.  **Place Devices:**
    *   1 Router (e.g., 2911 – name it `R1`)
    *   2 Switches (e.g., 2960 – name them `S1` and `S2`)
    *   2 PCs (name them `PC1` and `PC2`)
    *   1 Server (generic Server device – name it `DNS_Server`)

2.  **Connect Devices (using Copper Straight-Through cables):**
    *   `PC1` (FastEthernet0) -> `S1` (e.g., FastEthernet0/1)
    *   `S1` (e.g., GigabitEthernet0/1) -> `R1` (GigabitEthernet0/0)
    *   `DNS_Server` (FastEthernet0) -> `S2` (e.g., FastEthernet0/1)
    *   `PC2` (FastEthernet0) -> `S2` (e.g., FastEthernet0/2)
    *   `S2` (e.g., GigabitEthernet0/1) -> `R1` (GigabitEthernet0/1)

**Device Configuration Steps:**

**A. Configure `R1` Interfaces:**

```txt
R1> enable
R1# configure terminal

! Configure interface for LAN1 (192.168.10.0/24 network)
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description Link to S1 for LAN10
R1(config-if)# ip address 192.168.10.1 255.255.255.0  ! Gateway for PC1
R1(config-if)# no shutdown
R1(config-if)# exit

! Configure interface for LAN2 (192.168.11.0/24 network)
R1(config)# interface GigabitEthernet0/1
R1(config-if)# description Link to S2 for LAN11
R1(config-if)# ip address 192.168.11.1 255.255.255.0  ! Gateway for PC2 & DNS_Server
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# end
R1# write memory ! Save configuration
```

**B. Configure `DNS_Server`:**

1.  Click `DNS_Server` > Desktop tab > IP Configuration.
2.  Set the following **static** IP configuration:
    *   IP Address: `192.168.11.6` (as per diagram on page 12/15)
    *   Subnet Mask: `255.255.255.0`
    *   Default Gateway: `192.168.11.1` (R1's interface on this LAN)
    *   DNS Server: `192.168.11.6` (itself, or use `127.0.0.1`)
3.  Click `DNS_Server` > Services tab > DNS.
4.  Turn the DNS Service **ON**.
5.  Add a DNS record (optional, but good practice):
    *   Name: `pc1.example.com`
    *   Type: `A Record`
    *   Address: `192.168.10.10` (This assumes PC1 will get this IP from DHCP. Adjust if needed.)
    *   Click `Add`.
    *   Name: `dns.example.com`
    *   Type: `A Record`
    *   Address: `192.168.11.6`
    *   Click `Add`.

**C. Configure `R1` as DHCP Server for `LAN-POOL-1` (for PC1 on 192.168.10.0/24):**
   (This follows the PDF example on page 15)

```txt
R1# configure terminal

! Step 1: Exclude IPv4 addresses for LAN10
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.9
R1(config)# ip dhcp excluded-address 192.168.10.254

! Step 2: Define DHCPv4 pool name for LAN10
R1(config)# ip dhcp pool LAN-POOL-1

! Step 3: Configure the DHCPv4 pool parameters for LAN10
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# dns-server 192.168.11.6  ! Pointing to our configured DNS_Server
R1(dhcp-config)# domain-name example.com
R1(dhcp-config)# exit

R1(config)# end
R1# write memory
```

**D. (Optional) Configure `R1` as DHCP Server for `LAN-POOL-2` (for PC2 on 192.168.11.0/24):**
   The PDF doesn't explicitly show this config, but to make the setup fully functional for both LANs via DHCP from R1:

```txt
R1# configure terminal

! Step 1: Exclude IPv4 addresses for LAN11 (DNS_Server has 192.168.11.6 static)
R1(config)# ip dhcp excluded-address 192.168.11.1 192.168.11.9
R1(config)# ip dhcp excluded-address 192.168.11.6    ! Exclude DNS_Server's static IP
R1(config)# ip dhcp excluded-address 192.168.11.254

! Step 2: Define DHCPv4 pool name for LAN11
R1(config)# ip dhcp pool LAN-POOL-2

! Step 3: Configure the DHCPv4 pool parameters for LAN11
R1(dhcp-config)# network 192.168.11.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.11.1
R1(dhcp-config)# dns-server 192.168.11.6
R1(dhcp-config)# domain-name example.com
R1(dhcp-config)# exit

R1(config)# end
R1# write memory
```

**E. Configure `PC1` and `PC2` as DHCP Clients:**

1.  **For `PC1`:**
    *   Click `PC1` > Desktop tab > IP Configuration.
    *   Select the "DHCP" radio button.
    *   `PC1` should receive an IP address from `LAN-POOL-1` (e.g., `192.168.10.10`), along with gateway `192.168.10.1` and DNS `192.168.11.6`.
2.  **For `PC2`:**
    *   Click `PC2` > Desktop tab > IP Configuration.
    *   Select the "DHCP" radio button.
    *   If you configured `LAN-POOL-2` on R1, `PC2` should receive an IP address (e.g., `192.168.11.10`), gateway `192.168.11.1`, and DNS `192.168.11.6`.
    *   If you *didn't* configure `LAN-POOL-2`, PC2 would get an APIPA address (169.254.x.x) because no DHCP server is configured for its subnet on R1.

**F. Verification:**

*   **On `R1`:**
    ```bash
    R1# show ip dhcp binding
    ! You should see entries for PC1 and PC2 (if LAN-POOL-2 was configured)

    R1# show ip dhcp pool LAN-POOL-1
    ! Shows details about this specific pool

    R1# show ip dhcp pool LAN-POOL-2
    ! Shows details if configured

    R1# show running-config | section dhcp
    ! Shows all DHCP related configuration

    R1# show ip dhcp server statistics
    ! Shows message counts
    ```
*   **On `PC1` (Command Prompt):**
    ```bash
    C:\> ipconfig /all
    ```
    Verify it received correct IP, Gateway, DNS from `LAN-POOL-1`.
    Try pinging its gateway: `ping 192.168.10.1`
    Try pinging the DNS Server: `ping 192.168.11.6`
*   **On `PC2` (Command Prompt):**
    ```bash
    C:\> ipconfig /all
    ```
    Verify it received correct IP, Gateway, DNS (if `LAN-POOL-2` was configured).
    Try pinging its gateway: `ping 192.168.11.1`
    Try pinging the DNS Server: `ping 192.168.11.6`
    Try pinging PC1 (if both are configured and received IPs): `ping <PC1_IP_address>`

This setup now fully reflects the network components shown in the PDF diagram, with R1 capable of serving DHCP to clients on the `192.168.10.0/24` network and optionally to the `192.168.11.0/24` network. The DNS server is also active.

---

### CONFIGURE DHCPv4 CLIENT (Router as DHCP Client - Separate Scenario)

This is a different scenario where a router (like a SOHO router) gets its WAN IP address from an ISP's DHCP server.

**Packet Tracer Topology Setup (New PT file or clear previous):**

1.  **Place Devices:**
    *   1 Router (e.g., 1911 – name it `SOHO_Router`)
    *   1 Router (e.g., 1911 – to act as ISP's DHCP Server, name it `ISP_Router`)
2.  **Connect Devices:**
    *   `SOHO_Router` (GigabitEthernet0/0 or any "WAN" port) -> `ISP_Router` (GigabitEthernet0/0) using a Copper Straight-Through or Crossover cable.

**Configuration Steps:**

**A. Configure `ISP_Router` (to act as ISP's DHCP Server):**

```txt
ISP_Router> enable
ISP_Router# configure terminal

ISP_Router(config)# interface GigabitEthernet0/0
ISP_Router(config-if)# ip address 203.0.113.1 255.255.255.248  ! ISP-side IP, example public range
ISP_Router(config-if)# no shutdown
ISP_Router(config-if)# exit

ISP_Router(config)# ip dhcp pool ISP_POOL
ISP_Router(dhcp-config)# network 203.0.113.0 255.255.255.248 ! Subnet for customers
ISP_Router(dhcp-config)# default-router 203.0.113.1
ISP_Router(dhcp-config)# dns-server 8.8.8.8        ! Public DNS
ISP_Router(dhcp-config)# dns-server 8.8.4.4
ISP_Router(dhcp-config)# domain-name isp.example.com
ISP_Router(dhcp-config)# end

ISP_Router# write memory
```

**B. Configure `SOHO_Router` (to be a DHCP Client on its "WAN" interface):**

```txt
SOHO_Router> enable
SOHO_Router# configure terminal

SOHO_Router(config)# interface GigabitEthernet0/0  ! This is the SOHO_Router's "WAN" interface
SOHO_Router(config-if)# description Link to ISP
SOHO_Router(config-if)# ip address dhcp           ! Tells interface to get IP via DHCP
SOHO_Router(config-if)# no shutdown
SOHO_Router(config-if)# end

SOHO_Router# write memory
```

**C. Verification on `SOHO_Router`:**

*   After a few seconds, you should see a console message on `SOHO_Router` indicating an IP address assignment (like `%DHCP-6-ADDRESS_ASSIGN...`).
*   Check the interface:
    ```bash
    SOHO_Router# show ip interface GigabitEthernet0/0
    ```
    Look for "Internet address is <IP_from_ISP_POOL>/<prefix>" and "Address determined by DHCP".
*   Check routes:
    ```bash
    SOHO_Router# show ip route
    ```
    You should see a default route (S\*) learned via DHCP pointing to the ISP's gateway (`203.0.113.1`).

This provides a comprehensive guide to match the PDF's context. Let me know if any part needs further clarification!
