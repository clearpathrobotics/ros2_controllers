^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Changelog for package mecanum_drive_controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

4.42.2 (2026-09-21)
-------------------
* Formatting.
* fix(mecanum_drive_controller): reset rate-limiter history on NaN reference
  When update_and_write_commands takes the safety (else) branch because
  the reference is NaN, it zeroed the four wheel command interfaces but left
  previous_two_commands\_ populated with the last non-zero IK command. On the
  next tick with a real reference of 0.0 (e.g. operator taps the deadman
  with the stick centered), the limiter reads last = <stale non-zero> and
  slews toward 0 under the deceleration bound, producing a spurious wheel
  burst before decaying to zero (observed on hardware as brief motion when
  re-enabling the deadman with the stick centered).
  Reset previous_two_commands\_ to two zero entries in the else branch so
  limiter->limit() re-enters from rest.
  Regression test 'when_reference_goes_nan_then_zero_expect_no_wheel_burst_on_reenable'
  uses the with_limits config (max_acceleration=2.0, max_deceleration=-4.0)
  to build up ~0.5 m/s of limited command, drops to NaN, then resumes at
  zero and asserts every wheel is 0. Without this fix the test fails at
  ~1.12 rad/s on the rear wheels, matching the burst observed on hardware.
  (cherry picked from commit ef3d2732d038b6ee09b5976752bf0e2b9d4158ab)
* fix(mecanum_drive_controller): zero all four wheels on NaN reference
  In chainable mode, the previous update tick resets reference_interfaces\_
  to NaN, so the next tick takes the else branch of update_and_write_commands
  and is expected to write 0.0 to every wheel command interface. The old
  implementation chained the four set_value(0.0) calls with `&&`:
  const bool value_set_error =
  command_interfaces\_[FRONT_LEFT].set_value(0.0)  &&
  command_interfaces\_[FRONT_RIGHT].set_value(0.0) &&
  command_interfaces\_[REAR_RIGHT].set_value(0.0)  &&
  command_interfaces\_[REAR_LEFT].set_value(0.0);
  `&&` short-circuits on the first false, so if set_value on FRONT_LEFT
  ever returns false (its bounded CAS retry loop exhausts under contention
  or during a hardware-side stall), the remaining three wheels are never
  written and retain their last non-zero inverse-kinematics command.
  On a mecanum robot the operator sees three wheels keep spinning after
  the commanded twist is cleared.
  Replace the `&&`-chain with an explicit `&=` accumulator so each wheel
  is written independently and the log line still fires if any set_value
  failed. This matches the corresponding upstream fix in
  mecanum_drive_controller for the jazzy `||`+UINT_MAX variant, which had
  the same class of defect (short-circuit skipping wheel writes) with
  different short-circuit semantics.
  Add a regression test, when_reference_is_nan_in_chained_mode_expect\_
  all_wheels_zeroed, that:
  1. Activates the controller in chained mode.
  2. Tick 1: writes non-zero reference_interfaces\_ and asserts all four
  wheel command interfaces are non-zero (IK ran).
  3. Tick 2: relies on update_and_write_commands having reset the
  references to NaN and asserts every wheel is commanded to 0.0.
  Before the fix the test fails on tick 2 (three wheels retain their IK
  values); after the fix it passes.
  (cherry picked from commit 02f460c56c556e4f186ad75515e48eeed9f75278)
* test: cleanup controller fixture member variables (backport `#2562 <https://github.com/ros-controls/ros2_controllers/issues/2562>`_) (`#2565 <https://github.com/ros-controls/ros2_controllers/issues/2565>`_)
  Co-authored-by: Akshat Guduru <146907426+akki-g@users.noreply.github.com>
  Co-authored-by: Bence Magyar <bence.magyar.robotics@gmail.com>
* Contributors: Mark Ibrahim, Tony Baltovski, mergify[bot]

4.42.1 (2026-08-12)
-------------------

4.42.0 (2026-08-10)
-------------------
* Throttle speed limiter parameter error logs (backport `#2546 <https://github.com/ros-controls/ros2_controllers/issues/2546>`_) (`#2547 <https://github.com/ros-controls/ros2_controllers/issues/2547>`_)
* Use new Command/State Interfaces API for tests (backport `#2476 <https://github.com/ros-controls/ros2_controllers/issues/2476>`_) (`#2532 <https://github.com/ros-controls/ros2_controllers/issues/2532>`_)
* fix: Remove unused variable assignments in mecanum tests (backport `#2534 <https://github.com/ros-controls/ros2_controllers/issues/2534>`_) (`#2537 <https://github.com/ros-controls/ros2_controllers/issues/2537>`_)
* Use new chainable controller exports API (backport `#2350 <https://github.com/ros-controls/ros2_controllers/issues/2350>`_) (`#2453 <https://github.com/ros-controls/ros2_controllers/issues/2453>`_)
* Fix safety concerns with halt logic across controllers (backport `#2326 <https://github.com/ros-controls/ros2_controllers/issues/2326>`_) (`#2458 <https://github.com/ros-controls/ros2_controllers/issues/2458>`_)
* Contributors: mergify[bot]

