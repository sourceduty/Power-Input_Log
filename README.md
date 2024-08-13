![Power-Input Log](https://github.com/user-attachments/assets/43ab8965-d390-47aa-8b8b-6acb73fc313a)

> Log the total time a computer is powered on, and the time spent inputting or idling while powered on.

#

The Power-Input Log is a system feature designed to record and analyze the usage patterns of a computer. Specifically, it tracks the total time a computer is powered on and the time actively spent inputting (via keyboard or mouse) while powered on. The feature aims to provide insights into the operational efficiency of a computer by calculating the total running time from the purchase date, total idle time (powered on but unused), and the total active input time.

The Power-Input Log works by starting a timer each time the computer is powered on. Simultaneously, it monitors user input activity through keyboard and mouse events. When the computer is idle, with no input detected for a predefined period, the system notes this as idle time. The total running time is calculated from the moment the system is first powered on after purchase and continues to accumulate until the system is powered off.

To effectively calculate these metrics, the system maintains logs that timestamp when the computer is turned on and off, and when input activity is detected or stops. These logs are then processed to compute the total active and idle times. The Power-Input Log will periodically update these metrics and store them for users to review. This feature can be especially useful for both personal and enterprise environments to monitor device usage and optimize power management strategies.

Incorporating the Power-Input Log into Windows 11 will involve utilizing system APIs to detect power state changes and input events, and then storing this data in a local database or log file. Users can access a simple interface that displays their computer's total running time since purchase, active input time, and idle time. This feature can also include options for exporting the log data for further analysis.

#
### Python Code Implementation

```
import time![Power-Input Log](https://github.com/user-attachments/assets/2fe16f2a-9ffc-484d-900f-53274301eec1)

import datetime
import os
import json
from pynput import keyboard, mouse

# Define file path for saving logs
log_file_path = os.path.join(os.getenv('USERPROFILE'), 'power_input_log.json')

# Load or initialize log data
if os.path.exists(log_file_path):
    with open(log_file_path, 'r') as file:
        log_data = json.load(file)
else:
    log_data = {
        "purchase_date": str(datetime.datetime.now()),
        "power_on_times": [],
        "input_times": []
    }

# Function to calculate time differences
def calculate_time_diff(start_time, end_time):
    delta = end_time - start_time
    return delta.total_seconds()

# Function to update log data
def update_log():
    with open(log_file_path, 'w') as file:
        json.dump(log_data, file)

# Event handlers for power and input monitoring
def on_power_on():
    log_data['power_on_times'].append({
        "start_time": str(datetime.datetime.now()),
        "end_time": None
    })
    update_log()

def on_power_off():
    if log_data['power_on_times'] and log_data['power_on_times'][-1]['end_time'] is None:
        log_data['power_on_times'][-1]['end_time'] = str(datetime.datetime.now())
        update_log()

def on_input():
    if not log_data['input_times'] or (log_data['input_times'][-1]['end_time'] is not None):
        log_data['input_times'].append({
            "start_time": str(datetime.datetime.now()),
            "end_time": None
        })
    update_log()

def on_input_end():
    if log_data['input_times'] and log_data['input_times'][-1]['end_time'] is None:
        log_data['input_times'][-1]['end_time'] = str(datetime.datetime.now())
        update_log()

# Start monitoring power on event
on_power_on()

# Monitor keyboard and mouse events
keyboard_listener = keyboard.Listener(on_press=on_input, on_release=on_input_end)
mouse_listener = mouse.Listener(on_move=on_input, on_click=on_input, on_scroll=on_input)

keyboard_listener.start()
mouse_listener.start()

# Keep the script running
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    on_power_off()
    print("Script terminated by user.")
    update_log()

# Calculate total running, active, and idle time
def calculate_statistics():
    total_running_time = 0
    total_active_input_time = 0
    for power_session in log_data['power_on_times']:
        start_time = datetime.datetime.fromisoformat(power_session['start_time'])
        end_time = datetime.datetime.fromisoformat(power_session['end_time']) if power_session['end_time'] else datetime.datetime.now()
        total_running_time += calculate_time_diff(start_time, end_time)
    
    for input_session in log_data['input_times']:
        start_time = datetime.datetime.fromisoformat(input_session['start_time'])
        end_time = datetime.datetime.fromisoformat(input_session['end_time']) if input_session['end_time'] else datetime.datetime.now()
        total_active_input_time += calculate_time_diff(start_time, end_time)
    
    total_idle_time = total_running_time - total_active_input_time
    print(f"Total running time (in seconds): {total_running_time}")
    print(f"Total active input time (in seconds): {total_active_input_time}")
    print(f"Total idle time (in seconds): {total_idle_time}")

calculate_statistics()
```

This Python script is a basic implementation of the Power-Input Log feature for Windows 11. It tracks when the computer is powered on and monitors input activities using the pynput library. The script logs these activities and calculates total running time, active input time, and idle time. Note that additional error handling and integration with Windows-specific power events (like sleep, hibernate) would be necessary for a production environment.

#
### Related Links

[ChatGPT](https://github.com/sourceduty/ChatGPT)
<br>
[Power-Time Log](https://github.com/sourceduty/Power-Time_Logger)

***
Copyright (C) 2024, Sourceduty - All Rights Reserved.
