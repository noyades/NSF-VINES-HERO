# Torgerson 2015 D-MIMO Testbed Development Plan

## Purpose

Develop a four-node distributed MIMO testbed in Torgerson 2015 using four AMD/Xilinx ZCU208 RFSoC evaluation boards and an AMD Threadripper workstation as the distributed unit (DU). The design will use SFP+ for high-rate data transport, White Rabbit (WR) for timing, and frequency-translation front ends to reach the agreed FR3 band. Swedish and Virginia Tech Institute for Advanced Computing (IAC) members will be able to access the testbed remotely under lab and institutional security controls.

This is a development plan, not a finalized RF or network design. Resolve the decisions marked **Gate** before purchasing RF hardware or committing to a deployment topology.

## Target Architecture

```text
                      Institutional remote users
                    Sweden / VT IAC / local operators
                                |
                Approved VPN or zero-trust access + MFA
                                |
                   Lab bastion / management host
                                |
       +------------------------+-------------------------+
       |                                                  |
  Management network                              Test-data network
       |                                                  |
  DU control / logging     DU (Threadripper + NIC) -- 10GbE SFP+ switch
                                                         |  |  |  |
                                                 ZCU208 x4 (one per site/node)
                                                  |                 |
                                      RF frequency-translation front ends
                                                  |
                                         Antennas / conducted test setup

       Independent timing plane: WR grandmaster/switch -> WR-capable endpoints
       (ZCU208 endpoint implementation and DU participation remain to be verified)
```

Use a switched star for the data plane unless measured traffic or latency requirements justify direct DU-to-radio links. Keep the WR timing plane logically and, where practical, physically distinct from management and payload traffic. Do not assume a conventional Ethernet switch or an SFP+ port is WR-capable.

## Decisions and Verification Gates

| Gate | Decision or evidence required | Why it matters |
|---|---|---|
| G1: Requirements | Document carrier/band edges, bandwidth, channel count, sample rate, duplex mode, transmit power, antenna count, mobility, and target synchronization error. Clarify what “FR3” means for this project. | Determines RF conversion, bandwidth, clock quality, and transport capacity. |
| G2: Board capability | For the exact ZCU208 revision, verify RFSoC direct-conversion frequency/bandwidth, usable RFDC channels, clock inputs, high-speed transceiver mapping, and available SFP+ interface(s) from its AMD documentation and board files. | Avoids designing around assumed RF or Ethernet capabilities. |
| G3: WR endpoints | Select a WR grandmaster/switch and prove a supported WR endpoint path for every node. Check whether endpoints will be FPGA/firmware implementations, compatible interface hardware, or another qualified design. Decide whether the DU needs WR time or only distributes application data. | Standard SFP+ connectivity does not establish WR synchronization by itself. |
| G4: RF front ends | Confirm whether the system needs transmit-only upconversion or bidirectional up/downconversion, required LO distribution/coherence, filtering, gain, linearity, and antenna interfaces. Obtain vendor specifications and lab compatibility evidence. | A BUC is generally a transmit upconverter; it may not provide the receive path needed for D-MIMO. |
| G5: Site and policy | Complete Torgerson 2015 room survey, network/IT approval, facilities review, RF exposure assessment, and institutional review for international remote access and data handling. | Determines safe RF operation, connectivity, and allowed remote workflows. |

## Work Plan

### 1. Requirements and Site Survey

- Capture the radio, PHY/MAC, synchronization, latency, throughput, and experiment-control requirements in a short system requirements document.
- Survey Torgerson 2015: rack/bench locations, cable routes and lengths, power circuits, ventilation, grounding, antenna/conducted-test locations, physical access, and available institutional network services.
- Establish whether the four ZCU208s are co-located or spatially distributed. Record the maximum fiber and RF cable lengths and the intended antenna geometry.
- Define whether experiments are conducted, shielded, or radiated. Obtain facilities and campus approvals before transmitting over the air.
- Inventory exact ZCU208 board revisions, RF daughtercards/accessories, optics, antennas, Threadripper configuration, and available rack equipment.