4.41.0 (2026-07-01)
-------------------
* Test fix - call appropriate lifecycle transitions in controller tests: forward_command, mecanum_drive, range_sensor, imu_sensor (backport `#2406 <https://github.com/ros-controls/ros2_controllers/issues/2406>`_) (`#2407 <https://github.com/ros-controls/ros2_controllers/issues/2407>`_)
* Added velocity limiting to the mecanum controller. (backport `#2313 <https://github.com/ros-controls/ros2_controllers/issues/2313>`_) (`#2362 <https://github.com/ros-controls/ros2_controllers/issues/2362>`_)
* Contributors: mergify[bot]

4.40.1 (2026-05-12)
-------------------

4.40.0 (2026-04-22)
-------------------

4.39.0 (2026-04-04)
-------------------

4.38.0 (2026-03-12)
-------------------
* Consistently add <cmath> include with define for windows (backport `#2193 <https://github.com/ros-controls/ros2_controllers/issues/2193>`_) (`#2195 <https://github.com/ros-controls/ros2_controllers/issues/2195>`_)
* Contributors: mergify[bot]

4.37.0 (2026-02-03)
-------------------

4.36.0 (2025-12-31)
-------------------
* Controller interface api update to ros2_controller packages (backport `#1973 <https://github.com/ros-controls/ros2_controllers/issues/1973>`_) (`#2068 <https://github.com/ros-controls/ros2_controllers/issues/2068>`_)
* Contributors: mergify[bot]

4.35.0 (2025-12-01)
-------------------

4.34.0 (2025-11-10)
-------------------

4.33.1 (2025-10-17)
-------------------

4.33.0 (2025-10-03)
-------------------
* Update API for realtime publisher (backport `#1830 <https://github.com/ros-controls/ros2_controllers/issues/1830>`_) (`#1944 <https://github.com/ros-controls/ros2_controllers/issues/1944>`_)
* Update realtime containers (backport `#1721 <https://github.com/ros-controls/ros2_controllers/issues/1721>`_) (`#1935 <https://github.com/ros-controls/ros2_controllers/issues/1935>`_)
* mecanum_drive_controller: Declare missing backward_ros dependency (backport `#1941 <https://github.com/ros-controls/ros2_controllers/issues/1941>`_) (`#1943 <https://github.com/ros-controls/ros2_controllers/issues/1943>`_)
* Use new handles API in ros2_controllers to fix deprecation warnings (backport `#1566 <https://github.com/ros-controls/ros2_controllers/issues/1566>`_) (`#1934 <https://github.com/ros-controls/ros2_controllers/issues/1934>`_)
* Contributors: mergify[bot]

4.32.0 (2025-09-12)
-------------------

4.31.0 (2025-08-27)
-------------------

4.30.1 (2025-08-03)
-------------------

4.30.0 (2025-07-31)
-------------------

4.29.0 (2025-07-23)
-------------------

4.28.0 (2025-07-14)
-------------------
* Mecanum Drive: Populate the pose covariance matrix (backport `#1772 <https://github.com/ros-controls/ros2_controllers/issues/1772>`_) (`#1807 <https://github.com/ros-controls/ros2_controllers/issues/1807>`_)
* Add tf_frame_prefix parameters to mecanum_drive_controller (backport `#1680 <https://github.com/ros-controls/ros2_controllers/issues/1680>`_) (`#1810 <https://github.com/ros-controls/ros2_controllers/issues/1810>`_)
* Contributors: Hilary Luo, Dawid Kmak

4.27.1 (2025-07-02)
-------------------

4.27.0 (2025-06-23)
-------------------

4.26.0 (2025-06-06)
-------------------
* Add missing github_url to rst files (backport `#1717 <https://github.com/ros-controls/ros2_controllers/issues/1717>`_) (`#1719 <https://github.com/ros-controls/ros2_controllers/issues/1719>`_)
* Use target_link_libraries instead of ament_target_dependencies (backport `#1697 <https://github.com/ros-controls/ros2_controllers/issues/1697>`_) (`#1699 <https://github.com/ros-controls/ros2_controllers/issues/1699>`_)
* Contributors: mergify[bot]

4.25.0 (2025-05-17)
-------------------
* Simplify `on_set_chained_mode` avoiding cpplint warnings (backport `#1564 <https://github.com/ros-controls/ros2_controllers/issues/1564>`_) (`#1688 <https://github.com/ros-controls/ros2_controllers/issues/1688>`_)
* Deprecating tf2 C Headers (`#1325 <https://github.com/ros-controls/ros2_controllers/issues/1325>`_)
* Contributors: Lucas Wendland, mergify[bot], Bhagyesh Agresar

4.24.0 (2025-04-27)
-------------------
* Fix preceeding->preceding typos (`#1655 <https://github.com/ros-controls/ros2_controllers/issues/1655>`_)
* Contributors: Christoph Fröhlich

4.23.0 (2025-04-10)
-------------------
* Bump version of pre-commit hooks (`#1618 <https://github.com/ros-controls/ros2_controllers/issues/1618>`_)
* Use global cmake macros and fix gcc-10 build (`#1527 <https://github.com/ros-controls/ros2_controllers/issues/1527>`_)
* Contributors: Christoph Fröhlich, github-actions[bot]

