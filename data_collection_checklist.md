# AIC Data Collection Checklist
## Cable Docking Policy Training Dataset

---

## Overview

This checklist defines the systematic data collection requirements for training a robust cable docking policy using ACT (Action Chunking with Transformers) with force/torque feedback for the AI for Industry Challenge.

**Episode Requirements:**
- Start with ~5 episodes per configuration
- Recommended: 10+ episodes per configuration for better performance

---

## Collection Methods

### Method 1: CheatCode Demonstrations (Automated)
- **Purpose:** Generate bulk successful insertion demonstrations
- **Starting Position:** 2-3cm from socket with ±1cm randomization
- **Data Recorded:** RGB images (3 cameras), joint states, force/torque, actions
- **Filtering:** Remove episodes with >20N sustained force or collisions
<!-- - **Advantages:** High volume, consistent quality, fast generation -->

### Method 2: SpaceMouse Teleoperation (Human Demonstrations)  
- **Purpose:** Capture edge cases, recovery behaviors, nuanced interactions
- **Operator Requirements:** Gentle, force-aware insertions
- **Data Recorded:** RGB images (3 cameras), joint states, force/torque, actions
- **Prerequisites:** Tare F/T sensor at start of each episode using:
  ```bash
  ros2 service call /aic_controller/tare_force_torque_sensor std_srvs/srv/Trigger
  ```
- **Advantages:** Captures human strategies, handles failure recovery

---

## Test Bench Configurations

### Configuration Categories

Each configuration varies the following parameters:
1. **Cable Type** (2 variants)
2. **Plug Type Being Inserted** (2 types)
3. **Socket/Port Type** (2 types)  
4. **Task Board Pose** (randomized)
5. **Port Position on Rails** (randomized)
6. **Starting Robot Pose** (near-socket with randomization)

---

## Detailed Configuration Matrix

### Group A: SFP Module → NIC Card Insertions

#### Config A1: SFP Module to NIC Card (Rail 0, Port 0)
- **Cable:** `sfp_sc_cable`
- **Plug in Hand:** SFP Module (`sfp_tip`)
- **Target Port:** `sfp_port_0` on `nic_card_0`
- **Rail:** `nic_rail_0`
- **NIC Card Translation:** Randomized within [-0.0215, 0.0234] meters
- **NIC Card Yaw:** Randomized within [-10°, +10°] (≈ ±0.175 rad)
- **Task Board Pose:** 
  - Position: x ∈ [0.10, 0.20], y ∈ [-0.25, -0.15], z ≈ 1.14
  - Yaw: ∈ [2.8, 3.5] rad (≈ 160-200°)
- **Gripper Offset:** x: 0.0, y: 0.015385, z: 0.04245, roll: 0.4432, pitch: -0.4838, yaw: 1.3303
- **Teleoperation:** Manual/SpaceMouse or CheatCode

#### Config A2: SFP Module to NIC Card (Rail 0, Port 1)
- **Cable:** `sfp_sc_cable`
- **Plug in Hand:** SFP Module (`sfp_tip`)
- **Target Port:** `sfp_port_1` on `nic_card_0`
- **Rail:** `nic_rail_0`
- **NIC Card Translation:** Randomized within [-0.0215, 0.0234] meters
- **NIC Card Yaw:** Randomized within [-10°, +10°]
- **Task Board Pose:** (Same randomization as A1)
- **Gripper Offset:** (Same as A1)
- **Teleoperation:** Manual/SpaceMouse or CheatCode

#### Config A3: SFP Module to NIC Card (Rail 1)
- **Cable:** `sfp_sc_cable`
- **Plug in Hand:** SFP Module (`sfp_tip`)
- **Target Port:** `sfp_port_0` on `nic_card_1`
- **Rail:** `nic_rail_1`
- **NIC Card Translation:** Randomized within [-0.0215, 0.0234] meters
- **NIC Card Yaw:** Randomized within [-10°, +10°]
- **Task Board Pose:** (Same randomization as A1)
- **Gripper Offset:** (Same as A1)
- **Teleoperation:** Manual/SpaceMouse or CheatCode

#### Config A4: SFP Module to NIC Card (Rail 2)
- **Cable:** `sfp_sc_cable`
- **Plug in Hand:** SFP Module (`sfp_tip`)
- **Target Port:** `sfp_port_0` on `nic_card_2`
- **Rail:** `nic_rail_2`
- **NIC Card Translation:** Randomized
- **NIC Card Yaw:** Randomized within [-10°, +10°]
- **Task Board Pose:** (Same randomization as A1)
- **Gripper Offset:** (Same as A1)
- **Teleoperation:** Manual/SpaceMouse or CheatCode