**Exit criteria:** approved requirements baseline; site layout; initial block diagram; list of unresolved gates and owners.

### 2. DU and Data-Plane Bring-Up

- Configure the Threadripper workstation as the DU host. Confirm CPU, memory, PCIe lane availability, BIOS/IOMMU settings, cooling, storage, and OS/kernel compatibility with the selected radio software.
- Select a supported multiport 10GbE SFP+ NIC or a non-blocking SFP+ switch with enough ports for four radios and DU uplinks. Confirm NIC and switch driver/firmware support, optics type, fiber type, link budget, and interoperability with the ZCU208 interfaces.
- Specify whether each radio uses one or more 10GbE links; size aggregate bandwidth with framing, control traffic, and headroom included.
- Create separate management and experiment-data networks. Use dedicated VLANs only where campus policy and switch configuration support the required isolation; do not rely on VLANs as the sole security boundary.
- Bring up one ZCU208 first. Verify link negotiation, MTU, packet loss, throughput, CPU load, and sustained performance, then repeat with all four nodes under representative load.
- Version-control DU and FPGA images, build instructions, configuration files, and a known-good recovery image. Store calibration and experiment results with timestamps and configuration identifiers.

**Exit criteria:** repeatable single-node and four-node data-plane tests meeting the agreed throughput, latency, loss, and stability targets.

### 3. White Rabbit Timing

- Select an appropriate WR grandmaster and WR switch topology based on required accuracy, holdover, redundancy, and node placement. Verify qualified hardware, optical compatibility, and firmware availability.
- Select and implement the endpoint path for all four ZCU208s. Validate that endpoint timestamping, clock recovery, phase alignment, and required RFDC reference/sample-clock distribution are supported by the actual design.
- Decide whether the DU requires synchronized time, frequency, or phase. If it does, specify a supported WR NIC/endpoint or an explicitly bounded interface to the timing system.
- Characterize the complete timing chain, including RF front-end LO/reference distribution. WR time synchronization alone does not guarantee phase coherence at the antennas.
- Measure offset, drift, jitter, lock/relock behavior, and inter-node phase stability under cold start, warm-up, fiber changes, and representative data traffic. Record instrumentation and uncertainty.
- Define alarm thresholds and safe experiment behavior when a node loses synchronization.

**Exit criteria:** all four radio nodes meet the project-defined synchronization and phase-coherence requirements in repeated tests, with logs and recovery behavior documented.

### 4. FR3 RF Front Ends

- Confirm the project frequency range, occupied bandwidth, duplexing arrangement, transmit power at the antenna, receiver sensitivity/noise figure, and number of simultaneous RF chains.
- Evaluate suitable up/down-conversion front ends rather than assuming transmit-only BUCs are sufficient. Confirm RF connectors, gain, compression, spectral emissions, image/spur rejection, filtering, and thermal requirements.
- Establish a coherent LO/reference plan across all four nodes. Test whether a common reference can be distributed safely and whether converter phase repeatability is adequate after power cycling and retuning.
- Add appropriate filtering, attenuation, couplers, loads, and measurement access for conducted bring-up. Characterize each chain before connecting antennas.
- Document permitted frequencies, EIRP/power limits, RF exposure controls, interlocks, and required campus/regulatory approvals. Do not radiate until approvals and measured output limits are in place.

**Exit criteria:** calibrated and repeatable transmit/receive response over the agreed band; acceptable inter-node gain/phase match; documented safe operating limits and approvals.

### 5. Remote Access for Swedish and VT IAC Members

