## Southern IoT: Multi-Zone Industrial Automation System

This project is a modular, scalable home/office automation infrastructure designed for high-reliability control of lighting and access points. It prioritizes local control and hardware-software synchronization to ensure manual switches and digital commands work in harmony.

---

### **System Architecture**

#### **Hardware Layer**
The hardware is distributed across three zones, utilizing the **ESP32** for more complex IO-heavy rooms and the **ESP8266** for smaller footprints.

* **Zone A (Main Room & Lab):** Powered by **ESP32 DevKit V1**. These nodes handle multiple relay outputs for lights, electromagnetic door locks, and magnetic reed switches for door status feedback.
* **Zone B (HR):** Powered by **NodeMCU (ESP8266)**. This node focuses on lighting control and PIR (Passive Infrared) motion sensing to test presence-based automation.
* **Peripherals:** * **Relays:** 5V Optocoupler-isolated relay modules to prevent back-EMF from damaging the microcontrollers.
    * **Sensors:** PIR HC-SR501 for motion; MC-38 Magnetic Sensors for door states.
    * **Physical Interface:** Wall switches connected via GPIO pull-up configuration to allow hybrid control.

#### **Software Layer**
* **Orchestration:** **Home Assistant** running on a dedicated local server.
* **Communication:** **ESPHome** native API for low-latency, bi-directional communication (preferred over MQTT for faster state synchronization).
* **Logic Engine:** YAML-based automation scripts for occupancy-based timers and security alerts.

---

### **Sample Implementation (ESPHome YAML)**

This code snippet demonstrates a standard configuration for a room (e.g., the Lab), featuring a light with physical switch override and an automated door lock.

```yaml
esphome:
  name: lab_controller
  platform: ESP32
  board: esp32dev

# Connectivity
networking:
  wifi:
    ssid: "Your_SSID"
    password: "Your_Password"

# Light Control with Physical Switch Synchronization
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO14
      mode: INPUT_PULLUP
    name: "Lab Wall Switch"
    on_press:
      - light.toggle: lab_main_light

  - platform: gpio
    pin:
      number: GPIO12
      mode: INPUT_PULLUP
    name: "Lab Door Status"
    device_class: door

# Outputs
switch:
  - platform: gpio
    pin: GPIO13
    name: "Lab Door Lock"
    inverted: true # For fail-safe electronic locks

light:
  - platform: binary
    name: "Lab Main Light"
    output: light_output

output:
  - platform: gpio
    pin: GPIO27
    id: light_output

# Local Automation: Motion-based power saving
sensor:
  - platform: gpio
    pin: GPIO33
    name: "Lab Motion Sensor"
    device_class: motion
    on_state:
      if:
        condition:
          binary_sensor.is_off: lab_motion_sensor
        then:
          - delay: 5min
          - light.turn_off: lab_main_light
```

---

### **Portfolio Highlights**
* **Hybrid Switching:** Successfully solved the "smart switch" dilemma by integrating physical toggles that update the digital state instantly without breaking the circuit.
* **Energy Optimization:** Implemented a 5-minute inactivity timeout across all zones, reducing unnecessary electricity consumption in unoccupied rooms.
* **Security Integration:** Centralized door monitoring and locking, providing a single source of truth for facility security within the Home Assistant dashboard.