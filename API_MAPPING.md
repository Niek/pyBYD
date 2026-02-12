# BYD API Field Mapping Reference

This file documents the API fields that pyBYD currently parses and exposes.

How to update this file:

1. Use `scripts/dump_all.py` to capture vehicle state in different scenarios (parked, driving, charging, A/C on, doors open).
2. Compare the JSON outputs to identify which fields change and what the values mean.
3. Update the tables below and (if needed) extend the parsers/models.

Last updated: 2026-02-12

Base URL: https://dilinkappoversea-eu.byd.auto

---

## Endpoints used by this library

Only endpoints that pyBYD interfaces with are listed here.

| Endpoint (path) | Purpose | Implementation |
|---|---|---|
| `/app/account/login` | Authentication | `src/pybyd/_api/login.py` |
| `/app/account/getAllListByUserId` | Vehicle list | `src/pybyd/_api/vehicles.py` |
| `/vehicleInfo/vehicle/vehicleRealTimeRequest` | Realtime trigger | `src/pybyd/_api/realtime.py` |
| `/vehicleInfo/vehicle/vehicleRealTimeResult` | Realtime poll | `src/pybyd/_api/realtime.py` |
| `/control/getStatusNow` | HVAC status | `src/pybyd/_api/hvac.py` |
| `/control/getGpsInfo` | GPS trigger | `src/pybyd/_api/gps.py` |
| `/control/getGpsInfoResult` | GPS poll | `src/pybyd/_api/gps.py` |
| `/control/smartCharge/homePage` | Charging status | `src/pybyd/_api/charging.py` |
| `/vehicleInfo/vehicle/getEnergyConsumption` | Energy consumption | `src/pybyd/_api/energy.py` |
| `/control/remoteControl` | Remote control trigger | `src/pybyd/_api/control.py` |
| `/control/remoteControlResult` | Remote control poll | `src/pybyd/_api/control.py` |

---

## Status labels used below

- confirmed: verified by live captures
- unconfirmed: plausible but not verified yet
- conflicting: observed data contradicts the assumed meaning

---

## Realtime data

URL (trigger): https://dilinkappoversea-eu.byd.auto/vehicleInfo/vehicle/vehicleRealTimeRequest

URL (poll): https://dilinkappoversea-eu.byd.auto/vehicleInfo/vehicle/vehicleRealTimeResult

Model: `VehicleRealtimeData`

Parser: `src/pybyd/_api/realtime.py`

### Connection and state

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `onlineState` | `online_state` | `OnlineState` | 0=unknown (unconfirmed), 1=online (confirmed), 2=offline (unconfirmed) |
| `connectState` | `connect_state` | `ConnectState` | -1=unknown (conflicting: seen while driving and online), 0=disconnected (unconfirmed), 1=connected (unconfirmed) |
| `vehicleState` | `vehicle_state` | `VehicleState` | 0=standby (conflicting: seen while driving at 22 km/h), 1=active (unconfirmed) |
| `requestSerial` | `request_serial` | `str | None` | Poll serial token |

### Battery and range

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `elecPercent` | `elec_percent` | `float | None` | 0-100 percent (confirmed) |
| `powerBattery` | `power_battery` | `float | None` | Alternative battery percent (unconfirmed) |
| `enduranceMileage` | `endurance_mileage` | `float | None` | Estimated range in km (confirmed) |
| `evEndurance` | `ev_endurance` | `float | None` | Alternative EV range (unconfirmed) |
| `enduranceMileageV2` | `endurance_mileage_v2` | `float | None` | Range v2 (unconfirmed) |
| `enduranceMileageV2Unit` | `endurance_mileage_v2_unit` | `str | None` | "km" or "--" when unavailable (confirmed) |
| `totalMileage` | `total_mileage` | `float | None` | Odometer in km (confirmed) |
| `totalMileageV2` | `total_mileage_v2` | `float | None` | Odometer v2 (unconfirmed) |
| `totalMileageV2Unit` | `total_mileage_v2_unit` | `str | None` | "km" (confirmed) |

