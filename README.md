# SafeCross V2P

**A C-V2X Haptic Alert System for Visually Impaired Pedestrians**

CEG3006 Group Project | AY2025/26 Tri 2 | Group 12

---

## Solution Overview

SafeCross V2P is a Vehicle-to-Pedestrian safety system that leverages C-V2X PC5 sidelink direct communication to deliver real-time haptic warnings to visually impaired (VI) pedestrians via a lightweight wearable waist-band. Unlike conventional assistive navigation devices that rely solely on the pedestrian's own on-board sensors (cameras, ultrasonic, LiDAR), SafeCross taps into the cooperative awareness already broadcast by connected vehicles, extracting approach trajectory, speed, and turn-signal intent directly from Cooperative Awareness Messages (CAMs) and Decentralised Environmental Notification Messages (DENMs) defined in ETSI ITS standards.

The system consists of three subsystems:

1. **VRU Device (VD):** A smartphone-sized C-V2X modem carried in the wearable belt pouch, receiving PC5 sidelink broadcasts from nearby On-Board Units (OBUs) and Roadside Units (RSUs).
2. **Collision Risk Engine (CRE):** An edge-compute module that fuses the pedestrian's GNSS position with incoming vehicle kinematic data to compute Time-to-Collision (TTC) and Post-Encroachment Time (PET).
3. **Haptic Feedback Interface (HFI):** A waist-band with eight directional ERM vibration motors arranged in a compass-rose pattern, encoding the direction and urgency of an approaching vehicle through vibration intensity and pulse frequency.

The core value proposition is that SafeCross does not require the pedestrian to carry expensive sensors or rely on camera-based AI. The safety intelligence comes from the vehicles themselves broadcasting their state, making it robust in poor lighting, rain, and occluded environments.

## Literature Review

Pedestrians with visual impairments face disproportionate road safety risks. Studies show that driver yielding rates are significantly lower when a pedestrian uses a white cane, and complex intersections make gap-judgment especially difficult for VI individuals (Wall Emerson et al., 2011; Geruschat & Hassan, 2005). Existing assistive technologies fall into three categories:

**On-board perception systems.** Devices such as the VIS4ION backpack (Rizzo et al., 2018) and .lumen smart glasses (Silvestru et al., 2024) use depth cameras and AI object detection to identify obstacles and provide haptic or audio feedback. While effective for obstacle avoidance, they cannot perceive vehicle intent (e.g., a turning vehicle still 50 m away) and are limited by field-of-view and on-device compute power.

**Infrastructure-based V2P pilots.** The Maryland DOT I2V pedestrian pilot (2019–2021) and the NYC PED-SIG app equipped intersections with RSUs or smartphone interfaces to relay signal-phase-and-timing (SPaT) data. These are geographically constrained to instrumented intersections and do not provide vehicle-specific approach warnings.

**Smartphone-centric V2P prototypes.** Several academic prototypes broadcast Basic Safety Messages from vehicles and listen on the pedestrian's smartphone (Anaya et al., 2019). However, they rely on the Uu (cellular network) path, introducing 50–200 ms of additional latency and dependency on network coverage.

**SafeCross differs from the above in three key respects:**

| Aspect | Existing Solutions | SafeCross |
|---|---|---|
| Sensing modality | On-board cameras / LiDAR on pedestrian | Cooperative V2X broadcast from vehicles |
| Communication path | Uu (cellular) or Bluetooth | PC5 sidelink (direct, sub-20 ms) |
| User interface for VI | Audio beeps or smartphone screen | Directional haptic waist-band (hands-free, ears-free) |

## System Architecture

### High-Level Architecture