#### Config A5: SFP Module to NIC Card (Rail 3)
- **Cable:** `sfp_sc_cable`
- **Plug in Hand:** SFP Module (`sfp_tip`)
- **Target Port:** `sfp_port_0` on `nic_card_3`
- **Rail:** `nic_rail_3`
- **NIC Card Translation:** Randomized
- **NIC Card Yaw:** Randomized within [-10°, +10°]
- **Task Board Pose:** (Same randomization as A1)
- **Gripper Offset:** (Same as A1)
- **Teleoperation:** Manual/SpaceMouse or CheatCode

#### Config A6: SFP Module to NIC Card (Rail 4)
- **Cable:** `sfp_sc_cable`
- **Plug in Hand:** SFP Module (`sfp_tip`)
- **Target Port:** `sfp_port_0` on `nic_card_4`
- **Rail:** `nic_rail_4`
- **NIC Card Translation:** Randomized
- **NIC Card Yaw:** Randomized within [-10°, +10°]
- **Task Board Pose:** (Same randomization as A1)
- **Gripper Offset:** (Same as A1)
- **Teleoperation:** Manual/SpaceMouse or CheatCode

---

### Group B: SC Plug → SC Port Insertions

#### Config B1: SC Plug to SC Port (Rail 0)
- **Cable:** `sfp_sc_cable_reversed`
- **Plug in Hand:** SC Plug (`sc_tip`)
- **Target Port:** `sc_port_base` on `sc_port_0`
- **Rail:** `sc_rail_0`
- **SC Port Translation:** Randomized within [-0.06, 0.055] meters
- **Task Board Pose:**
  - Position: x ∈ [0.12, 0.22], y ∈ [-0.05, 0.05], z ≈ 1.14
  - Yaw: ∈ [2.8, 3.2] rad
- **Gripper Offset:** x: 0.0, y: 0.015385, z: 0.04045, roll: 0.4432, pitch: -0.4838, yaw: 1.3303
- **Teleoperation:** Manual/SpaceMouse or CheatCode

#### Config B2: SC Plug to SC Port (Rail 1)
- **Cable:** `sfp_sc_cable_reversed`
- **Plug in Hand:** SC Plug (`sc_tip`)
- **Target Port:** `sc_port_base` on `sc_port_1`
- **Rail:** `sc_rail_1`
- **SC Port Translation:** Randomized within [-0.06, 0.055] meters
- **Task Board Pose:** (Same randomization as B1)
- **Gripper Offset:** (Same as B1)
- **Teleoperation:** Manual/SpaceMouse or CheatCode

---

### Group C: Edge Cases and Variations

#### Config C1: SFP with Grasp Offset Variation (+2mm, +0.04 rad)
- **Cable:** `sfp_sc_cable`
- **Plug in Hand:** SFP Module
- **Target:** Any NIC Card SFP port
- **Grasp Perturbation:** +2mm translation, +0.04 rad rotation from nominal
- **Task Board Pose:** Randomized
- **Teleoperation:** Primarily SpaceMouse (human demonstrations)

#### Config C2: SC with Grasp Offset Variation (+2mm, +0.04 rad)
- **Cable:** `sfp_sc_cable_reversed`
- **Plug in Hand:** SC Plug
- **Target:** Any SC port
- **Grasp Perturbation:** +2mm translation, +0.04 rad rotation from nominal
- **Task Board Pose:** Randomized
- **Teleoperation:** Primarily SpaceMouse (human demonstrations)

#### Config C3: Challenging Board Orientations (Extreme Yaw)
- **Cable:** Both types
- **Plug:** Both SFP and SC
- **Target:** Various ports
- **Task Board Yaw:** Extreme angles (2.6-2.8 rad or 3.4-3.6 rad)
- **Teleoperation:** CheatCode + SpaceMouse for failures

#### Config C4: Partial Occlusion Scenarios
- **Cable:** Both types
- **Plug:** Both SFP and SC
- **Target:** Ports with adjacent components populated
- **Task Board:** Multiple NIC cards or SC ports present simultaneously
- **Teleoperation:** Primarily SpaceMouse

#### Config C5: Recovery from Near-Miss
- **Cable:** Both types
- **Plug:** Both SFP and SC
- **Scenario:** Start with intentional slight misalignment (1-2mm offset from ideal)
- **Purpose:** Teach policy to recover from imperfect approach handoff
- **Teleoperation:** SpaceMouse only (requires human judgment)

---

## Data Quality Requirements

### Episode Success Criteria
- ✅ Successful insertion detected (insertion_event published)
- ✅ No sustained forces >20N for >1 second
- ✅ No collisions with off-limit items
- ✅ Episode duration <180 seconds
- ✅ All sensor data streams synchronized and complete