### Driving

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `speed` | `speed` | `float | None` | km/h (confirmed) |
| `powerGear` | `power_gear` | `PowerGear | None` | 1=parked (confirmed), 3=drive (confirmed) |

### Climate

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `tempInCar` | `temp_in_car` | `float | None` | Interior temp in C; -129 means unavailable (confirmed) |
| `mainSettingTemp` | `main_setting_temp` | `int | None` | Cabin set temperature, integer (confirmed) |
| `mainSettingTempNew` | `main_setting_temp_new` | `float | None` | Cabin set temperature, precise C (unconfirmed) |
| `airRunState` | `air_run_state` | `AirCirculationMode | None` | 0=external (confirmed), 1=internal recirculation (confirmed) |

### Seat heating and ventilation

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `mainSeatHeatState` | `main_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `mainSeatVentilationState` | `main_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `copilotSeatHeatState` | `copilot_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `copilotSeatVentilationState` | `copilot_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `steeringWheelHeatState` | `steering_wheel_heat_state` | `SeatHeatVentState | int | None` | 0=off (confirmed), 1 observed (meaning unclear) |
| `lrSeatHeatState` | `lr_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |
| `lrSeatVentilationState` | `lr_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |
| `rrSeatHeatState` | `rr_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |
| `rrSeatVentilationState` | `rr_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |

Notes:

- Value `1` is observed for some seat and steering-wheel fields and is not a `SeatHeatVentState` member. pyBYD keeps it as a raw `int`.

### Charging

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `chargingState` | `charging_state` | `ChargingState | int` | -1=disconnected (confirmed), 0=not charging (confirmed), 15=gun connected not charging (confirmed via `chargeState`, unconfirmed here) |
| `chargeState` | `charge_state` | `ChargingState | int | None` | -1=disconnected (confirmed), 15=gun plugged in not charging (confirmed) |
| `waitStatus` | `wait_status` | `int | None` | charge wait status (unconfirmed) |
| `fullHour` | `full_hour` | `int | None` | hours to full, -1 means not available (confirmed) |
| `fullMinute` | `full_minute` | `int | None` | minutes to full, -1 means not available (confirmed) |
| `remainingHours` | `charge_remaining_hours` | `int | None` | -1 means not available (confirmed) |
| `remainingMinutes` | `charge_remaining_minutes` | `int | None` | -1 means not available (confirmed) |
| `bookingChargeState` | `booking_charge_state` | `int | None` | 0=off (confirmed) |
| `bookingChargingHour` | `booking_charging_hour` | `int | None` | scheduled charge start hour (unconfirmed) |
| `bookingChargingMinute` | `booking_charging_minute` | `int | None` | scheduled charge start minute (unconfirmed) |

### Doors

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `leftFrontDoor` | `left_front_door` | `DoorOpenState | None` | 0=closed (confirmed), 1=open (unconfirmed) |
| `rightFrontDoor` | `right_front_door` | `DoorOpenState | None` | 0=closed (confirmed), 1=open (unconfirmed) |
| `leftRearDoor` | `left_rear_door` | `DoorOpenState | None` | 0=closed (confirmed), 1=open (unconfirmed) |
| `rightRearDoor` | `right_rear_door` | `DoorOpenState | None` | 0=closed (confirmed), 1=open (unconfirmed) |
| `trunkLid` | `trunk_lid` | `DoorOpenState | None` | 0=closed (confirmed), 1=open (unconfirmed) |
| `slidingDoor` | `sliding_door` | `DoorOpenState | None` | 0=closed (confirmed), 1=open (unconfirmed) |
| `forehold` | `forehold` | `DoorOpenState | None` | frunk; 0=closed (confirmed), 1=open (unconfirmed) |

### Door locks

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `leftFrontDoorLock` | `left_front_door_lock` | `LockState | None` | 0=locked (unconfirmed), 1=unlocked (confirmed) |
| `rightFrontDoorLock` | `right_front_door_lock` | `LockState | None` | 0=locked (unconfirmed), 1=unlocked (confirmed) |
| `leftRearDoorLock` | `left_rear_door_lock` | `LockState | None` | 0=locked (unconfirmed), 1=unlocked (confirmed) |
| `rightRearDoorLock` | `right_rear_door_lock` | `LockState | None` | 0=locked (unconfirmed), 1=unlocked (confirmed) |
| `slidingDoorLock` | `sliding_door_lock` | `LockState | None` | 0=locked (unconfirmed), 1=unlocked (confirmed) |

