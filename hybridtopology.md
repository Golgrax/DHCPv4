## 1. Access Layer (Star)

* **PC1 → Switch5**
* **PC0 → Switch4**
* **PC3 → Switch7**
* **PC2 → Switch6**
* **PC4 → Switch1**
* **PC8 → Switch3**
* **PC5 → Switch2**

Bawat PC may **dedicated** straight‑through cable papunta sa sariling access switch—‘yan ang star pattern sa bawat endpoint.

---

## 2. Distribution/Core Layer (Partial Mesh/Ring)

### A. Top “bus” segment

* **Switch5 ↔ Switch4 ↔ Switch0 ↔ Switch6 ↔ Switch7**

  * Naka‑line ito parang backbone: lahat ng top‑of‑rack switches (5, 4, 0, 6, 7) magkakasunod.

### B. Middle diamond (mesh)

* **Switch4**, **Switch1**, **Switch0**, **Switch3** at **Switch2**

  * Switch4 → Switch1 → Switch0 → Switch3 → Switch2 → balik sa Switch0 → balik sa Switch4
  * Binubuo ng dashed lines na nagpapakita ng trunks sa pagitan nila.

### C. STP Block

* Nakita mo yung **orange link**—‘yun yung isang trunk port na naka‑block ng Spanning Tree Protocol (STP).

  * Panatag si STP na walang broadcast loop, pero kapag bumagsak ang ibang path, agad siyang mag‑unblock para may alternate route.

---

## 3. Bakit Hybrid?

1. **Star**: PC → access switch
2. **Ring/Mesh**: Access switches interconnected for redundancy
3. **Tree**: Access layer → Distribution/Core layer hierarchy

Pinaghalo‑halo ‘to para:

* **Scalable**—madali magdagdag ng PC o switch.
* **Redundant**—kung may failure sa isang cable o switch, reroute traffic sa ibang daan.
* **Loop‑free** dahil sa STP, pero handa ‘yung backup path.

---

## 4. Real‑World Benefit

Sa production networks, gusto mo high availability. ‘Di ka mag‑asa sa isang giant switch lang—kung down ‘yun, lahat ng PC mawawala sa network. Kaya:

1. **Group users** each under an access switch (star).
2. **Interconnect** switches in mesh/ring (redundancy).
3. **Patakbuhin** STP para safe ka sa loops ngunit may automatic failover.
