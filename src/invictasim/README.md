# Invictasim

Invictasim is the Formula Student FEUP simulator package for autonomous system development. It
supports ROS-driven input and SDL keyboard input, publishes vehicle/track/visualization outputs,
and runs with configurable vehicle and track models.

## Current Package Structure

```text
src/invictasim/
├── include/
│   ├── config/
│   ├── io/
│   │   ├── input/
│   │   └── output/
│   ├── simulator/
│   ├── track/
│   └── vehicle_model/
├── launch/
│   └── invictasim.launch.py
├── resources/
│   ├── meshes/
│   └── tracks/
├── src/
│   ├── config/
│   ├── io/
│   │   ├── input/
│   │   └── output/
│   ├── simulator/
│   ├── track/
│   ├── vehicle_model/
│   └── main.cpp
├── test/
├── CMakeLists.txt
└── package.xml
```

## Runtime Overview

- `invictasim` executable starts simulator, input adapter, and output adapter.
- Input adapter options:
  - `ros`: subscribes to `/control/command`
  - `keyboard`: opens an SDL window (`W/S` throttle, `A/D` steering)
- Output adapter currently available:
  - `ros`: publishes simulator telemetry, input echo, track, TF, and visualization topics.

Adapter mapping is defined in:

- [include/io/input/map.hpp](include/io/input/map.hpp)
- [include/io/output/map.hpp](include/io/output/map.hpp)

## Configuration Files Used

Invictasim reads configuration through `common_lib::config_load`:

- [config/invictasim/global.yaml](../../config/invictasim/global.yaml)
  - `input_adapter`, `output_adapter`, `vehicle_model`, `sim_frequency`, `track_name`,
    `publish_frequencies`
- [config/global/global_config.yaml](../../config/global/global_config.yaml)
  - global discipline/system parameters used by the simulator context
- [config/invictasim/vehicle_models/02.yaml](../../config/invictasim/vehicle_models/02.yaml)
  (or `03.yaml`, selected by `vehicle_model` in `global.yaml`)
  - selected vehicle model: which sub-models to use and which parameter files to load
  - `car_parameters`: name of the car parameter file (e.g. `"02"`)
  - `car_parameters_dir`: directory, relative to `config/`, holding that file and the
    `*_model/` sub-parameter directories. Set to `"invictasim/car"`; defaults to `"car"` if omitted
- [config/invictasim/car/](../../config/invictasim/car/)
  - the simulator's own car parameters: `02.yaml`, `03.yaml` and `aero_model/`, `tire_model/`,
    `motor_model/`, etc.
  - this is where simulator tuning goes (`scripts/apply_tuned_parameters.py` refuses to write
    to `config/car`)

> **Note:** [config/car/](../../config/car/) holds the **real car's** parameters, read by the
> nodes that run on the car (`ekf_state_est`, `velocity_estimation`, `control`). The simulator
> does not read it. Changes to `config/invictasim/car/` do not affect the real car, and changes
> to `config/car/` do not affect the simulator.

Code reference for config loading:

- [src/config/config.cpp](src/config/config.cpp)

## Build

From workspace root:

```sh
source install/setup.bash
colcon build --packages-select invictasim
```

## Run

From workspace root:

```sh
source install/setup.bash
ros2 launch invictasim invictasim.launch.py
```

Or run the node directly:

```sh
source install/setup.bash
ros2 run invictasim invictasim
```

## Keyboard Mode

Set in [config/invictasim/global.yaml](../../config/invictasim/global.yaml):

```yaml
invictasim:
  input_adapter: "keyboard"
```

Controls and UI:

- `W`: increase throttle
- `S`: brake/reverse throttle
- `A`: steer left
- `D`: steer right
- `Q` or window close: exit keyboard input window
- SDL window shows labeled throttle and steering bars plus a focus hint

Keyboard implementation:

- [src/io/input/keyboard.cpp](src/io/input/keyboard.cpp)

## ROS Input Mode

Set in [config/invictasim/global.yaml](../../config/invictasim/global.yaml):

```yaml
invictasim:
  input_adapter: "ros"
```

Expected input topic:

- `/control/command` (`custom_interfaces/msg/ControlCommand`)

ROS input implementation:

- [src/io/input/ros.cpp](src/io/input/ros.cpp)

## Tests

Run tests for this package:

```sh
source install/setup.bash
colcon test --packages-select invictasim
colcon test-result --all --verbose
```