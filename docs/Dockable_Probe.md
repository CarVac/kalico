# Dockable Probe

Dockable probes are typically microswitches mounted to a printed body that
attaches to the toolhead through some means of mechanical coupling.
This coupling is commonly done with magnets though there is support for
a variety of designs including servo and stepper actuated couplings.

## Basic Configuration

To use a dockable probe the following options are required at a minimum.
Some users may be transitioning from a macro based set of commands and
many of the options for the `[probe]` config section are the same.
The `[dockable_probe]` module is first and foremost a `[probe]`
but with additional functionality. Most of the options that can be specified
for `[probe]` are valid for `[dockable_probe]`.

```
[dockable_probe]
pin:
z_offset:
sample_retract_dist:
approach_position:
dock_position:
detach_position:
(check_open_attach: OR probe_sense_pin:) AND/OR dock_sense_pin:
```

### Attaching and Detaching Positions

- `dock_position: 300, 295, 0`\
  _Required_\
  This is the XYZ coordinates where the toolhead needs to be positioned
  in order to attach the probe. This parameter is X, Y and, Z separated
  by commas.

  Many configurations have the dock attached to a moving gantry. This
  means that Z axis positioning is irrelevant. However, it may be necessary
  to move the gantry clear of the bed or other printer components before
  performing docking steps. In this case, specify `z_hop` to force a Z movement.

  Other configurations may have the dock mounted next to the printer bed so
  that the Z position _must_ be known prior to attaching the probe. In this
  configuration the Z axis parameter _must_ be supplied, and the Z axis
  _must_ be homed prior to attaching the probe.

- `approach_position: 300, 250, 0`\
  _Required_\
  The most common dock designs use a fork or arms that extend out from the dock.
  In order to attach the probe to the toolhead, the toolhead must move into and
  away from the dock to a particular position so these arms can capture the
  probe body.

  As with `dock_position`, a Z position is not required but if specified the
  toolhead will be moved to that Z location before the X, Y coordinates.

  For magnetically coupled probes, the `approach_position` should be far enough
  away from the probe dock such that the magnets on the probe body are not
  attracted to the magnets on the toolhead.

- `detach_position: 250, 295, 0`\
  _Required_\
  Most probes with magnets require the toolhead to move in a direction that
  strips the magnets off with a sliding motion. This is to prevent the magnets
  from becoming unseated from repeated pulling and thus affecting probe accuracy.
  The `detach_position` is typically defined as a point that is perpendicular to
  the dock so that when the toolhead moves, the probe stays docked but cleanly
  detaches from the toolhead mount.

  As with `dock_position`, a Z position is not required but if specified the
  toolhead will be moved to that Z location before the X, Y coordinates.

  For magnetically coupled probes, the `detach_position` should be far enough
  away from the probe dock such that the magnets on the probe body are not
  attracted to the magnets on the toolhead.

- `extract_position: 295, 250, 0`\
  _Default Value: approach\_position_\
  Euclid probe requires the toolhead to move to a different direction to extract
  or dock mag_probe.

- `insert_position: 295, 250, 0`\
  _Default Value: extract\_position_\
  Usually the same as extract position for Euclid probe when the dock is on the
  gantry.

- `z_hop: 15.0`\
  _Default Value: None_\
  Distance (in mm) to lift the Z axis prior to attaching/detaching the probe.
  If the Z axis is already homed and the current Z position is less
  than `z_hop`, then this will lift the head to a height of `z_hop`. If
  the Z axis is not already homed the head is lifted by `z_hop`.
  The default is to not implement Z hop.

- `restore_toolhead: False|True`\
  _Default Value: True_\
  The position of the toolhead is restored to the position prior to 
  the attach/detach movements. See table below.

| Command                 | Module            | restore_th=True | restore_th=False | notes                          |
| ----------------------- | ----------------- | --------------- | ---------------- | ------------------------------ |
| ATTACH_PROBE            | dockable_probe.py | True            | False            |                                |
| Z_TILT_ADJUST           | probe.py          | True            | False            |                                |
| QUAD_GANTRY_LEVEL       | probe.py          | True            | False            |                                |
| PROBE                   | probe.py          | True            | True             |                                |
| PROBE_ACCURACY          | probe.py          | True            | True             |                                |
| AXIS_TWIST_COMPENSATION | probe.py          | True            | True             |                                |
| CALIBRATE_Z             | z_calibration.py  | True            | False            |                                |
| G28 Z                   | probe.py          | True            | True             | **only for z_virtual_endstop** |


