# Prox-Ops BT Goal Example

`goal_example.json` is the WARA-compatible JSON payload that goes inside
`smarc_msgs/action/BaseAction.goal.data` when sending a goal to
`/prox_ops_bt`. The action server also accepts the unwrapped nested prox-ops
mission object directly for developer tests.

Example command:

```bash
GOAL_JSON="$(jq -c . install/share/prox_ops_bt/goal_example.json)"
ros2 action send_goal /evolo/prox_ops_bt smarc_msgs/action/BaseAction \
"{goal: {data: '${GOAL_JSON}'}}"
```

## Accepted Goal Formats

Recommended WARA-compatible wrapper:

```json
{
  "json-params": "{\"inspect\":{\"timeout_s\":5.0},\"intercept\":{\"geofence_check_enabled\":false},\"loiter_patrol\":{\"timeout_s\":30.0,\"speed\":\"standard\",\"waypoints\":[{\"latitude\":58.822943,\"longitude\":17.634076},{\"latitude\":58.821758,\"longitude\":17.634371}]}}"
}
```

Developer-friendly direct payload, also accepted:

```json
{
  "inspect": {
    "timeout_s": 5.0
  },
  "intercept": {
    "geofence_check_enabled": false
  },
  "loiter_patrol": {
    "timeout_s": 30.0,
    "speed": "standard",
    "waypoints": [
      {"latitude": 58.822943, "longitude": 17.634076},
      {"latitude": 58.821758, "longitude": 17.634371}
    ]
  }
}
```

To wait in place instead of running loiter/patrol, replace `loiter_patrol` with
`wait_for_convergence`:

```json
{
  "inspect": {
    "timeout_s": 5.0
  },
  "intercept": {
    "geofence_check_enabled": false
  },
  "wait_for_convergence": {
    "timeout_s": 30.0
  }
}
```

When `json-params` is present, `prox_ops_bt` parses it first and validates the
inner object.

## Inner Sections

`inspect`
: Configuration for the inspection branch. The BT forwards this object to
`evolo_target_inspect` and also uses its timeout to decide when the overall
prox-ops mission has succeeded.

`intercept`
: Configuration for the intercept branch. The BT forwards this object to
`evolo_target_intercept`.

Exactly one of `loiter_patrol` or `wait_for_convergence` must be present.

`loiter_patrol`
: Configuration for the moving await-convergence branch. The BT forwards this
object to `evolo_loiter_patrol` and also uses its timeout to decide when to
give up the prox-ops mission.

`wait_for_convergence`
: Configuration for the stationary await-convergence branch. The BT converts
this object to the `smarc_wait` action goal format.

## Fields

`inspect.timeout_s`
: Positive number of seconds the target must remain in the inspection branch
before `prox_ops_bt` reports overall success. This is owned by the BT.

`intercept.geofence_check_enabled`
: Boolean per-goal override for `evolo_target_intercept` geofence validation.
`false` is useful for bench tests with the fake backend. `true` requires fresh
`smarc/geofence_status` and `smarc/geofence_polygons`.

`loiter_patrol.timeout_s`
: Positive number of seconds the BT may spend in loiter/patrol before failing
the overall prox-ops goal and sending backend `RESET` then `STOP`.

`wait_for_convergence.timeout_s`
: Positive number of seconds the BT may spend waiting in place for backend
convergence before failing the overall prox-ops goal and sending backend
`RESET` then `STOP`. The BT sends this to `smarc_wait` as `timeout`.

`loiter_patrol.speed`
: Speed passed through to `evolo_loiter_patrol`, then delegated to the existing
`move_to` action. Current expected values include `slow`, `standard`, `fast`,
or a numeric speed accepted by `evolo_move_to`.

`loiter_patrol.waypoints`
: Ordered list of patrol waypoints. The current loiter/patrol action uses the
first two waypoints and alternates between them.

`loiter_patrol.waypoints[].latitude`
: Waypoint latitude in decimal degrees.

`loiter_patrol.waypoints[].longitude`
: Waypoint longitude in decimal degrees.
