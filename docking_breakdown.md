# ACT + Force: Cable Docking Policy Design Specification

## Overview

This document specifies the design for a learning-based cable docking policy using **ACT (Action Chunking with Transformers)** with force/torque feedback. The policy handles the final insertion phase after the robot arm has been brought near the target socket (Team A's responsibility).

### Scope

- **In scope**: Docking the cable connector into the socket once near (~5-10cm)
- **Out of scope**: Approach phase (Team A), cable grasping (already gripped)

### Goals

1. Maximize Tier 3 scoring (75 points for successful insertion)
2. Avoid force penalties (>20N for >1s = -12 points)
3. Fast insertion for duration score (≤5s = 12 points)
4. Smooth trajectory for jerk score (low jerk = 6 points)

---

## Architecture

### Model: ACT (Action Chunking with Transformers)

Based on the existing `RunACT.py` implementation with added force/torque input.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ACT + Force Docking Policy                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────────┐  │
│  │ Left Camera │   │Center Camera│   │Right Camera │   │ Force/Torque    │  │
│  │   224×224   │   │   224×224   │   │   224×224   │   │   6D vector     │  │
│  └──────┬──────┘   └──────┬──────┘   └──────┬──────┘   └────────┬────────┘  │
│         │                 │                 │                   │           │
│         ▼                 ▼                 ▼                   │           │
│  ┌─────────────────────────────────────────────────┐            │           │
│  │              ResNet-18 Encoders (×3)            │            │           │
│  │              (pretrained, shared weights)       │            │           │
│  └─────────────────────────┬───────────────────────┘            │           │
│                            │                                    │           │
│                            ▼                                    │           │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                     Feature Concatenation                             │  │
│  │   [img_emb_L, img_emb_C, img_emb_R, joint_state(7D), F/T(6D)]        │  │
│  └─────────────────────────────────┬─────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    ACT Transformer Decoder                            │  │
│  │                    (Action Chunking = 10 steps)                       │  │
│  └─────────────────────────────────┬─────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                      Action Output (7D × 10)                          │  │
│  │              [dx, dy, dz, droll, dpitch, dyaw, gripper] × chunk_size  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## State Space (Inputs)

### Visual Inputs (Existing)

| Input | Dimensions | Source | Processing |
|-------|-----------|--------|------------|
| Left camera | 224×224×3 | `/left_camera/image` | Resize from 640×480, normalize |
| Center camera | 224×224×3 | `/center_camera/image` | Resize from 640×480, normalize |
| Right camera | 224×224×3 | `/right_camera/image` | Resize from 640×480, normalize |

Image normalization: ImageNet mean/std (same as RunACT.py)

### Proprioceptive Inputs (Existing)

| Input | Dimensions | Source | Processing |
|-------|-----------|--------|------------|
| Joint positions | 6D | `/joint_states.position` | Normalize to [-1, 1] using joint limits |
| Gripper position | 1D | `/joint_states.position[6]` | Normalize to [0, 1] |

### Force/Torque Input (NEW)

| Input | Dimensions | Source | Processing |
|-------|-----------|--------|------------|
| Force/Torque | 6D | `/fts_broadcaster/wrench` | See normalization below |

**Force/Torque Normalization:**

```python
def normalize_ft(wrench):
    """Normalize force/torque to [-1, 1] range."""
    MAX_FORCE = 50.0   # Newtons
    MAX_TORQUE = 5.0   # Newton-meters
    
    force = np.clip(wrench[:3] / MAX_FORCE, -1.0, 1.0)
    torque = np.clip(wrench[3:] / MAX_TORQUE, -1.0, 1.0)
    
    return np.concatenate([force, torque])
```

**Rationale for 50N/5Nm limits:**
- Scoring penalizes >20N sustained force
- 50N gives headroom for transient spikes
- Typical insertion forces are 5-15N

### Total State Dimensions

| Component | Dimensions |
|-----------|-----------|
| Image embeddings (3 × 512) | 1536 |
| Joint state | 7 |
| Force/Torque | 6 |
| **Total (after encoding)** | **1549** |

---

## Action Space (Outputs)

### Cartesian Velocity Mode (Primary)

| Dimension | Description | Range | Units |
|-----------|-------------|-------|-------|
| dx | Linear velocity X | [-0.1, 0.1] | m/s |
| dy | Linear velocity Y | [-0.1, 0.1] | m/s |
| dz | Linear velocity Z | [-0.1, 0.1] | m/s |
| droll | Angular velocity roll | [-0.5, 0.5] | rad/s |
| dpitch | Angular velocity pitch | [-0.5, 0.5] | rad/s |
| dyaw | Angular velocity yaw | [-0.5, 0.5] | rad/s |
| gripper | Gripper command | [0, 1] | normalized |

### Action Chunking

- **Chunk size**: 10 actions
- **Execution**: Execute first action, re-plan every step (or temporal ensemble)
- **Frequency**: ~50 Hz inference, ~100 Hz control

### Alternative: Pose Targets (Future Option)

If velocity control proves insufficient:

| Dimension | Description | Range |
|-----------|-------------|-------|
| x, y, z | Target position | workspace limits |
| qx, qy, qz, qw | Target orientation | unit quaternion |
| gripper | Gripper command | [0, 1] |

The ACT architecture supports both; only data labeling changes.

---

## Data Collection

### Method 1: CheatCode Demonstrations (Automated)

**Goal**: Generate 500-1000 episodes of successful docking. But we can start with just 100 to test integrity. (ie, test by overfitting the model first)

**Modifications to CheatCode.py:**

1. **Starting pose**: Begin from a pose ~5-10cm above/near the socket
   - Simulates handoff from Team A
   - Randomize starting position within ±3cm

2. **Data logging**: Add force/torque to recorded data
   ```python
   # Add to observation recording
   observation['force_torque'] = self.get_force_torque()
   ```

3. **Episode filtering**: 
   - Discard episodes with >20N sustained force
   - Discard episodes with collisions
   - Keep only successful insertions

**Domain Randomization:**

| Parameter | Range | Purpose |
|-----------|-------|---------|
| Starting pose offset | ±3cm XYZ, ±5° orientation | Robustness to handoff variation |
| Socket position | Per trial config | Generalization |
| Lighting (if applicable) | Varied | Visual robustness |

### Method 2: SpaceMouse Teleoperation (Human Demos)

**Goal**: 50-100 high-quality demos for edge cases such as mistakes, recovery, etc.

**Setup:**
- Use `aic_utils` SpaceMouse teleoperation support
- Operator performs gentle, force-aware insertions
- Captures nuanced reactive behaviors

**When to use:**
- After initial CheatCode training
- For failure cases the automated demos don't cover

### Data Format (LeRobot Compatible)

```python
{
    # Images
    "observation.images.left": np.ndarray,      # (224, 224, 3), uint8
    "observation.images.center": np.ndarray,    # (224, 224, 3), uint8
    "observation.images.right": np.ndarray,     # (224, 224, 3), uint8
    
    # Proprioception
    "observation.state": np.ndarray,            # (7,), float32 - joints + gripper
    
    # Force/Torque (NEW)
    "observation.force_torque": np.ndarray,     # (6,), float32 - [Fx,Fy,Fz,Tx,Ty,Tz]
    
    # Actions
    "action": np.ndarray,                       # (7,), float32 - cartesian velocity
    
    # Metadata
    "timestamp": float,
    "episode_id": int,
    "step_id": int,
}
```

---

## Training Configuration

### Model Hyperparameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Architecture | ACT | From LeRobot |
| Image encoder | ResNet-18 | Pretrained on ImageNet |
| Encoder output dim | 512 | Per camera |
| Hidden dimension | 512 | Transformer |
| Number of layers | 4 | Transformer decoder |
| Number of heads | 8 | Multi-head attention |
| Chunk size | 10 | Action sequence length |
| Dropout | 0.1 | Regularization |

### Training Hyperparameters

| Parameter | Value |
|-----------|-------|
| Batch size | 64 |
| Learning rate | 1e-4 |
| Optimizer | AdamW |
| Weight decay | 1e-4 |
| LR scheduler | Cosine annealing |
| Training steps | 100k-200k |
| Gradient clipping | 1.0 |

### Force Integration

| Parameter | Value | Notes |
|-----------|-------|-------|
| Input fusion | Early concatenation | With state vector |
| Normalization | Divide by [50N, 5Nm] | Clamp to [-1, 1] |
| History window | 1 (current only) | Optional: last 5 readings |

### Loss Function

Standard ACT loss (from LeRobot)

---

## Evaluation Metrics

### Primary: Scoring Alignment

| Metric | Target | Scoring Impact |
|--------|--------|----------------|
| Insertion success rate | >90% | Tier 3: 75 pts |
| Max sustained force | <20N | Avoid -12 penalty |
| Collision rate | 0% | Avoid -24 penalty |
| Avg insertion time | <10s | Tier 2: ~10 pts |
| Avg jerk | <25 m/s³ | Tier 2: ~3 pts |

### Secondary: Training Metrics

| Metric | Purpose |
|--------|---------|
| Action prediction MSE | Training convergence |
| Validation loss | Overfitting detection |
| Episode success rate (sim) | Policy quality |

---

## Implementation Roadmap

### Phase 1: Environment Setup (Days 1-3)

**Tasks:**
- [ ] Install LeRobot and verify ACT training works
- [ ] Set up Gazebo simulation with AIC toolkit
- [ ] Verify CheatCode.py runs and completes insertions
- [ ] Set up data logging infrastructure
- [ ] Create conda/pixi environment for training

**Deliverables:**
- Working simulation environment
- Verified CheatCode baseline
- Data logging script

### Phase 2: Data Collection Stack (Days 4-7)

**Tasks:**
- [ ] Modify CheatCode to start from "near socket" pose
- [ ] Add force/torque to data recording
- [ ] Implement domain randomization for starting poses
- [ ] Generate 50-80 demonstration episodes
- [ ] Implement episode filtering (remove failures)
- [ ] Convert data to LeRobot format

**Deliverables:**
- Clean demonstration episodes
- LeRobot-compatible dataset

### Phase 3: Model Training (Days 8-14)

**Tasks:**
- [ ] Extend ACT model to accept force/torque input
- [ ] Train initial model on collected data
- [ ] Implement evaluation in simulation
- [ ] Iterate on hyperparameters
- [ ] Monitor for overfitting

**Deliverables:**
- Trained ACT+Force model
- Training logs and curves
- Initial simulation results

### Phase 4: Evaluation & Tuning (Days 15-21)

**Tasks:**
- [ ] Run full scoring evaluation (all tiers)
- [ ] Analyze failure cases
- [ ] Add targeted domain randomization based on failures
- [ ] Collect additional demos for edge cases (SpaceMouse)
- [ ] Retrain with augmented data
- [ ] Tune action clipping / smoothing for jerk score

**Deliverables:**
- Scoring report across trial configurations
- Failure analysis document
- Improved model

### Phase 5: Integration Testing (Days 22-25)

**Tasks:**
- [ ] Define handoff interface with Team A
- [ ] Test full pipeline (approach → docking)
- [ ] Verify scoring across all trial configurations
- [ ] Stress test with randomized scenarios

**Deliverables:**
- Integration test report
- Handoff specification document

### Phase 6: Submission Preparation (Days 26-28)

**Tasks:**
- [ ] Package policy into Docker container
- [ ] Verify submission requirements met
- [ ] Run final scoring locally
- [ ] Document any known limitations
- [ ] Submit

**Deliverables:**
- Submission-ready Docker image
- Final scoring report

---

## Handoff Interface (Team A → Docking Team)

### To Be Defined

The handoff from Team A (approach) to the docking policy needs specification:

**Option A: Pose-based trigger**
- When TCP reaches predefined pre-insertion pose
- Team A publishes "ready" signal

**Option B: Distance threshold**
- When plug is within X cm of socket
- Docking policy monitors distance

**Option C: Service call**
- Team A calls service when ready
- Docking policy activates

**Recommendation**: Start with Option B (distance threshold) for simplicity or Option C for determinism, refine based on integration testing.

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Force feedback doesn't help | Ablation study: compare with/without force |
| Insufficient training data | SpaceMouse demos for edge cases |
| Slow inference | Profile and optimize; reduce image resolution if needed |
| Sim-to-real gap (future) | Domain randomization during training |
| Action space unsuitable | Design supports switching to pose targets |

---

## References

- Existing policy: `aic_example_policies/aic_example_policies/ros/RunACT.py`
- CheatCode baseline: `aic_example_policies/aic_example_policies/ros/CheatCode.py`
- Scoring documentation: `docs/scoring.md`
- AIC Interfaces: `docs/aic_interfaces.md`
- Controller documentation: `docs/aic_controller.md`

---

## Changelog

| Date | Author | Changes |
|------|--------|---------|
| 2026-04-01 | Brainstorm session | Initial design specification |

