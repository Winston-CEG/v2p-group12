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