- Confirm sponsorship, institutional identity providers, data classification, export-control or other applicable restrictions, and campus IT approval for cross-border access before enabling accounts.
- Provide access through the approved institutional VPN or zero-trust service with MFA and named individual accounts. Do not expose SSH, board-management interfaces, FPGA control ports, or testbed services directly to the public Internet.
- Use a hardened bastion or managed access host as the sole remote entry point. Restrict access by identity, group, device posture where available, and source policy; require an individual account and auditable session for each collaborator.
- Separate roles: **observer** (read-only status and logs), **experimenter** (submit approved jobs and operate scheduled experiments), and **administrator** (system/network/firmware changes). Grant the minimum role needed and review membership on a fixed schedule.
- Keep management interfaces on a dedicated restricted network. Allow remote users to reach only the bastion and approved experiment-control services; do not provide general routing to the DU, radio management ports, or campus LAN.
- Use key-based authentication managed through approved institutional processes, short-lived credentials where supported, least privilege, patching, endpoint protection, audit logs, and documented revocation. Never share accounts or credentials.
- Define experiment scheduling, local supervision requirements, remote emergency stop, RF transmit interlocks, maintenance windows, and a contact/escalation path across time zones.
- Test remote workflows first with RF transmit disabled: login/MFA, role boundaries, job submission, log retrieval, session auditing, account revocation, and recovery after a network interruption.

**Exit criteria:** both organizations' approved users can perform their authorized tasks; unauthorized paths are blocked; sessions and changes are auditable; emergency stop and access revocation are tested.

### 6. Integration and D-MIMO Validation

- Integrate in this order: one ZCU208 data link; one WR endpoint; one RF chain; then add nodes incrementally to four.
- Establish per-node calibration for timing, phase, gain, and frequency offset. Store calibration metadata against board serial number, RF front-end serial number, firmware version, temperature, and configuration.
- Run baseline tests for each node independently, then pairwise synchronization/phase tests, then four-node D-MIMO experiments.
- Validate operation after restart, fiber reseat, clock relock, RF retune, and remote session interruption. Define what state must be restored automatically and what requires local confirmation.
- Produce an operator runbook, topology and cable map, bill of materials, recovery procedures, test reports, and experiment configuration archive.

## Acceptance Criteria

Before declaring the testbed ready, agree numeric targets for the following and record measured results:

- Sustained aggregate and per-node data throughput, end-to-end latency, packet loss, and CPU utilization.
- WR lock status, inter-node time offset, frequency/phase stability, and reacquisition time.
- RF coverage band, bandwidth, output power, receive performance, spurious emissions, and repeatability.
- Four-node D-MIMO performance against a documented single-node or non-coherent baseline.
- Remote access availability, role separation, audit coverage, revocation time, and emergency-stop response.

## Initial Risks and Mitigations

| Risk | Mitigation |
|---|---|
| ZCU208 interfaces or RFDC limits do not match assumed SFP+ or FR3 requirements. | Verify board revision documentation and perform a one-node proof of concept before buying four sets of front ends or optics. |
| No supported WR endpoint exists for the selected ZCU208 configuration. | Make endpoint feasibility an early design gate; prototype one endpoint before full system integration. Keep data transport and timing requirements independently testable. |
| BUC solution is transmit-only or lacks coherent LO support. | Specify transmit and receive paths separately; require reference/LO and phase-stability data from vendors and test a representative chain. |
| Four-node data traffic exceeds the DU NIC, switch, or PCIe capacity. | Estimate link budgets and PCIe topology; measure one node first, then stress all four with headroom. |
| Remote access conflicts with campus or cross-border policy. | Obtain IT/security and research-policy approval before accounts or routes are enabled; keep local-only operation available. |
| RF emissions, exposure, or interference are not approved. | Use conducted tests and RF loads first; require written approvals, power limits, and interlocks before radiated tests. |

## Immediate Next Actions

1. Assign owners for requirements, site/IT, DU networking, WR, and RF front ends.
2. Fill in G1 and G5: target FR3 range/use case, radio requirements, room layout, and institutional access constraints.
3. Verify the exact ZCU208 board revisions and obtain the applicable AMD product guide and board schematics; map RFDC, reference-clock, and high-speed serial resources.
4. Select one candidate WR master/switch and demonstrate one ZCU208 endpoint before procuring the full timing system.
5. Select one candidate FR3 transceiver chain and perform conducted bench characterization before scaling to four nodes.
6. Obtain campus IT approval for the management/data network and cross-institution remote-access model.

## Preliminary Cost Estimate