## Position Examples

Probe mounted on frame at back of print bed at a fixed Z position. To attach
the probe, the toolhead will move back and then forward. To detach, the toolhead
will move back, and then to the side.

```
+--------+
|   p>   |
|   ^    |
|        |
+--------+
```

```
approach_position: 150, 300, 5
dock_position: 150, 330, 5
detach_position: 170, 330
```


Probe mounted at side of moving gantry with fixed bed. Here the probe is
attachable regardless of the Z position. To attach the probe, the toolhead will
move to the side and back. To detach the toolhead will move to the side and then
forward.

```
+--------+
|        |
| p<     |
| v      |
+--------+
```

```
approach_position: 50, 150
dock_position: 10, 150
detach_position: 10, 130
```


Probe mounted at side of fixed gantry with bed moving on Z. Probe is attachable
regardless of Z but force Z hop for safety. The toolhead movement is the same
as above.

```
+--------+
|        |
| p<     |
| v      |
+--------+
```

```
approach_position: 50, 150
dock_position: 10, 150
detach_position: 10, 130
z_hop: 15
```


Euclid style probe that requires the attach and detach movements to happen in
opposite order. Attach: approach, move to dock, extract. Detach: move to
extract position, move to dock, move to approach position. The approach and
detach positions are the same, as are the extract and insert positions.

```
Attach:
+--------+
|        |
| p<     |
| v      |
+--------+
Detach:
+--------+
|        |
| p>     |
| ^      |
+--------+
```

```
approach_position: 50, 150
dock_position: 10, 150
extract_position: 10, 130
detach_position: 50, 150
z_hop: 15
```

### Additional G-codes