```
  ┌──────────────┐       PC5 Sidelink (5.9 GHz)       ┌──────────────────────┐
  │  Connected   │ ──────────────────────────────────► │   VRU Device (VD)    │
  │  Vehicle     │   CAM / DENM broadcast              │                      │
  │  (OBU)       │   (10 Hz, ≤ 300 m range)            │  ┌────────────────┐  │
  │              │                                      │  │ C-V2X Modem    │  │
  │  - GNSS      │                                      │  │ (Qualcomm      │  │
  │  - CAN bus   │       PC5 Sidelink                   │  │  9150 C-V2X)   │  │
  │  - C-V2X OBU │ ◄──────────────────────────────────  │  └───────┬────────┘  │
  └──────────────┘   VRU Awareness Message (VAM)        │          │           │
                                                        │  ┌───────▼────────┐  │
  ┌──────────────┐       PC5 Sidelink                   │  │ Collision Risk │  │
  │  Roadside    │ ──────────────────────────────────►  │  │ Engine (CRE)   │  │
  │  Unit (RSU)  │   SPaT / MAP messages                │  │                │  │
  │              │                                      │  │ - TTC calc     │  │
  │  - Traffic   │                                      │  │ - PET calc     │  │
  │    signal    │       Uu Interface (5G)               │  │ - Threat level │  │
  │    controller│  ┌────────────────────────────────►  │  └───────┬────────┘  │
  └──────────────┘  │  (backup / MEC offload)           │          │           │
                    │                                   │  ┌───────▼────────┐  │
  ┌──────────────┐  │                                   │  │ Haptic Feedback│  │
  │  MEC Server  │ ─┘                                   │  │ Interface (HFI)│  │
  │  (optional)  │                                      │  │                │  │
  │  - Extended  │                                      │  │ 8× ERM motors  │  │
  │    collision │                                      │  │ in waist-band  │  │
  │    prediction│                                      │  └────────────────┘  │
  └──────────────┘                                      └──────────────────────┘
```

### Protocol Stack

