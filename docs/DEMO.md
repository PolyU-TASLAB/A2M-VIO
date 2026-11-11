# Demonstration details

| Flight | Trajectory | Dataset lighting metadata | Video |
|:---|:---|:---|:---|
| 02a | Ellipse | High / natural, mean 1500 lux | [MP4](../assets/homepage_demo_02a.mp4) |
| 18a | Race track | Low / artificial, mean 72 lux | [MP4](../assets/homepage_demo_18a.mp4) |

These Test examples were chosen after inspecting results to show favorable
A2M-VIO outcomes under different conditions. They are qualitative highlights,
not a representative-sample accuracy estimate. All nine evaluation flights and
all localization methods remain in the [per-flight table](../results/localization_per_flight.csv),
including cases where a baseline outperforms A2M-VIO.

- Upper left: Kimera's native feature points and tracks from VIO+Full.
- Upper right: four estimated trajectories and the motion-capture reference.
- Bottom: start-aligned Euclidean position-error curves.
- Ours is red. Each GIF holds its completed result for four additional seconds.

These are synchronized post-run visualizations of saved estimates, not four
algorithms running simultaneously live or a real-time throughput measurement.

## Alignment

First, a fixed SE(3) alignment without scale correction is applied to each saved
trajectory over the common evaluation interval. For display, each trajectory is
then translated by its own first position so all starts coincide. Those fixed
display translations also apply to the error curves.

Quantitative paper ATE is computed before the display translation, using official
evo `-a`. It must not be confused with the RMSE of the displayed start-aligned
error curves.

| Method | 02a ATE RMSE [m] | 18a ATE RMSE [m] |
|:---|---:|---:|
| Kimera-VIO | 0.7770 | 3.1427 |
| AI-IO | 1.1287 | 2.1397 |
| TartanIMU+LoRA | 1.4254 | 2.1833 |
| VIO+Full (ours) | 0.4884 | 1.2261 |

Motion capture is used only for offline alignment, evaluation, and display. It is
not fed to the online networks or estimator. No new training or tuning was
performed to produce these animations. They show the frozen paper configuration,
not the separate raw-command replay adapter evaluated later.