### Windows

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `leftFrontWindow` | `left_front_window` | `WindowState | None` | 0=open (unconfirmed), 1=closed (confirmed) |
| `rightFrontWindow` | `right_front_window` | `WindowState | None` | 0=open (unconfirmed), 1=closed (confirmed) |
| `leftRearWindow` | `left_rear_window` | `WindowState | None` | 0=open (unconfirmed), 1=closed (confirmed) |
| `rightRearWindow` | `right_rear_window` | `WindowState | None` | 0=open (unconfirmed), 1=closed (confirmed) |
| `skylight` | `skylight` | `WindowState | None` | 0=open (unconfirmed), 1=closed (confirmed) |

### Tire pressure

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `leftFrontTirepressure` | `left_front_tire_pressure` | `float | None` | pressure value in unit given by `tirePressUnit` (confirmed) |
| `rightFrontTirepressure` | `right_front_tire_pressure` | `float | None` | pressure value in unit given by `tirePressUnit` (confirmed) |
| `leftRearTirepressure` | `left_rear_tire_pressure` | `float | None` | pressure value in unit given by `tirePressUnit` (confirmed) |
| `rightRearTirepressure` | `right_rear_tire_pressure` | `float | None` | pressure value in unit given by `tirePressUnit` (confirmed) |
| `leftFrontTireStatus` | `left_front_tire_status` | `int | None` | 0=normal (confirmed) |
| `rightFrontTireStatus` | `right_front_tire_status` | `int | None` | 0=normal (confirmed) |
| `leftRearTireStatus` | `left_rear_tire_status` | `int | None` | 0=normal (confirmed) |
| `rightRearTireStatus` | `right_rear_tire_status` | `int | None` | 0=normal (confirmed) |
| `tirePressUnit` | `tire_press_unit` | `TirePressureUnit | None` | 1=bar (confirmed), 2=psi (unconfirmed), 3=kPa (unconfirmed) |
| `tirepressureSystem` | `tirepressure_system` | `int | None` | TPMS system state (unconfirmed) |
| `rapidTireLeak` | `rapid_tire_leak` | `int | None` | 0=no leak (confirmed) |

### Energy consumption strings (realtime)

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `totalPower` | `total_power` | `float | None` | total power (unconfirmed) |
| `totalEnergy` | `total_energy` | `str | None` | "--" when unavailable (confirmed) |
| `nearestEnergyConsumption` | `nearest_energy_consumption` | `str | None` | "--" when unavailable (confirmed) |
| `nearestEnergyConsumptionUnit` | `nearest_energy_consumption_unit` | `str | None` | unit string (unconfirmed) |
| `recent50kmEnergy` | `recent_50km_energy` | `str | None` | "--" when unavailable (confirmed) |

### Fuel (hybrid vehicles)

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `oilEndurance` | `oil_endurance` | `float | None` | -1 means not applicable for EV (confirmed) |
| `oilPercent` | `oil_percent` | `float | None` | 0 for EV (confirmed) |
| `totalOil` | `total_oil` | `float | None` | 0 for EV (confirmed) |

### System indicators and warning lights

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `powerSystem` | `power_system` | `int | None` | 0=normal (confirmed) |
| `engineStatus` | `engine_status` | `int | None` | 0=off (confirmed) |
| `epb` | `epb` | `int | None` | 0=released (confirmed) |
| `eps` | `eps` | `int | None` | 0=normal (confirmed) |
| `esp` | `esp` | `int | None` | 0=normal (confirmed) |
| `abs` | `abs_warning` | `int | None` | 0=normal (confirmed) |
| `svs` | `svs` | `int | None` | 0=normal (confirmed) |
| `srs` | `srs` | `int | None` | 0=normal (confirmed) |
| `ect` | `ect` | `int | None` | 0=normal (confirmed) |
| `ectValue` | `ect_value` | `int | None` | -1 means not available (confirmed) |
| `pwr` | `pwr` | `int | None` | 2 observed (unconfirmed) |

