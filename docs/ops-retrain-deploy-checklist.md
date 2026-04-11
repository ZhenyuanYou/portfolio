# Ops Checklist — Safe Retrain & Deploy Cycle

## Prerequisites
- [ ] Assign cycle owner (Name / Team)
- [ ] Record baseline model **M0** version and verify it is stable
- [ ] Ensure fixed validation set `V_fixed` is available and immutable
- [ ] Verify logging, monitoring, and emergency-stop hooks are tested

## Data Collection & Preparation
- [ ] Snapshot data buffer at retrain trigger (timestamped)
- [ ] Reserve `V_recent` from buffer (define N, e.g., 200 episodes)
- [ ] Apply data filters:
  - [ ] Remove corrupt or missing-sensor episodes
  - [ ] Remove low-quality labels and duplicates
  - [ ] Exclude episodes with known firmware/sensor bugs
- [ ] Ensure class balance or document imbalance and mitigation plan
- [ ] Version dataset and record provenance (who / when / why)

## Trigger Conditions
- [ ] Define drift thresholds:
  - 7-day rolling success rate drop ≥ 3%
  - OR vision confidence distribution shift (KS test p < 0.01)
- [ ] Confirm human review before starting retrain

## Training Candidate Model (Mcand — Offline)
- [ ] Use versioned training script and fixed random seed
- [ ] Record hyperparameters
- [ ] Train on versioned dataset
- [ ] Store model artifact and training logs
- [ ] Run sanity checks:
  - [ ] No NaNs
  - [ ] Stable loss curve
- [ ] Archive environment (code, libraries, seed)

## Offline Evaluation
- [ ] Evaluate on `V_fixed` and record metrics + confidence intervals
- [ ] Evaluate on `V_recent` and record metrics + confidence intervals
- [ ] Run domain-randomized simulations (e.g., 1k–2k episodes)
- [ ] Log safety events
- [ ] Run rule-based safety checks (force/speed limits)

## Promotion Criteria (to Canary)
- [ ] `V_fixed` success rate ≥ baseline (no regression)
- [ ] `V_recent` success rate ≥ baseline + improvement target
- [ ] False-pick and safety-violation rates within bounds
- [ ] No critical safety violations in simulation

## Canary Deployment
- [ ] Deploy **Mcand** to canary robots
- [ ] Enforce hard safety limits
- [ ] Keep human supervision enabled
- [ ] Ensure rollback capability is ready
- [ ] Record cohort identifiers and start time

## Canary Monitoring & Rollback Triggers
### Metrics to Monitor
- Success rate
- False-pick rate
- Safety interrupts
- Sensor anomaly rate
- Operator reports

### Immediate Rollback Triggers
- [ ] Success rate drops below absolute threshold
- [ ] Relative regression exceeds allowed threshold
- [ ] Physical safety interrupt occurs
- [ ] Reward-hacking pattern or false-pick spike appears

## Post-Rollback Analysis
- [ ] Triage logs and failure episodes
- [ ] Reproduce issue in simulation
- [ ] Determine root cause: data / model / infra / sensor / firmware
- [ ] Apply fix and document mitigation before re-initiation

## Gradual Rollout
- [ ] Expand from canary to partial rollout
- [ ] Continue monitoring during each expansion step
- [ ] Keep rollback rules active throughout rollout

## Ongoing Operations
- [ ] Schedule retraining cadence or drift triggers
- [ ] Maintain model and dataset registry
- [ ] Run regular safety audits and edge-case simulations
- [ ] Maintain human-in-the-loop QA

## Key Principles
- Never allow unreviewed full online model updates
- Enforce hard safety constraints outside model outputs
- Favor conservative canaries, clear thresholds, and strong logging
