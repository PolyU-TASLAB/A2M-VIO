# Method and baseline descriptions

This page explains the algorithms used in the reported experiments. Source code
is withheld during the manuscript submission process and will be released after
publication. Descriptions do not imply that third-party code or weights are
redistributed in this showcase.

## A2M-VIO

All three learned mean models use temporal convolutional networks (TCNs).
Training-only statistics standardize inputs and targets; predictions and
covariances are converted back to physical units before fusion.

- **TransNet** predicts body-frame specific force. Supervised pretraining uses
  fitted derivative labels. Pose-level refinement uses a 0.2 s position
  weak-form objective without requiring pointwise acceleration truth.
- **RotNet** predicts angular acceleration. Its pose refinement uses a 1 s
  rollout with equally weighted cumulative attitude checkpoints every 0.1 s.
  Gyro values and predicted angular accelerations are soft observations of an
  angular-velocity spline and its derivative, respectively.
- **DragNet** predicts instantaneous body-frame velocity from IMU and actuation
  histories. After mean training, the mean/features are frozen and an MLP
  log-variance head is trained with Gaussian NLL.

The two factors are:

1. **Dynamics preintegration factor:** a six-dimensional relative velocity and
   position constraint. Predicted force and spline-derived rotation are
   propagated with midpoint integration. The spline observation model uses an
   independence approximation; correlations induced by shared spline
   coefficients and reused force predictions within an interval remain in the
   propagated covariance.
2. **Learned body-velocity factor:** a three-dimensional velocity observation
   with DragNet's predicted diagonal covariance.

TransNet/RotNet use fixed calibrated diagonal model variances. Two external
multipliers adjust the full dynamics-factor covariance and the learned
velocity-factor covariance relative to the fixed visual/IMU configuration.
These are effective fusion weights, not claims of exact physical-noise recovery.

## Estimation backbone

[Kimera-VIO](https://github.com/MIT-SPARK/Kimera-VIO) supplies the visual–inertial
estimation backbone. [GTSAM](https://github.com/borglab/gtsam) supplies the
factor-graph optimization framework. The same visual/IMU setup is used for the
four ablation conditions:

| Display name | Dynamics preintegration | Learned body velocity |
|:---|:---:|:---:|
| Kimera-VIO | No | No |
| VIO+Dyn. | Yes | No |
| VIO+Vel. | No | Yes |
| VIO+Full (ours) | Yes | Yes |

## Learning-based baselines

**[AI-IO](https://github.com/SJTU-ViSYS-team/AI-IO)** is an aerodynamics-inspired
inertial-odometry method using a Transformer-based velocity model with actuation
and inertial measurements. The velocity comparison uses the official
implementation trained with the same split. The localization comparison runs
AI-IO as standalone odometry, not as an additional Kimera-VIO factor.

**[TartanIMU+LoRA](https://github.com/superxslam/TartanIMU)** adapts a pretrained
inertial model using LoRA on the nine Train flights, with Validation-only
selection. The velocity targets are instantaneous token-endpoint body velocities.
The evaluated causal adapter uses only available IMU measurements, starts
prediction after 1 s, and grows its preceding context to 10 s. It does not use a
zero-phase filter or Test data for adaptation. Its localization results are
standalone odometry, not fusion with Kimera-VIO.

Different pretraining, model capacities, input information, and training recipes
are retained and disclosed. These comparisons are not architecture-only ablations.
The linked upstream repositories may evolve; reported scores refer to the
experiment versions, not untested future upstream revisions.

## Motion-prediction baselines

- **Identified model:** an analytical dynamics model fitted on the nine Train
  flights, rather than a model using unavailable ground-truth vehicle parameters.
- **MSE-TCN:** a learned full-state-history baseline trained with MSE, without a
  physics-informed loss. It is not presented as a reproduction of PI-TCN.
- **Before refinement:** the supervised TransNet–RotNet pair before pose-level
  refinement, retained as a paired ablation.

All motion models share initial states in the joint rollout evaluation. The
full-state baseline additionally requires its observed pre-start state history.
No future reference states are injected after initialization. Recorded future
commands are supplied as exogenous controls in this offline prediction task;
this is distinct from causal online localization.