```
┌─────────────────────────────────────────────────────┐
│                  APPLICATION LAYER                    │
│  ┌───────────┐  ┌───────────┐  ┌─────────────────┐  │
│  │    CAM     │  │   DENM    │  │      VAM        │  │
│  │ (EN 302   │  │ (EN 302   │  │  (TS 103 300)   │  │
│  │  637-2)   │  │  637-3)   │  │                  │  │
│  └─────┬─────┘  └─────┬─────┘  └───────┬──────────┘ │
│        │              │                │              │
│  ┌─────▼──────────────▼────────────────▼──────────┐  │
│  │           FACILITIES LAYER                      │  │
│  │   GeoNetworking (EN 302 636-4-1)               │  │
│  │   Basic Transport Protocol (BTP)               │  │
│  └──────────────────────┬─────────────────────────┘  │
│                         │                             │
│  ┌──────────────────────▼─────────────────────────┐  │
│  │           ACCESS LAYER                          │  │
│  │   C-V2X PC5 Sidelink (3GPP Rel-14/16)         │  │
│  │   5.9 GHz ITS Band | 10/20 MHz channel BW     │  │
│  └────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Haptic Motor Layout

```
           Front (12 o'clock)
                 M1
            M8        M2

       M7    [WAIST]     M3

            M6        M4
                 M5
           Back (6 o'clock)
```

| Threat Level | TTC / PET Threshold | Pulse Rate | Vibration Intensity |
|---|---|---|---|
| LOW | TTC > 5 s | 1 Hz (1 pulse/s) | 30% duty cycle |
| MEDIUM | 3 s < TTC ≤ 5 s | 3 Hz (3 pulses/s) | 60% duty cycle |
| HIGH | TTC ≤ 3 s | 5 Hz (continuous burst) | 100% duty cycle |
| EMERGENCY (DENM) | N/A | All 8 motors simultaneous | 1 s burst |

## Functions and Messages

### Message Definitions

**Messages received by VRU Device:**

| Message | Standard | Content | Frequency | Size |
|---|---|---|---|---|
| CAM | ETSI EN 302 637-2 | Vehicle position, speed, heading, acceleration, dimensions, turn signal state | 1–10 Hz (adaptive) | 50–300 bytes |
| DENM | ETSI EN 302 637-3 | Hazard alerts: emergency braking, wrong-way driver, accident | Event-triggered | 200–800 bytes |
| SPaT | SAE J2735 | Traffic signal current phase, time-to-change, pedestrian phase | 10 Hz | 100–500 bytes |
| MAP | SAE J2735 | Intersection geometry, lane markings, crosswalk locations | Static (on change) | 500–2000 bytes |

**Messages transmitted by VRU Device:**

| Message | Standard | Content | Frequency | Size |
|---|---|---|---|---|
| VAM | ETSI TS 103 300 | VRU position, speed, heading, VRU type (pedestrian/wheelchair), mobility aid type | 1–5 Hz (adaptive) | 50–200 bytes |

### Core Processing Flow

```
  System Power On
       │
       ▼
  Initialise C-V2X modem, GNSS, IMU, haptic motors
       │
       ▼
  Acquire GNSS fix (< 1.5 m accuracy) ──► Begin broadcasting VAM at 1 Hz
       │
       ▼
  ┌─── MAIN LOOP (100 ms cycle) ◄──────────────────────────┐
  │                                                          │
  │  Receive PC5 messages (CAM, DENM, SPaT, MAP)            │
  │       │                                                  │
  │       ▼                                                  │
  │  Any vehicles in range?                                  │
  │       │ No ──► Clear all haptic motors ─────────────────►│
  │       │ Yes                                              │
  │       ▼                                                  │
  │  For each vehicle:                                       │
  │    1. Compute relative bearing and distance              │
  │    2. Compute TTC = effective_distance / closing_speed   │
  │    3. Compute PET at predicted conflict point            │
  │       │                                                  │
  │       ▼                                                  │
  │  Classify threat level:                                  │
  │    TTC ≤ 3 s OR PET ≤ 1.5 s  ──► HIGH                   │
  │    TTC ≤ 5 s OR PET ≤ 3.0 s  ──► MEDIUM                 │
  │    TTC ≤ 8 s OR PET ≤ 5.0 s  ──► LOW                    │
  │    else                       ──► NONE                   │
  │       │                                                  │
  │       ▼                                                  │
  │  Map to haptic output:                                   │
  │    - Bearing ──► motor index (8 sectors of 45°)          │
  │    - Threat level ──► pulse rate + duty cycle            │
  │    - DENM emergency ──► all-motor 1 s burst              │
  │    - SPaT ped-phase ending < 5 s ──► front double-tap   │
  │       │                                                  │
  │       ▼                                                  │
  │  Actuate haptic motors via PWM (I²C, 400 kHz)           │
  │       │                                                  │
  └───────┘                                                  │
          └──────────────────────────────────────────────────┘
```

### TTC and PET Computation (Pseudo-code)

```python
def compute_ttc(veh_pos, veh_speed, veh_heading, ped_pos, ped_speed, ped_heading):
    """
    Compute Time-to-Collision between a vehicle and pedestrian.
    All positions in local ENU (East-North-Up) frame.
    """
    dx = ped_pos.east - veh_pos.east
    dy = ped_pos.north - veh_pos.north
    distance = sqrt(dx**2 + dy**2)

    # Vehicle velocity components
    veh_vx = veh_speed * sin(veh_heading)
    veh_vy = veh_speed * cos(veh_heading)

    # Pedestrian velocity components
    ped_vx = ped_speed * sin(ped_heading)
    ped_vy = ped_speed * cos(ped_heading)

    # Closing speed (projection of relative velocity onto connecting line)
    bearing = atan2(dx, dy)
    rel_vx = veh_vx - ped_vx
    rel_vy = veh_vy - ped_vy
    closing_speed = rel_vx * sin(bearing) + rel_vy * cos(bearing)

    if closing_speed <= 0:
        return INFINITY  # Not approaching

    safety_buffer = veh_width / 2 + 0.5  # 0.5 m pedestrian radius
    effective_distance = max(distance - safety_buffer, 0)

    return effective_distance / closing_speed


def compute_pet(veh_pos, veh_speed, ped_pos, ped_speed, conflict_point):
    """
    Compute Post-Encroachment Time at a predicted conflict point.
    """
    t_veh = distance(veh_pos, conflict_point) / veh_speed if veh_speed > 0.5 else INFINITY
    t_ped = distance(ped_pos, conflict_point) / ped_speed if ped_speed > 0.3 else INFINITY

    return abs(t_veh - t_ped)


def classify_threat(ttc, pet):
    if ttc <= 3.0 or pet <= 1.5:
        return THREAT_HIGH
    elif ttc <= 5.0 or pet <= 3.0:
        return THREAT_MEDIUM
    elif ttc <= 8.0 or pet <= 5.0:
        return THREAT_LOW
    else:
        return THREAT_NONE


def bearing_to_motor(bearing_deg):
    """Map 0–360° bearing (0 = front) to motor index 1–8."""
    sector = int((bearing_deg + 22.5) % 360 / 45)
    return sector + 1  # M1 = front, M2 = front-right, ... M8 = front-left
```

## Hardware Components and Parameters

### Core Components

| Component | Model | Key Parameters |
|---|---|---|
| C-V2X Modem | Qualcomm 9150 C-V2X | PC5 direct: 5.855–5.925 GHz; 10/20 MHz BW; Tx: 23 dBm; Range: ≤ 450 m (LOS); Latency: < 5 ms |
| GNSS Module | u-blox ZED-F9P | Multi-band L1/L2; < 1.5 m accuracy (urban); 10 Hz update rate |
| IMU | Bosch BNO055 | 9-axis (accel + gyro + mag); 100 Hz; dead-reckoning for GNSS dropouts |
| Application Processor | Qualcomm QCS6490 | ARM Cortex-A78 @ 2.7 GHz; ~2.5 W active |
| Battery | Li-Po 3000 mAh, 3.7 V | Runtime: ~6.2 hrs (adaptive mode); USB-C charging |
| Enclosure | IP65-rated belt pouch | 90 × 60 × 25 mm; ~120 g (excl. battery) |

### Haptic Interface

| Component | Model | Key Parameters |
|---|---|---|
| Vibration Motors | 8× Precision Microdrives 310-117 (ERM) | 10 mm dia; 3.0 V; 1.2 G amplitude; < 50 ms response; 75 mA each |
| Motor Driver | 8× TI DRV2605L | I²C (400 kHz); PWM duty cycle 0–100% |
| I²C Multiplexer | TCA9548A | 8-channel; individual motor addressing |
| Waist-band | Elastic fabric, adjustable | 65–120 cm circumference; motor pockets at 45° intervals; washable |

### End-to-End Latency Budget

| Stage | Latency | Notes |
|---|---|---|
| Vehicle OBU → PC5 air interface | < 5 ms | Direct sidelink, no base station |
| PC5 reception + decoding | 2–3 ms | Hardware decoding on Qualcomm 9150 |
| CRE processing (TTC/PET) | 1–2 ms | Fixed-point arithmetic |
| Motor driver actuation | < 50 ms | ERM spin-up time |
| **Total end-to-end** | **< 60 ms** | Well within human reaction time (~250 ms) |

### Power Budget

| Component | Active Power | Duty Cycle | Average Power |
|---|---|---|---|
| C-V2X Modem (Rx) | 800 mW | 100% | 800 mW |
| C-V2X Modem (Tx, VAM) | 1200 mW | 5% | 60 mW |
| GNSS Module | 70 mW | 100% | 70 mW |
| IMU | 15 mW | 100% | 15 mW |
| Application Processor | 2500 mW | 30% (bursty) | 750 mW |
| Haptic Motors (worst case) | 900 mW | 10% | 90 mW |
| **Total** | | | **~1785 mW** |

Battery: 3000 mAh × 3.7 V = 11.1 Wh → approximately **6.2 hours** continuous use with adaptive power management.

### Communication Parameters Comparison

| Parameter | C-V2X PC5 (LTE-V2X) | C-V2X PC5 (NR-V2X) | DSRC (802.11p) |
|---|---|---|---|
| Frequency Band | 5.9 GHz | 5.9 GHz | 5.9 GHz |
| Channel Bandwidth | 10/20 MHz | 10/20/40 MHz | 10 MHz |
| Modulation | QPSK, 16-QAM | QPSK to 256-QAM | BPSK to 64-QAM |
| Max Data Rate | ~12 Mbps | ~50 Mbps | ~27 Mbps |
| Typical Range (LOS) | 300–450 m | 300–500 m | ~300 m |
| Air Interface Latency | < 5 ms | < 3 ms | < 2 ms |
| Resource Allocation | SPS (Mode 4) | Sensing-based (Mode 2) | CSMA/CA |
| 3GPP Release | Rel-14/15 | Rel-16/17 | IEEE 802.11p |

## Use Case

**Scenario: Visually impaired pedestrian crossing at a signalised intersection in Singapore**

Mdm Lim, who has advanced glaucoma and uses a white cane, approaches a signalised crosswalk at the junction of North Bridge Road and Victoria Street. She wears the SafeCross haptic belt under her jacket. As she stands at the kerb, the VRU Device acquires her position and begins receiving CAM broadcasts from six nearby C-V2X-equipped vehicles and SPaT data from the intersection RSU.

A bus turning left from Victoria Street approaches at 25 km/h with its left indicator active. The CRE computes a TTC of 4.2 s and classifies it as MEDIUM threat. Mdm Lim feels three gentle pulses per second on her left-front motor (M8), intuitively signalling "vehicle approaching from your left-front." She pauses before stepping off the kerb. As the bus completes its turn and clears the crosswalk, the vibration fades.

Simultaneously, the SPaT data shows the pedestrian green phase will end in 4 seconds. The front motor (M1) delivers a distinctive double-tap pattern, which Mdm Lim has learned to associate with "hurry or wait for the next phase." She decides to wait. Fifteen seconds later, the new pedestrian phase begins, no vehicles are on a conflicting approach, and all motors remain still — Mdm Lim steps confidently into the crossing.

This scenario illustrates how SafeCross provides non-visual, hands-free situational awareness by combining cooperative vehicle data with intuitive haptic encoding, complementing rather than replacing Mdm Lim's existing mobility skills with her white cane.

## AI Usage and Individual Reflection

### AI Tools Used

| AI Tool | Project Phases | How It Was Used |
|---|---|---|
| **Claude (Anthropic)** | Research, architecture design, literature review, pseudo-code review | Brainstorming V2P concepts, comparing communication technologies, reviewing TTC/PET algorithms, generating system architecture descriptions |
| **ChatGPT (OpenAI)** | Background research, standards summarisation | Summarising 3GPP release history, ETSI message structures, FCC spectrum regulatory background |

### Example Prompts and Outputs

#### Example 1: Brainstorming V2P Applications

**Prompt (to Claude):**
> "We are designing a novel V2P application for a module titled Car interconnects & Vehicular networks. The application should use C-V2X communication to improve pedestrian safety. We want to focus on a specific vulnerable road user group that has an unmet need not addressed by existing V2P prototypes. Can you suggest three original V2P application ideas with pros and cons for each?"

**Output (summarised):**
Claude suggested three ideas: (1) a C-V2X haptic glove for VI pedestrians, (2) a V2P speed advisory for elderly pedestrians at crosswalks, and (3) a wheelchair-mounted V2P device. For each, it provided a description, target user group, communication requirements, and feasibility assessment.

**Team evaluation:** Idea 1 was most promising but we changed the form factor from glove to waist-band after researching haptic perception literature. Ideas 2 and 3 were rejected due to overlap with existing infrastructure-based solutions.

#### Example 2: Comparing PC5 and Uu Latency

**Prompt (to Claude):**
> "Compare the end-to-end latency of C-V2X PC5 sidelink communication vs C-V2X Uu (cellular network) communication for a V2P safety alert. Include specific numbers from 3GPP specifications."

**Output (summarised):**
Claude stated PC5 sidelink provides < 5 ms air-interface latency, while Uu via cellular network incurs 50–200 ms depending on network load. It referenced 3GPP TS 22.186 for V2X latency requirements.

**Team evaluation:** Latency figures were consistent with lecture notes. However, Claude confused Mode 3 (network-scheduled) with Mode 4 (autonomous). Corrected from 3GPP TS 36.300.

#### Example 3: Generating TTC Pseudo-code

**Prompt (to Claude):**
> "Write a Python function to compute Time-to-Collision between a vehicle and pedestrian given their positions, speeds, and headings in a local ENU coordinate frame."

**Output (summarised):**
Claude generated a basic function that divided straight-line distance by vehicle speed. It did not account for closing speed (vector projection), pedestrian movement, or a safety buffer for vehicle width.

**Team evaluation:** The output was a useful skeleton but physically incorrect for crossing scenarios. We rewrote it using vector projection of relative velocity and added the safety buffer. Corrected version is in the Functions and Messages section.

### Identified Weaknesses and Hallucinations

#### Hallucination 1: Non-existent Message Standard
**Tool:** Claude
**Issue:** When asked about V2P message standards, Claude described a "Pedestrian Safety Message (PSM)" as part of the ETSI framework. This is incorrect — PSM is from SAE J2735 (US standard). The correct ETSI equivalent is the VRU Awareness Message (VAM) in ETSI TS 103 300-2.
**Correction:** Verified against the ETSI TS 103 300 specification directly.

#### Hallucination 2: FCC Spectrum Claim
**Tool:** ChatGPT
**Issue:** ChatGPT stated "the FCC completely banned DSRC in the 5.9 GHz band in 2020." Incorrect — the FCC's First Report and Order (FCC 20-305) reallocated the lower 45 MHz for unlicensed use but retained the upper 30 MHz (5.895–5.925 GHz) for ITS/V2X.
**Correction:** Read the actual FCC Order 20-305 and the ITS America 2024 report.

#### Hallucination 3: Overestimated GNSS Accuracy
**Tool:** Claude
**Issue:** Claude claimed standalone GNSS provides "1–2 m accuracy" in urban environments. This is overly optimistic for dense areas like Singapore's CBD, where multipath effects degrade accuracy to 3–5 m CEP.
**Correction:** Referenced published SLA/LTA positioning studies for Singapore.

### Individual Reflections


#### Winston Lee
I took the lead on the documentation, system architecture and message flow design. My main contribution was defining the protocol stack and ensuring our system uses fully standard ETSI ITS messages for interoperability. I used Claude extensively for initial research on V2X communication standards, which saved significant time in understanding the landscape of available technologies. However, I found that AI tools were unreliable for specific standard document numbers and version details, which always required manual verification against the original specifications. For instance, Claude conflated the SAE J2735 PSM with the ETSI VAM, which could have led us to reference an incorrect standard if left unchecked. The most valuable AI interaction was using Claude to compare PC5 and Uu latency characteristics, which helped our team converge quickly on the PC5-first architecture. While most design decisions were made collectively, I drove the standards and protocol-related entries in the decision log. I learnt that AI is most useful as a research accelerator for scoping out a problem space, but it cannot replace careful reading of primary standard documents.

#### Christopher Wong
My primary responsibility was the haptic feedback interface design, including motor selection, layout, and encoding scheme. I researched vibrotactile perception literature to determine the optimal number of motors and validated that 8-direction resolution is achievable on the torso, drawing on Jones and Sarter (2008) to justify our choice over the 6 motors that Claude initially suggested. I also sourced the specific hardware components (Precision Microdrives 310-117 ERM motors, TI DRV2605L drivers, TCA9548A multiplexer) and verified their specifications against datasheets, as AI-generated specs were often approximate or outdated. The team collectively discussed the threat-level thresholds, but I was responsible for translating those thresholds into concrete pulse rates and duty cycles that would be perceptually distinguishable. I used AI tools sparingly for this portion of the work because haptic perception is a fairly niche domain where general-purpose language models lack depth. My key learning was that hardware specifications such as motor response time and I2C bus limitations must always be verified from datasheets rather than taken at face value from AI outputs.

#### Yeo Hong Yi
I focused on the collision risk engine, including the TTC and PET computation algorithms and the core processing loop. Starting from the surrogate safety measures covered in CEG3006 lectures, I adapted the standard TTC formula to handle the specific geometry of a V2P crossing scenario, where the vehicle and pedestrian trajectories are often perpendicular rather than head-on. The initial pseudo-code generated by Claude used a straightforward distance-over-speed calculation, which does not account for crossing conflicts. I rewrote it to use vector projection of relative velocity, which correctly captures closing rate regardless of approach angle. Claude and ChatGPT were helpful for scaffolding the pseudo-code and for overall brainstorming sessions during the early stages of the project. I also contributed to the power budget analysis, where I discovered that Claude's estimate had omitted the C-V2X receiver's always-on 800 mW draw, which would have made our battery life projections overly optimistic. A key takeaway is that AI can complement nearly all of our workflows, but every numerical output needs to be independently verified, particularly for safety-critical applications.

#### Chai Yong Huat
I contributed to the use-case scenario, literature review, and haptic encoding scheme. Writing the use-case for Mdm Lim helped ground our technical design in a realistic human scenario and forced us to think through practical edge cases, such as what happens when the SPaT data indicates the pedestrian phase is about to end. For the literature review, I surveyed existing V2P solutions across three categories (on-board perception, infrastructure-based, and smartphone-centric) and identified the specific gap that SafeCross addresses. AI was extremely helpful in identifying relevant papers and pilot projects that I would not have found otherwise, but I always checked the original sources because AI sometimes attributed findings to the wrong paper or confused publication years. The team discussed all design decisions together, but I focused on the user-experience and accessibility considerations, ensuring that our system complements rather than replaces existing mobility skills such as the white cane. If this project were to continue, I would want to consult with actual VI users and orientation-and-mobility specialists to validate our haptic encoding assumptions.

#### Roshan Shaji Philip
I was responsible for the GNSS and positioning subsystem design, the privacy analysis for VAM broadcasting, and supporting the overall project documentation. On the positioning side, I investigated why standalone GNSS is insufficient for our use case in Singapore's urban environment and proposed the GNSS plus IMU plus MAP-matching approach, which avoids adding Uu dependency in line with our PC5-only safety path decision. The GNSS accuracy issue was a key learning moment: Claude claimed standalone GNSS achieves 1 to 2 m accuracy in urban settings, when published SLA and LTA studies for Singapore indicate that 3 to 5 m is more realistic in the CBD. Had we accepted that figure without verification, our collision risk engine would have been built on a flawed assumption. I also designed the pseudonym rotation scheme for privacy protection, drawing on the ETSI TS 102 940 security framework, where Claude was accurate and saved considerable time in understanding the SCMS mechanism. I learnt that for safety-critical systems, every numerical parameter must be independently verified, and that documenting why we rejected an option is just as important as describing the option we chose.


## References

[1] R. Wall Emerson, K. Naghshineh, J. Hapeman, and W. Wiener, "A Pilot Study of Pedestrians with Visual Impairments Detecting Traffic Gaps and Surges Containing Hybrid Vehicles," *Transportation Research Part F: Traffic Psychology and Behaviour*, vol. 14, no. 2, pp. 117–127, 2011.

[2] J.-R. Rizzo et al., "The VIS4ION project: Design and pilot testing of a wearable, multimodal sensing and feedback system for the visually impaired," in *Proceedings of IEEE EMBS*, 2018.

[3] C. Silvestru et al., "Usability of an AI-based independent pedestrian navigation device designed for blind people," *European Journal of Public Health*, vol. 34, suppl. 3, 2024.

[4] Maryland Department of Transportation, "Dual Mode DSRC/C-V2X I2V Pedestrian System Pilot," 2019–2021.

[5] New York City DOT, "PED-SIG: Smartphone Application for Signalized Intersection Information," 2020.

[6] A. Anaya et al., "Vehicle-to-Pedestrian Communication for Vulnerable Road Users: Survey, Design Considerations, and Challenges," *Sensors*, vol. 19, no. 2, 2019.

[7] ETSI EN 302 637-2, "Intelligent Transport Systems (ITS); Vehicular Communications; Part 2: Specification of Cooperative Awareness Basic Service," v1.4.1, 2019.

[8] ETSI EN 302 637-3, "Intelligent Transport Systems (ITS); Vehicular Communications; Part 3: Specifications of Decentralized Environmental Notification Basic Service," v1.3.1, 2019.

[9] ETSI TS 103 300-2, "Intelligent Transport Systems (ITS); Vulnerable Road Users (VRU) awareness; Part 2: Functional Architecture and Requirements," v2.1.1, 2021.

[10] SAE J2735, "V2X Communications Message Set Dictionary," 2023.

[11] 3GPP TS 36.300, "Evolved Universal Terrestrial Radio Access (E-UTRA); Overall description," Rel-14.

[12] 3GPP TS 22.186, "Service requirements for enhanced V2X scenarios," Rel-16.

[13] Qualcomm Technologies, "Cellular-V2X Technology Overview," 2019.

[14] L. A. Jones and N. B. Sarter, "Tactile Displays: Guidance for Their Design and Application," *Human Factors*, vol. 50, no. 1, pp. 90–111, 2008.

[15] FCC, "First Report and Order, Use of the 5.850–5.925 GHz Band," FCC 20-305, November 2020.

[16] ITS America, "Future of V2X in 5.9 GHz Report," May 2024.

[17] F. S. Ricci et al., "Navigation Training for Persons With Visual Disability Through Multisensory Assistive Technology," *JMIR Rehabilitation and Assistive Technologies*, vol. 11, e55776, 2024.

[18] Singapore Epidemiology of Eye Disease (SEED) Study, Singapore Eye Research Institute / SNEC.

---