### Feature states

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `sentryStatus` | `sentry_status` | `int | None` | 0=off (unconfirmed), 1=on (unconfirmed), 2 observed (unconfirmed) |
| `batteryHeatState` | `battery_heat_state` | `int | None` | 0=off (confirmed) |
| `chargeHeatState` | `charge_heat_state` | `int | None` | 0=off (confirmed) |
| `upgradeStatus` | `upgrade_status` | `int | None` | 0=none (confirmed) |

### Metadata

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `time` | `timestamp` | `int | None` | epoch seconds (confirmed) |

---

## HVAC / climate status

URL: https://dilinkappoversea-eu.byd.auto/control/getStatusNow

Model: `HvacStatus`

Parser: `src/pybyd/_api/hvac.py`

Response wraps data under the `statusNow` key.

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `acSwitch` | `ac_switch` | `int | None` | 0=off, 1=on (unconfirmed) |
| `status` | `status` | `int | None` | overall HVAC status (unconfirmed) |
| `airConditioningMode` | `air_conditioning_mode` | `int | None` | mode (unconfirmed) |
| `windMode` | `wind_mode` | `int | None` | fan mode (unconfirmed) |
| `windPosition` | `wind_position` | `int | None` | airflow direction (unconfirmed) |
| `cycleChoice` | `cycle_choice` | `int | None` | 1=external circulation (confirmed) |
| `mainSettingTemp` | `main_setting_temp` | `int | None` | set temp integer (unconfirmed) |
| `mainSettingTempNew` | `main_setting_temp_new` | `float | None` | set temp C (unconfirmed) |
| `copilotSettingTemp` | `copilot_setting_temp` | `int | None` | passenger set temp (unconfirmed) |
| `copilotSettingTempNew` | `copilot_setting_temp_new` | `float | None` | passenger set temp C (unconfirmed) |
| `tempInCar` | `temp_in_car` | `float | None` | interior C; -129 means unavailable (confirmed) |
| `tempOutCar` | `temp_out_car` | `float | None` | exterior C (unconfirmed) |
| `whetherSupportAdjustTemp` | `whether_support_adjust_temp` | `int | None` | 1=supported (confirmed) |
| `frontDefrostStatus` | `front_defrost_status` | `int | None` | 0=off (unconfirmed) |
| `electricDefrostStatus` | `electric_defrost_status` | `int | None` | 0=off (unconfirmed) |
| `wiperHeatStatus` | `wiper_heat_status` | `int | None` | 0=off (unconfirmed) |
| `mainSeatHeatState` | `main_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `mainSeatVentilationState` | `main_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `copilotSeatHeatState` | `copilot_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `copilotSeatVentilationState` | `copilot_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off, 2=low, 3=high (confirmed) |
| `steeringWheelHeatState` | `steering_wheel_heat_state` | `SeatHeatVentState | int | None` | 0=off (confirmed), 1 observed (unconfirmed) |
| `lrSeatHeatState` | `lr_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |
| `lrSeatVentilationState` | `lr_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |
| `rrSeatHeatState` | `rr_seat_heat_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |
| `rrSeatVentilationState` | `rr_seat_ventilation_state` | `SeatHeatVentState | int | None` | 0=off (confirmed) |
| `rapidIncreaseTempState` | `rapid_increase_temp_state` | `int | None` | 0=off (confirmed) |
| `rapidDecreaseTempState` | `rapid_decrease_temp_state` | `int | None` | 0=off (confirmed) |
| `refrigeratorState` | `refrigerator_state` | `int | None` | 0=off (confirmed) |
| `refrigeratorDoorState` | `refrigerator_door_state` | `int | None` | 0=closed (confirmed) |
| `pm` | `pm` | `int | None` | PM2.5 value (unconfirmed) |
| `pm25StateOutCar` | `pm25_state_out_car` | `int | None` | outside PM2.5 state (unconfirmed) |

---

## Charging status

URL: https://dilinkappoversea-eu.byd.auto/control/smartCharge/homePage

Model: `ChargingStatus`

Parser: `src/pybyd/_api/charging.py`

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `vin` | `vin` | `str` | VIN |
| `soc` | `soc` | `int | None` | SOC 0-100 (confirmed) |
| `chargingState` | `charging_state` | `int | None` | 15 means not charging (confirmed); other active charging values not captured yet |
| `connectState` | `connect_state` | `int | None` | 0=not connected (confirmed), 1=connected (confirmed) |
| `waitStatus` | `wait_status` | `int | None` | 0 (confirmed) |
| `fullHour` | `full_hour` | `int | None` | -1 means not available (confirmed) |
| `fullMinute` | `full_minute` | `int | None` | -1 means not available (confirmed) |
| `updateTime` | `update_time` | `int | None` | epoch seconds (confirmed) |

---

## GPS / location

URL (trigger): https://dilinkappoversea-eu.byd.auto/control/getGpsInfo

URL (poll): https://dilinkappoversea-eu.byd.auto/control/getGpsInfoResult

Model: `GpsInfo`

Parser: `src/pybyd/_api/gps.py`

Note: the parser accepts multiple aliases for some fields.

| API field (aliases) | Python field | Type | Values / notes |
|---|---|---|---|
| `latitude` / `lat` / `gpsLatitude` | `latitude` | `float | None` | degrees (confirmed) |
| `longitude` / `lng` / `lon` / `gpsLongitude` | `longitude` | `float | None` | degrees (confirmed) |
| `speed` / `gpsSpeed` | `speed` | `float | None` | km/h (unconfirmed) |
| `direction` / `heading` / `course` | `direction` | `float | None` | degrees 0-360 (confirmed) |
| `gpsTimeStamp` / `gpsTimestamp` / `gpsTime` / `time` / `uploadTime` | `gps_timestamp` | `int | None` | epoch seconds (confirmed) |
| `requestSerial` | `request_serial` | `str | None` | poll serial token (unconfirmed) |

---

## Energy consumption

URL: https://dilinkappoversea-eu.byd.auto/vehicleInfo/vehicle/getEnergyConsumption

Model: `EnergyConsumption`

Parser: `src/pybyd/_api/energy.py`

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `vin` | `vin` | `str` | VIN |
| `totalEnergy` | `total_energy` | `float | None` | string "--" maps to None (confirmed) |
| `avgEnergyConsumption` | `avg_energy_consumption` | `float | None` | unconfirmed |
| `electricityConsumption` | `electricity_consumption` | `float | None` | unconfirmed |
| `fuelConsumption` | `fuel_consumption` | `float | None` | unconfirmed |

Note: when the API returns error `1001`, the parser can synthesise partial data from the realtime cache.

---

## Vehicle list

URL: https://dilinkappoversea-eu.byd.auto/app/account/getAllListByUserId

Model: `Vehicle`

Parser: `src/pybyd/_api/vehicles.py`

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `vin` | `vin` | `str` | VIN |
| `modelName` | `model_name` | `str` | model name (confirmed) |
| `brandName` | `brand_name` | `str` | brand name (confirmed) |
| `energyType` | `energy_type` | `str` | "0" for EV (confirmed) |
| `autoAlias` | `auto_alias` | `str | None` | user alias (unconfirmed) |
| `autoPlate` | `auto_plate` | `str | None` | license plate (unconfirmed) |
| `cfPic.picMainUrl` | `pic_main_url` | `str | None` | image URL (unconfirmed) |
| `cfPic.picSetUrl` | `pic_set_url` | `str | None` | image URL set (unconfirmed) |
| `outModelType` | `out_model_type` | `str | None` | external model label (unconfirmed) |
| `totalMileage` | `total_mileage` | `float | None` | odometer (unconfirmed) |
| `modelId` | `model_id` | `int | None` | internal model id (unconfirmed) |
| `carType` | `car_type` | `int | None` | internal car type id (unconfirmed) |
| `defaultCar` | `default_car` | `bool` | 1 maps to True (confirmed) |
| `empowerType` | `empower_type` | `int | None` | 2=owner (confirmed), -1=shared user (confirmed) |
| `permissionStatus` | `permission_status` | `int | None` | 2 observed for full permissions (confirmed) |
| `tboxVersion` | `tbox_version` | `str | None` | e.g. "3" (unconfirmed) |
| `vehicleState` | `vehicle_state` | `str | None` | e.g. "1" (unconfirmed) |
| `autoBoughtTime` | `auto_bought_time` | `int | None` | epoch ms (unconfirmed) |
| `yunActiveTime` | `yun_active_time` | `int | None` | epoch ms (unconfirmed) |
| `empowerId` | `empower_id` | `int | None` | empower relationship id (confirmed) |
| `rangeDetailList` | `range_detail_list` | `list[EmpowerRange]` | permission scopes (confirmed) |

---

## Remote control

URL (trigger): https://dilinkappoversea-eu.byd.auto/control/remoteControl

URL (poll): https://dilinkappoversea-eu.byd.auto/control/remoteControlResult

Model: `RemoteControlResult`

Parser: `src/pybyd/_api/control.py`

### Command types

| Python enum (`RemoteCommand`) | API value (`commandType`) | Description |
|---|---|---|
| `LOCK` | `LOCKDOOR` | lock all doors |
| `UNLOCK` | `OPENDOOR` | unlock all doors |
| `START_CLIMATE` | `OPENAIR` | start A/C |
| `STOP_CLIMATE` | `CLOSEAIR` | stop A/C |
| `SCHEDULE_CLIMATE` | `BOOKINGAIR` | schedule A/C |
| `FIND_CAR` | `FINDCAR` | find my car |
| `FLASH_LIGHTS` | `FLASHLIGHTNOWHISTLE` | flash lights |
| `CLOSE_WINDOWS` | `CLOSEWINDOW` | close windows |
| `SEAT_CLIMATE` | `VENTILATIONHEATING` | seat heat/vent |
| `BATTERY_HEAT` | `BATTERYHEAT` | battery heat |

### Result fields

| API field | Python field | Type | Values / notes |
|---|---|---|---|
| `controlState` | `control_state` | `ControlState` | 0=pending, 1=success, 2=failure (unconfirmed) |
| `requestSerial` | `request_serial` | `str | None` | poll serial token (unconfirmed) |
| `res` | (immediate) | `int` | 2 observed as success (unconfirmed) |

---

## Enum mappings (shared)

These reflect the enums currently implemented in `src/pybyd/models/realtime.py`.

### ChargingState

| Value | Meaning | Status |
|---:|---|---|
| -1 | disconnected | confirmed |
| 0 | not charging | confirmed |
| 15 | gun connected, not charging | confirmed |

### PowerGear

| Value | Meaning | Status |
|---:|---|---|
| 1 | parked | confirmed |
| 3 | drive | confirmed |

### SeatHeatVentState

| Value | Meaning | Status |
|---:|---|---|
| 0 | off | confirmed |
| 2 | low | confirmed |
| 3 | high | confirmed |

Notes:

- The status scale differs from the remote control command scale.
- Value `1` is observed and is kept as a raw integer.

### AirCirculationMode

| Value | Meaning | Status |
|---:|---|---|
| 0 | external | confirmed |
| 1 | internal recirculation | confirmed |

### TirePressureUnit

| Value | Meaning | Status |
|---:|---|---|
| 1 | bar | confirmed |
| 2 | psi | unconfirmed |
| 3 | kPa | unconfirmed |

### WindowState

| Value | Meaning | Status |
|---:|---|---|
| 0 | open | unconfirmed |
| 1 | closed | confirmed |

---

## Unparsed fields

Some endpoints contain additional fields that pyBYD currently keeps only in `raw`.
If you discover value meanings, add them here and consider mapping them into models.

Realtime examples (not exhaustively maintained):

| API field | Observed value | Notes |
|---|---|---|
| `rate` | `-999` | possibly charging rate (unconfirmed) |
| `lessOneMin` | `false` | possibly time-to-full flag (unconfirmed) |