### Episode Rejection Criteria
- ❌ Force spikes >50N (equipment safety)
- ❌ Collision with task board mounting or other off-limit items
- ❌ Incomplete sensor data (camera dropout, F/T sensor issues)
- ❌ Gripper drops cable mid-episode
- ❌ Episode timeout without successful insertion

---

## Data Recording Format

All episodes must include the following synchronized data streams:

### Visual Data
- **Left Camera:** 224×224×3 RGB (resized from 640×480), `/left_camera/image`
- **Center Camera:** 224×224×3 RGB (resized from 640×480), `/center_camera/image`
- **Right Camera:** 224×224×3 RGB (resized from 640×480), `/right_camera/image`
- **Frequency:** ~30 Hz (camera rate)

### Proprioceptive Data
- **Joint Positions:** 7D vector (6 arm joints + gripper), `/joint_states`
- **Frequency:** ~100 Hz (robot control rate)

### Force/Torque Data (NEW - Critical for ACT+Force model)
- **Force:** 3D vector [Fx, Fy, Fz] in Newtons
- **Torque:** 3D vector [Tx, Ty, Tz] in Newton-meters
- **Source:** `/fts_broadcaster/wrench`
- **Frequency:** ~100 Hz
- **Normalization:** Clip to ±50N for force, ±5Nm for torque

### Action Labels
- **Format:** 7D Cartesian velocity [dx, dy, dz, droll, dpitch, dyaw, gripper_cmd]
- **Range:** Linear: ±0.1 m/s, Angular: ±0.5 rad/s, Gripper: [0, 1]
- **Frequency:** ~50 Hz (action command rate)

### Metadata
- Episode ID
- Configuration ID (e.g., "A1", "B2", "C5")
- Timestamp
- Success flag
- Task board pose (ground truth)
- Port position on rail (ground truth)
- Cable type
- Plug/port types

---

## Data Organization Structure

Organize data by configuration type and scenario:
- Group SFP insertions by rail and port
- Group SC insertions by rail
- Separate edge cases into distinct categories
- Each episode should contain: observations (images, state, force/torque), actions, and metadata

---

## Collection Progress Tracking

### Phase 1: Core Configurations (Priority)
- [ ] Config A1: SFP → NIC Rail 0 Port 0 (____ episodes)
- [ ] Config A2: SFP → NIC Rail 0 Port 1 (____ episodes)
- [ ] Config B1: SC → SC Port Rail 0 (____ episodes)
- [ ] Config B2: SC → SC Port Rail 1 (____ episodes)

### Phase 2: Full Rail Coverage
- [ ] Config A3: SFP → NIC Rail 1 (____ episodes)
- [ ] Config A4: SFP → NIC Rail 2 (____ episodes)
- [ ] Config A5: SFP → NIC Rail 3 (____ episodes)
- [ ] Config A6: SFP → NIC Rail 4 (____ episodes)

### Phase 3: Edge Cases
- [ ] Config C1: SFP grasp offset (____ episodes)
- [ ] Config C2: SC grasp offset (____ episodes)
- [ ] Config C3: Extreme orientations (____ episodes)
- [ ] Config C4: Occlusion scenarios (____ episodes)
- [ ] Config C5: Recovery behaviors (____ episodes)

### Phase 4: Validation Set
- [ ] Take 15% of the collected data for validation testing later

---

## Quality Assurance Checklist

Before considering dataset collection complete:

### Data Integrity
- [ ] All episodes have complete sensor streams (no missing frames)
- [ ] Force/torque data properly tared at episode start
- [ ] Timestamps synchronized across all modalities
- [ ] No corrupted HDF5 files

### Coverage
- [ ] All rails represented for SFP insertions
- [ ] Both SC rails represented
- [ ] Grasp offset variations included
- [ ] Range of task board orientations covered
- [ ] Success rate >85% in collected demonstrations

### Diversity
- [ ] Task board poses span full randomization range
- [ ] Port translations cover rail limits
- [ ] Mix of CheatCode and human demonstrations
- [ ] Edge cases and recovery behaviors represented

### Metadata
- [ ] Configuration labels accurate and consistent
- [ ] Ground truth poses recorded for all episodes
- [ ] Success/failure flags verified
- [ ] Data collection method tagged (CheatCode vs. SpaceMouse)

---

## References

- **Docking Breakdown:** `/home/deepansh/aic_ws/aic/docking_breakdown.md`
- **Scene Description:** `/home/deepansh/aic_ws/aic/docs/scene_description.md`
- **Qualification Phase:** `/home/deepansh/aic_ws/aic/docs/qualification_phase.md`
- **Sample Config:** `/home/deepansh/aic_ws/aic/aic_engine/config/sample_config.yaml`
- **Task Board Spec:** `/home/deepansh/aic_ws/aic/docs/task_board_description.md`

---

**Last Updated:** 2026-04-06
**Version:** 1.0