All amounts are rough-order-of-magnitude (ROM) planning estimates in USD, not vendor quotes. The DU and four ZCU208 boards are already owned and are excluded. Taxes, shipping, recurring institutional network charges, and major room/electrical construction are excluded.

**Planning basis:** four nodes, one bidirectional coherent RF chain per ZCU208, switched 10GbE SFP+ data plane, WR synchronization, fiber runs within a typical lab/building, campus VPN/MFA available, and existing access to basic RF measurement equipment. The FR3 band and RF power requirements are not yet specified.

| Missing item | Planning quantity / basis | ROM cost |
|---|---|---:|
| 10GbE SFP+ data-plane hardware | Managed switch, DU NIC if needed, compatible optics, fiber patch leads, and spares | $1,000-$4,000 |
| White Rabbit timing hardware | Grandmaster/switch and four endpoint paths; endpoint line assumes compatible implementations can be used | $10,000-$40,000 |
| FR3 Tx/Rx converter chains | Four bidirectional chains, one per ZCU208; band, bandwidth, and power can move this substantially | $12,000-$48,000 |
| RF support hardware | Coherent LO/reference distribution, filtering, gain/attenuation, couplers, loads, and RF cabling | $5,000-$25,000 |
| Antennas and RF feed | Four-node pilot setup; antenna count, gain, and cabling depend on geometry | $1,000-$6,000 |
| RF test equipment | $0 if suitable shared instruments are available; otherwise basic spectrum/vector analysis and calibration accessories | $0-$25,000 |
| Remote access and management | $0-$4,000 assuming institutional VPN/MFA; covers a dedicated bastion or management-network hardware if required | $0-$4,000 |
| Rack, UPS, PDU, and installation materials | Small lab installation, patching, labeling, and power protection | $2,000-$8,000 |
| Initial integration effort | Bring-up, measurement, documentation, and limited vendor/contractor support | $5,000-$25,000 |
| **Subtotal** | Excludes DU and ZCU208s | **$36,000-$185,000** |
| **Planning amount with 25% contingency** | Contingency covers current design uncertainty, not major construction | **$45,000-$230,000** |

For a proposal-level placeholder, use **$100,000-$150,000** if shared RF instrumentation and a supported WR endpoint path are available. Use the broader **$45,000-$230,000** range for internal planning until the RF band, chain count, and WR endpoint are confirmed.

### Cost Drivers and Exclusions

- The RF estimate assumes one Tx/Rx chain per node. Additional bidirectional RF chains scale roughly with chain count; use **$3,000-$12,000+ per additional chain** as an initial allowance, subject to vendor quotes and the required band/power.
- “BUC” commonly means a transmit upconverter. If receive operation is required, budget and qualify the receive downconverter separately; confirm that the selected converters support the actual FR3 frequencies, bandwidth, reference input, and phase repeatability.
- The WR allowance assumes an endpoint approach exists for the selected ZCU208 design. If custom FPGA/firmware endpoint development is necessary, add a separate engineering reserve of approximately **$20,000-$75,000+** or revisit the synchronization architecture before procurement.
- The RF test-equipment line assumes shared access is checked first. A calibrated high-performance vector signal analyzer, wideband signal generator, or anechoic/shielded facility can exceed this allowance individually.
- Remote access assumes campus IT supplies the VPN/identity/MFA service. A new managed security appliance, external support contract, or cross-institution service could add purchase and recurring costs.
- Obtain at least two quotes for WR and RF equipment after G1-G4 are resolved. Confirm lead times, calibration, support, import/shipping, and whether prices include required cables, licenses, and endpoint firmware.

## Reference Links

- [AMD ZCU208 Evaluation Board](https://www.amd.com/en/products/adaptive-socs-and-fpgas/evaluation-boards/zcu208.html) - use the documentation for the exact board revision as the source of record for interfaces and RF capabilities.
- [White Rabbit Collaboration: Technology](https://www.white-rabbit.tech/wr-technology/) - WR technology background; validate selected products and endpoint implementations against project requirements.
- [White Rabbit Collaboration: Qualified Products](https://www.white-rabbit.tech/trademark/) - product qualification information.
