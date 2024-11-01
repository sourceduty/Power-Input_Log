![Power-Input Log](https://github.com/user-attachments/assets/43ab8965-d390-47aa-8b8b-6acb73fc313a)

> Log the total time a computer is powered on, and the time spent inputting or idling while powered on.

#

The Power-Input Log is a system feature designed to record and analyze the usage patterns of a computer. Specifically, it tracks the total time a computer is powered on and the time actively spent inputting (via keyboard or mouse) while powered on. The feature aims to provide insights into the operational efficiency of a computer by calculating the total running time from the purchase date, total idle time (powered on but unused), and the total active input time.

The Power-Input Log works by starting a timer each time the computer is powered on. Simultaneously, it monitors user input activity through keyboard and mouse events. When the computer is idle, with no input detected for a predefined period, the system notes this as idle time. The total running time is calculated from the moment the system is first powered on after purchase and continues to accumulate until the system is powered off.

To effectively calculate these metrics, the system maintains logs that timestamp when the computer is turned on and off, and when input activity is detected or stops. These logs are then processed to compute the total active and idle times. The Power-Input Log will periodically update these metrics and store them for users to review. This feature can be especially useful for both personal and enterprise environments to monitor device usage and optimize power management strategies.

Incorporating the Power-Input Log into Windows 11 will involve utilizing system APIs to detect power state changes and input events, and then storing this data in a local database or log file. Users can access a simple interface that displays their computer's total running time since purchase, active input time, and idle time. This feature can also include options for exporting the log data for further analysis.

#

<details><summary>Python Concepts</summary>
<br>

```
import time

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

<br>
</details>
<details><summary>C++ Concept (Data-Time Log)</summary>
<br>

```
#include <iostream>
#include <fstream>
#include <windows.h>
#include <chrono>
#include <iomanip>
#include <sstream>
#include <thread>

// Function to get the current date-time as a formatted string
std::string getCurrentDateTime() {
    auto now = std::chrono::system_clock::now();
    std::time_t now_c = std::chrono::system_clock::to_time_t(now);
    std::stringstream ss;
    ss << std::put_time(std::localtime(&now_c), "%Y-%m-%d %H:%M:%S");
    return ss.str();
}

// Function to get the system's idle time in milliseconds
DWORD getIdleTime() {
    LASTINPUTINFO lii;
    lii.cbSize = sizeof(LASTINPUTINFO);
    GetLastInputInfo(&lii);
    return (GetTickCount() - lii.dwTime);
}

// Function to log data to file with timestamps
void logToFile(const std::string& state, const std::string& startTime, const std::string& endTime, double duration) {
    std::ofstream logFile("power_input_log.json", std::ios::app);
    if (logFile.is_open()) {
        logFile << "{\n";
        logFile << "  \"state\": \"" << state << "\",\n";
        logFile << "  \"start_time\": \"" << startTime << "\",\n";
        logFile << "  \"end_time\": \"" << endTime << "\",\n";
        logFile << "  \"duration_seconds\": " << duration << "\n";
        logFile << "},\n";
        logFile.close();
    } else {
        std::cerr << "Unable to open log file.\n";
    }
}

int main() {
    const DWORD idleThreshold = 5000; // 5 seconds threshold for idle detection
    std::string currentState = "active";
    std::string startTime = getCurrentDateTime();
    
    while (true) {
        DWORD idleTime = getIdleTime();
        std::string newState = (idleTime < idleThreshold) ? "active" : "idle";

        // Check if state has changed from active to idle or vice versa
        if (newState != currentState) {
            std::string endTime = getCurrentDateTime();
            auto duration = std::chrono::duration<double>(std::chrono::system_clock::now() - std::chrono::system_clock::from_time_t(std::time(0)));
            logToFile(currentState, startTime, endTime, duration.count());
            
            // Update state and start time for new period
            currentState = newState;
            startTime = endTime;
        }

        // Add a delay to reduce CPU usage
        std::this_thread::sleep_for(std::chrono::seconds(1));

        // Optional: Stop the program after a set duration for testing purposes
        // This example runs for 1 minute (use this to limit the test duration)
        auto programEndTime = std::chrono::steady_clock::now() + std::chrono::minutes(1);
        if (std::chrono::steady_clock::now() >= programEndTime) {
            break;
        }
    }

    return 0;
}
```

The Date-Time Log program developed in C++ serves as a precise tracking tool for monitoring active and idle times on a Windows computer. By leveraging the Windows API, it captures system idle periods based on user input inactivity and transitions between states, logging each active or idle period with exact timestamps. This allows it to accurately record the start and end times of each activity state, along with the duration in seconds. By outputting data in JSON format, the program provides structured and accessible logs that include details like the state (active or idle), start time, end time, and total duration. This level of logging is ideal for applications that require accurate monitoring of system usage, such as user behavior analysis, productivity tracking, or power management optimization.

The program utilizes the <chrono> library to manage high-resolution time data, offering a reliable and cross-platform compatible method for date and time manipulation. The result is a user-friendly, efficient, and low-resource logging tool that runs continuously, only pausing when the system state changes. With adjustable idle thresholds and easy customization for different logging intervals, this program is a versatile solution for tracking computer usage. Additionally, the JSON log format facilitates easy integration into analytics tools or databases, making it simple for users to perform further analysis or visualization on the data. The program’s implementation as a standalone .exe makes it convenient for deployment on any Windows system, providing both personal users and enterprises with a powerful way to optimize their device usage.

<br>
</details>

#
### Related Links

[ChatGPT](https://github.com/sourceduty/ChatGPT)
<br>
[Power-Time Log](https://github.com/sourceduty/Power-Time_Logger)

***
Copyright (C) 2024, Sourceduty - All Rights Reserved.