4.22.0 (2025-03-17)
-------------------
* [mecanum_drive_controller] Fix Odometry Initialization  (`#1573 <https://github.com/ros-controls/ros2_controllers/issues/1573>`_)
* Contributors: Julia Jia

4.21.0 (2025-03-01)
-------------------
* Fix mecanum_drive_controller documentation (`#1547 <https://github.com/ros-controls/ros2_controllers/issues/1547>`_)
* Fix the exported interface naming in the chainable controllers (`#1528 <https://github.com/ros-controls/ros2_controllers/issues/1528>`_)
* Contributors: Christoph Fröhlich, Sai Kishor Kothakota

4.20.0 (2025-01-29)
-------------------
* Update paths of GPL includes (`#1487 <https://github.com/ros-controls/ros2_controllers/issues/1487>`_)
* Contributors: Christoph Fröhlich

4.19.0 (2025-01-13)
-------------------
* Remove visibility macros (`#1451 <https://github.com/ros-controls/ros2_controllers/issues/1451>`_)
* Clean up unused variable EPS in mecanum_drive_controller (`#1444 <https://github.com/ros-controls/ros2_controllers/issues/1444>`_)
* Contributors: Bence Magyar, Shankar-Balajee

4.18.0 (2024-12-19)
-------------------

4.17.0 (2024-12-07)
-------------------
* Use the .hpp headers from `realtime_tools` package (`#1406 <https://github.com/ros-controls/ros2_controllers/issues/1406>`_)
* Add Mecanum Drive Controller (`#512 <https://github.com/ros-controls/ros2_controllers/issues/512>`_)
* Contributors: Dr. Denis, Sai Kishor Kothakota

4.16.0 (2024-11-08)
-------------------

4.15.0 (2024-10-07)
-------------------

4.14.0 (2024-09-11)
-------------------

4.13.0 (2024-08-22)
-------------------

4.12.1 (2024-08-14)
-------------------

4.12.0 (2024-07-23)
-------------------

4.11.0 (2024-07-09)
-------------------

4.10.0 (2024-07-01)
-------------------

4.9.0 (2024-06-05)
------------------

4.8.0 (2024-05-14)
------------------

4.7.0 (2024-03-22)
------------------

4.6.0 (2024-02-12)
------------------

4.5.0 (2024-01-31)
------------------

4.4.0 (2024-01-11)
------------------

4.3.0 (2024-01-08)
------------------

4.2.0 (2023-12-12)
------------------

4.1.0 (2023-12-01)
------------------

4.0.0 (2023-11-21)
------------------

3.17.0 (2023-10-31)
-------------------

3.16.0 (2023-09-20)
-------------------

3.15.0 (2023-09-11)
-------------------

3.14.0 (2023-08-16)
-------------------

3.13.0 (2023-08-04)
-------------------

3.12.0 (2023-07-18)
-------------------

3.11.0 (2023-06-24)
-------------------

3.10.1 (2023-06-06)
-------------------

3.10.0 (2023-06-04)
-------------------

3.9.0 (2023-05-28)
------------------

3.8.0 (2023-05-14)
------------------

3.7.0 (2023-05-02)
------------------

3.6.0 (2023-04-29)
------------------

3.5.0 (2023-04-14)
------------------

3.4.0 (2023-04-02)
------------------

3.3.0 (2023-03-07)
------------------

3.2.0 (2023-02-10)
------------------

3.1.0 (2023-01-26)
------------------

3.0.0 (2023-01-19)
------------------

2.15.0 (2022-12-06)
-------------------

2.14.0 (2022-11-18)
-------------------

2.13.0 (2022-10-05)
-------------------

2.12.0 (2022-09-01)
-------------------

2.11.0 (2022-08-04)
-------------------

2.10.0 (2022-08-01)
-------------------

2.9.0 (2022-07-14)
------------------

2.8.0 (2022-07-09)
------------------

2.7.0 (2022-07-03)
------------------

2.6.0 (2022-06-18)
------------------

2.5.0 (2022-05-13)
------------------

2.4.0 (2022-04-29)
------------------

2.3.0 (2022-04-21)
------------------

2.2.0 (2022-03-25)
------------------

2.1.0 (2022-02-23)
------------------

2.0.1 (2022-02-01)
------------------

2.0.0 (2022-01-28)
------------------

1.3.0 (2022-01-11)
------------------

1.2.0 (2021-12-29)
------------------

1.1.0 (2021-10-25)
------------------

1.0.0 (2021-09-29)
------------------

0.5.0 (2021-08-30)
------------------

0.4.1 (2021-07-08)
------------------

0.4.0 (2021-06-28)
------------------

0.3.1 (2021-05-23)
------------------

0.3.0 (2021-05-21)
------------------

0.2.1 (2021-05-03)
------------------

0.2.0 (2021-02-06)
------------------

0.1.2 (2021-01-07)
------------------

0.1.1 (2021-01-06)
------------------

0.1.0 (2020-12-23)
------------------
