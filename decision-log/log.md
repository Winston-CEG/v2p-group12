## Entry 1 — Target VRU Group Selection

| Field | Details |
|---|---|
| **Date** | 05 Mar 2026 |
| **Trigger / Problem** | Initial brainstorming: which Vulnerable Road User (VRU) group should we focus on? |
| **Options / Alternatives** | A) General pedestrians; B) Cyclists; C) Visually impaired pedestrians; D) Wheelchair users |
| **Evaluation Criteria** | Novelty, unmet safety need, feasibility of haptic interface, connection to V2X curriculum |
| **Decision and Rationale** | Focus on visually impaired (VI) pedestrians. Highest unmet need — existing V2P prototypes assume visual feedback (phone screens), which is useless for VI users. Haptic feedback is proven effective for VI navigation but has never been combined with C-V2X cooperative awareness. |
| **AI Usage** | ChatGPT was prompted to compare VRU groups by fatality statistics and existing solutions. Output was accurate but lacked Singapore-specific data, which was manually added from LTA reports. |
| **Team Members** | All |

## Entry 2 — Communication Technology and Feedback Modality Direction

| Field | Details |
|---|---|
| Date | 05 Mar 2026 |
| Trigger / Problem | Need to decide on two foundational design choices: (1) which wireless communication technology to use, and (2) how to deliver alerts to a visually impaired user. |
| Options / Alternatives | Comm tech: A) IEEE 802.11p (DSRC); B) LTE-V2X PC5 (3GPP Rel-14); C) NR-V2X PC5 (3GPP Rel-16). Feedback modality: A) Audio-only (bone conduction); B) Haptic-only (vibration belt); C) Audio + haptic combined. |
| Evaluation Criteria | Latency, range, industry momentum, chipset availability; cognitive load on VI users, ambient noise interference, hands-free / ears-free requirement |
| Decision and Rationale | Selected LTE-V2X PC5 (Rel-14) for communication — commercially available chipsets (Qualcomm 9150), < 5 ms direct sidelink latency, 300–450 m range, no cellular network dependency. DSRC rejected due to declining support and FCC spectrum reallocation. For feedback, leaning towards haptic vibration belt — VI pedestrians rely heavily on ambient sound for traffic judgment, so audio risks masking critical environmental cues. A waist-band with directional vibration motors could encode both direction and urgency without blocking hearing. Will finalise motor count and layout in a follow-up entry. |
| AI Usage | ChatGPT was used to summarise the FCC 5.9 GHz spectrum history. One error found: it claimed the FCC fully banned DSRC, when in fact 30 MHz was retained for ITS (corrected from FCC Order 20-305). Claude was asked to compare haptic vs audio for VI users and correctly flagged the ambient sound masking concern. |
| Team Members | All |

## Entry 3 — PC5-Only vs Hybrid PC5 + Cellular

| Field | Details |
|---|---|
| **Date** | 07 Mar 2026 |
| **Trigger / Problem** | Having selected C-V2X PC5 as the communication technology (Entry 2), we need to decide whether the wearable should rely on PC5 direct sidelink only, or also use the Uu (cellular network) path as a secondary channel. |
| **Options / Alternatives** | A) PC5-only for all functions; B) PC5 for safety alerts + Uu via 5G for non-safety features (e.g., route planning, firmware updates); C) Uu-only via MEC server |
| **Evaluation Criteria** | End-to-end latency, dependency on network coverage, hardware cost, power consumption |
| **Decision and Rationale** | PC5-only for all safety-critical alerts, with Uu as an optional fallback for non-safety features. PC5 direct sidelink adds < 5 ms air-interface latency vs 50–200 ms via the cellular path. For a safety application, the system must work even in areas with poor cellular coverage (e.g., underground crossings, parking structures). Adding Uu as a mandatory path would also increase power consumption and require a SIM/eSIM, raising cost and complexity. |
| **AI Usage** | Claude was asked to compare PC5 vs Uu latency. Response was mostly accurate but confused Mode 3 (network-scheduled) with Mode 4 (autonomous resource selection). Corrected from 3GPP TS 36.300. |
| **Team Members** | All |

## Entry 4 — Haptic Motor Count and Arrangement

| Field | Details |
|---|---|
| **Date** | 09 Mar 2026 |
| **Trigger / Problem** | With the haptic wearable direction confirmed (Entry 2), we need to decide how many vibration motors to use and how to arrange them to convey directional information effectively. |
| **Options / Alternatives** | A) 4 motors (cardinal directions only — front, back, left, right); B) 6 motors (60° spacing); C) 8 motors (45° compass-rose pattern); D) 16 motors (22.5° high resolution) |
| **Evaluation Criteria** | Directional resolution, user discriminability on the torso, wearability and comfort, I²C bus complexity, hardware cost |
| **Decision and Rationale** | 8 motors at 45° intervals. Research on vibrotactile perception (Jones & Sarter, 2008) shows that users can reliably distinguish 8 directions on the torso. 4 motors is too coarse — the user cannot tell the difference between front-left and left, which matters when a vehicle is approaching from an angle. 16 motors exceeds the tactile spatial acuity of the waist area and doubles the wiring complexity for negligible perceptual gain. 8 provides the best trade-off between directional precision and practical wearability. |
| **AI Usage** | Claude initially suggested 6 motors. When shown the Jones & Sarter (2008) study on torso-based vibrotactile displays, it agreed that 8 is the established standard. This highlighted the importance of verifying AI suggestions against domain-specific literature. |
| **Team Members** | All |