If your probe has special setup/teardown steps (e.g., moving a servo), you can use the following configuration options to execute custom G-code before or after attaching/detaching the probe instead of overwriting [individual movements](#individual-movements) commands:

- `pre_attach_gcode:`\
  _Default Value: None_\
  G-code to execute immediately before the probe is attached.

- `post_attach_gcode:`\
  _Default Value: None_\
  G-code to execute immediately after the probe is attached.

- `pre_detach_gcode:`\
  _Default Value: None_\
  G-code to execute immediately before the probe is detached.

- `post_detach_gcode:`\
  _Default Value: None_\
  G-code to execute immediately after the probe is detached.

### Homing

No configuration specific to the dockable probe is required. However, when
using the probe as a virtual endstop, it's required to
use `[safe_z_home]` or `[homing_override]`.

#### Examples for probe as virtual endstop
- #### Homing override
```elixir
[homing_override]
axes: xyz
set_position_z: 0
gcode:
  
  G90
  {% set home_all = 'X' not in params and 'Y' not in params and 'Z' not in params %}

  {% if home_all or 'X' in params %}
    G0 Z10
    G28 X
  {% endif %}

  {% if home_all or 'Y' in params %}
    G0 Z10
    G28 Y
  {% endif %}
  
  {% if home_all or 'Z' in params %}
    ATTACH_PROBE
    MOVE_AVOIDING_DOCK X=150 Y=150 SPEED=300
    # Probe is already attached, no need to make a return trip to the dock.
    G28 Z  
  {% endif %}
```

- #### safe_z_home
While homing Z with safe_z_home, the toolhead will move to home_xy_position then will 
move  to the dock and return back to home_xy_position.
```elixir
[safe_z_home]
home_xy_position: 150,150
z_hop: 10
```


### Probe Attachment Verification

Given the nature of this type of probe, it is necessary to verify whether or
not it has successfully attached prior to attempting a probing move. Several
methods can be used to verify probe attachment states.

- `check_open_attach:`\
  _Default Value: None_\
  Certain probes will report `OPEN` when they are attached and `TRIGGERED`
  when they are detached in a non-probing state. When `check_open_attach` is
  set to `True`, the state of the probe pin is checked after performing a
  probe attach or detach maneuver. If the probe does not read `OPEN`
  immediately after attaching the probe, an error will be raised and any
  further action will be aborted.

  This is intended to prevent crashing the nozzle into the bed since it is
  assumed if the probe pin reads `TRIGGERED` prior to probing, the probe is
  not attached.

  Setting this to `False` will cause all action to be aborted if the probe
  does not read `TRIGGERED` after attaching.

- `probe_sense_pin:`\
  _Default Value: None_\
  The probe may include a separate pin for attachment verification. This is a
  standard pin definition, similar to an endstop pin that defines how to handle
  the input from the sensor. Much like the `check_open_attach` option, the check
  is done immediately after the tool attaches or detaches the probe. If the
  probe is not detected after attempting to attach it, or it remains attached
  after attempting to detach it, an error will be raised and further
  action aborted.

- `dock_sense_pin:`\
  _Default Value: None_\
  Docks can have a sensor or switch incorporated into their design in
  order to report that the probe is presently located in the dock. A
  `dock_sense_pin` can be used to provide verification that the probe is
  correctly positioned in the dock. This is a standard pin definition similar
  to an endstop pin that defines how to handle the input from the sensor.
  Prior to attempting to attach the probe, and after attempting to detach it,
  this pin is checked. If the probe is not detected in the dock, an error will
  be raised and further action aborted.

- `dock_retries: 5`\
  _Default Value: 0_\
  A magnetic probe may require repeated attempts to attach or detach. If
  `dock_retries` is specified and the probe fails to attach or detach, the
  attach/detach action will be repeated until it succeeds. If the retry limit
  is reached and the probe is still not in the correct state, an error will be
  raised and further action aborted.

## Tool Velocities

- `attach_speed: 5.0`\
  _Default Value: Probe `speed` or 5_\
  Movement speed when attaching the probe during `MOVE_TO_DOCK_PROBE`.

- `detach_speed: 5.0`\
  _Default Value: Probe `speed` or 5_\
  Movement speed when detaching the probe during `MOVE_TO_DETACH_PROBE`.

- `travel_speed: 5.0`\
  _Default Value: Probe `speed` or 5_\
  Movement speed when approaching the probe during `MOVE_TO_APPROACH_PROBE`
  and returning the toolhead to its previous position after attach/detach.

## Safe dock area

A safe dock area is defined to avoid collision with the dock during probe attachment/detachment moves. see `MOVE_AVOIDING_DOCK`.

- `safe_dock_distance:`\
  _Default Value: minimum distance to the dock of approach\_position or insert\_position_ \
  This indroduce on the first version of the plugin. It defines a security area
  centered on dock position during ATTACH/DETACH_PROBE operations.
  Approach, insert and detach position shoulb outside this area.  

- `safe_position: 250, 295, 0`
  _Default Value: approach_position_
  A safe position to ensure MOVE_AVOIDING_DOCK travel does not move out of range

### MOVE_AVOIDING_DOCK description
![safe dock area](./img/move_avoiding_dock.jpg)
The strategy described below is used by the attach and detach commands to avoid dock collision.

> [!NOTE]  
> The default `safe_position` is `approach_position`. To help determine the avoiding path and prevent moving out of range, it should be configured as a point next to the safe area, farthest from the "move out of range" zone. 

Several cases are illustrated:
1. Moving from `A` to `B`: The requested trajectory passes over the safe dock area, so the calculated trajectory passes around the dock area, close to the safe position.
2. Moving from `A'` to `B`: The toolhead leaves the safe dock area by the shortest path and reaches `B` as before.
3. Moving from `A` to `B'`: Since `B'` is in the safe area, the toolhead stops at `B"`.
4. Moving from `A'` to `B'`: The toolhead leaves the safe dock area by the shortest path.

## Dockable Probe Gcodes

### General

`ATTACH_PROBE`

This command will move the toolhead to the dock, attach the probe, and return
it to its previous position. If the probe is already attached, the command
does nothing.

This command will call `MOVE_TO_APPROACH_PROBE`, `MOVE_TO_DOCK_PROBE`,
and `MOVE_TO_EXTRACT_PROBE`.

`DETACH_PROBE`

This command will move the toolhead to the dock, detach the probe, and return
it to its previous position. If the probe is already detached, the command
will do nothing.

This command will call `MOVE_TO_APPROACH_PROBE`, `MOVE_TO_DOCK_PROBE`,
and `MOVE_TO_DETACH_PROBE`.

### Individual Movements

These commands are useful during setup to prevent the full attach/detach
sequence from crashing into the bed or damaging the probe/dock.

If your probe has special setup/teardown steps (e.g. moving a servo),
accommodating that could be accomplished by using [additional G_codes](#additional-g-codes) in configuration or overriding the following gcodes.

`MOVE_TO_APPROACH_PROBE`

This command will move the toolhead to the `approach_position`. It can be
overridden to move a servo if that's required for attaching your probe.

`MOVE_TO_DOCK_PROBE`

This command will move the toolhead to the `dock_position`.

`MOVE_TO_EXTRACT_PROBE`

This command will move the toolhead to the `extract_position`.

`MOVE_TO_INSERT_PROBE`

This command will move the toolhead to the `insert_position`.

`MOVE_TO_DETACH_PROBE`

This command will move the toolhead to the `detach_position`. It can be
overridden to move a servo if that's required for detaching your probe.

`MOVE_AVOIDING_DOCK [X=<value>] [Y=<value>] [SPEED=<value>]`

This command will move the toolhead to the absolute coordinates, avoiding the 
safe dock area.

### Status

`QUERY_DOCKABLE_PROBE`

Responds in the gcode terminal with the current probe status. Valid
states are UNKNOWN, ATTACHED, and DOCKED. This is useful during setup
to confirm probe configuration is working as intended.

`SET_DOCKABLE_PROBE AUTO_ATTACH_DETACH=0|1`

Enable/Disable the automatic attaching/detaching of the probe during
actions that require the probe.

This command can be helpful in print-start macros where multiple actions will
be performed with the probe and there's no need to detach the probe.
For example:

```
SET_DOCKABLE_PROBE AUTO_ATTACH_DETACH=0
G28
ATTACH_PROBE                             # Explicitly attach the probe
QUAD_GANTRY_LEVEL                        # Tram the gantry parallel to the bed
BED_MESH_CALIBRATE                       # Create a bed mesh
DETACH_PROBE                             # Manually detach the probe
SET_DOCKABLE_PROBE AUTO_ATTACH_DETACH=1  # Make sure the probe is attached in future
```

## Typical probe execution flow

### Probing is Started:

    - A gcode command requiring the use of the probe is executed.

    - This triggers the probe to attach.

    - If configured, the dock sense pin is checked to see if the probe is
      presently in the dock.

    - The toolhead position is compared to the dock position.

    - The pre_attach_gcode is executed

    - If the toolhead is outside of the minimum safe radius, the toolhead is
      commanded to move to the approach vector, that is, a position that is
      the minimum safe distance from the dock in line with the dock angle.
      (MOVE_TO_APPROACH_PROBE)

    - If the toolhead is inside of the minimum safe radius, the toolhead is
      commanded to move to the nearest point on the line of the approach vector.
      (MOVE_TO_APPROACH_PROBE)

    - The tool is moved along the approach vector to the dock coordinates.
      (MOVE_TO_DOCK_PROBE)

    - The toolhead is commanded to move out of the dock back to the minimum
      safe distance in the reverse direction along the dock angle.
      (MOVE_TO_EXTRACT_PROBE)

    - The post_attach_gcode is executed

    - If configured, the probe is checked to see if it is attached.

    - If the probe is not attached, the module may retry until it's attached or
      an error is raised.

    - If configured, the dock sense pin is checked to see if the probe is still
      present, the module may retry until the probe is absent not or an error
      is raised.

    - The probe moves to the first probing point and begins probing.

### Probing is Finished:

    - After the probe is no longer needed, the probe is triggered to detach.

    - The toolhead position is compared to the dock position.

    - The pre_detach_gcode is executed

    - If the toolhead is outside of the minimum safe radius, the toolhead is
      commanded to move to the approach vector, that is, a position that is
      the minimum safe distance from the dock in line with the dock angle.
      (MOVE_TO_APPROACH_PROBE)

    - If the toolhead is inside of the minimum safe radius, the toolhead is
      commanded to move to the nearest point on the line of the approach vector.
      (MOVE_TO_APPROACH_PROBE)

    - The toolhead is moved along the approach vector to the dock coordinates.
    (MOVE_TO_DOCK_PROBE)

    - The toolhead is commanded to move along the detach vector if supplied or a
      calculated direction based on axis parameters. (MOVE_TO_DETACH_PROBE)

    - The post_detach_gcode is executed

    - If configured, the probe is checked to see if it detached.

    - If the probe did not detach, the module moves the toolhead back to the
      approach vector and may retry until it detaches or an error is raised.

    - If configured, the dock sense pin is checked to see if the probe is
      present in the dock. If it is not the module moves the toolhead back to
      the approach vector and may retry until it detaches or an error is raised.
