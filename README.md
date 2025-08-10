# PiClock
The Official PiClock Repository

# License
Copyright 2025 Noxtec (NoXXoN)

All Rights Reserved.

You may view, download, and manufacture physical copies of this PCB and schematic design for **personal, non-commercial use only** (e.g., DIY projects or prototyping).

You may **not**:
- Distribute, share, or sell the design files or derived works.
- Manufacture boards for sale, resale, or commercial deployment.
- Host or mirror this content online.

For commercial licensing, manufacturing rights, or redistribution permissions, please contact:
noxxonyt@gmail.com

This license applies to all files in this project unless explicitly stated otherwise.

# Schematic
<img width="4092" height="2893" alt="PiClock" src="https://github.com/user-attachments/assets/059845b0-818e-44dc-9eab-ddd795eafa1c"/>
# PCB and schematic files
PCB: https://github.com/NoXXoN123/PiClock/blob/main/ClockPi.kicad_pcb  
Schematic: https://github.com/NoXXoN123/PiClock/blob/main/ClockPi.kicad_sch  
Kicad_Pro: https://github.com/NoXXoN123/PiClock/blob/main/ClockPi.kicad_pro  
# How to install
Install Thonny, Install MicroPython on a Pi Pico, Save the code below as 'main.py' on the directory of the Raspberry Pi by opening File Manager in Thonny (View > Files) or use Save As and choose the Raspberry Pi.
# Code:
main.py:

```
import time
from machine import Pin, PWM, ADC


def percent_to_duty(pct):
    return int((pct / 100.0) * 65535)

def get_light_level():
    raw = light_sensor.read_u16()     # 0–65535
    return raw >> 8                   # Scale to 0–255

def set_pwm_all(level):
    pwm_seconds.duty_u16(level)
    pwm_minutes.duty_u16(level)
    pwm_hours.duty_u16(level)

def light_control_loop():
    while True:
        light = get_light_level()

        if light < LIGHT_THRESHOLD:
            set_pwm_all(DIM_LEVEL)
        else:
            set_pwm_all(BRIGHT_LEVEL)

        time.sleep(0.1)  # Check every 100ms
def shift_out(data_pin, clock_pin, latch_pin, data_bytes):
    latch_pin.value(0)
    for byte in data_bytes:
        for bit in range(8):
            data_pin.value((byte >> (7 - bit)) & 1)
            clock_pin.value(1)
            clock_pin.value(0)
    latch_pin.value(1)
def seconds(*bits):
    # bits is a list of 64 bits for 8 chips * 8 bits
    # convert bits to bytes and send
    data_bytes = []
    for i in range(0, 64, 8):
        byte = 0
        for b in bits[i:i+8]:
            byte = (byte << 1) | (b & 1)
        data_bytes.append(byte)
    shift_out(DATA_PIN_SECONDS, CLOCK_PIN_SECONDS, LATCH_PIN_SECONDS, data_bytes)

def minutes(*bits):
    data_bytes = []
    for i in range(0, 64, 8):
        byte = 0
        for b in bits[i:i+8]:
            byte = (byte << 1) | (b & 1)
        data_bytes.append(byte)
    shift_out(DATA_PIN_MINUTES, CLOCK_PIN_MINUTES, LATCH_PIN_MINUTES, data_bytes)

def hours(*bits):
    data_bytes = []
    for i in range(0, 16, 8):
        byte = 0
        for b in bits[i:i+8]:
            byte = (byte << 1) | (b & 1)
        data_bytes.append(byte)
    shift_out(DATA_PIN_HOURS, CLOCK_PIN_HOURS, LATCH_PIN_HOURS, data_bytes)
def seconds(*bits):
    # your code to shift out bits to the seconds chips
    pass

def minutes(*bits):
    # same for minutes
    pass

def hours(*bits):
    # same for hours
    pass




# Pin setup - Don't touch this!
DATA_PIN_SECONDS = Pin(2, Pin.OUT)
CLOCK_PIN_SECONDS = Pin(3, Pin.OUT)
LATCH_PIN_SECONDS = Pin(4, Pin.OUT)

DATA_PIN_MINUTES = Pin(5, Pin.OUT)
CLOCK_PIN_MINUTES = Pin(6, Pin.OUT)
LATCH_PIN_MINUTES = Pin(7, Pin.OUT)

DATA_PIN_HOURS = Pin(9, Pin.OUT)
CLOCK_PIN_HOURS = Pin(8, Pin.OUT)
LATCH_PIN_HOURS = Pin(10, Pin.OUT)

PHOTORESISTOR_PIN = 26 # ADC0
PWM_SECONDS_PIN = 19
PWM_MINUTES_PIN = 20
PWM_HOURS_PIN = 21
BUTTON_HOUR = Pin(17, Pin.IN, Pin.PULL_DOWN)
BUTTON_MINUTE = Pin(16, Pin.IN, Pin.PULL_DOWN)
LED = Pin(25, Pin.OUT)
# Setup Variables - Don't touch this either!
SECOND_CHIPS = 8
MINUTE_CHIPS = 8
HOUR_CHIPS = 2
# USER SETTINGS - Touch this!
PWM_FREQ = 1000 #Hz DEFAULT: 1000
PWM_BRIGHT = percent_to_duty(60)  # DEFAULT: 60
PWM_DIM = percent_to_duty(10)     # DEFAULT: 10
LIGHT_THRESHOLD = 100 # DEFAULT: 100
# ---------------------
light_sensor = ADC(PHOTORESISTOR_PIN)
pwm_seconds = PWM(Pin(PWM_SECONDS_PIN))
pwm_minutes = PWM(Pin(PWM_MINUTES_PIN))
pwm_hours   = PWM(Pin(PWM_HOURS_PIN))
for pwm in (pwm_seconds, pwm_minutes, pwm_hours):
    pwm.freq(PWM_FREQ)

seconds_buffer = bytearray(SECOND_CHIPS)
minutes_buffer = bytearray(MINUTE_CHIPS)
hours_buffer = bytearray(HOUR_CHIPS)


# ---------------- Functions ----------------
def boot_animation_chase():
    for i in range(64):  # 64 steps for seconds and minutes
        sec_pattern = [0 if j == i else 1 for j in range(64)]
        min_pattern = [0 if j == (i + 20) % 64 else 1 for j in range(64)]  # offset for visual effect
        hr_pattern  = [0 if j == (i % 16) else 1 for j in range(16)]       # wrap 0–15

        seconds(*sec_pattern)
        minutes(*min_pattern)
        hours(*hr_pattern)

        time.sleep(0.03)


def fast_forward_boot(cycle_duration_sec=10):
    total_seconds = 12 * 60 * 60  # 12 hours = 43200 seconds
    delay = cycle_duration_sec / total_seconds

    for seconds_passed in range(total_seconds):
        sec = seconds_passed % 60
        min = (seconds_passed // 60) % 60
        hr  = (seconds_passed // 3600) % 12  # 12-hour format

        sec_buf = [1] * 64
        sec_buf[sec] = 0  # active low
        seconds(*sec_buf)

        min_buf = [1] * 64
        min_buf[min] = 0
        minutes(*min_buf)

        hr_buf = [1] * 16
        hr_buf[hr] = 0
        hours(*hr_buf)

        time.sleep(delay)

def run_clock():
    last_sec_update = 0
    last_min_update = 0
    last_hour_update = 0

    sec = 0
    minute = 0
    hour = 0

    last_button_hour = 0
    last_button_minute = 0
    debounce_ms = 200  # debounce delay

    led_on_time = 0
    led_duration = 200  # ms to keep LED on visibly

    while True:
        now = time.ticks_ms()

        if BUTTON_HOUR.value() == 1 and time.ticks_diff(now, last_button_hour) > debounce_ms:
            hour = (hour + 1) % 12
            hours(*[0 if i == hour else 1 for i in range(16)])
            last_button_hour = now
            last_hour_update = now

        if BUTTON_MINUTE.value() == 1 and time.ticks_diff(now, last_button_minute) > debounce_ms:
            minute = (minute + 1) % 60
            minutes(*[0 if i == minute else 1 for i in range(64)])
            last_button_minute = now
            last_min_update = now

        if time.ticks_diff(now, last_sec_update) >= 1000:
            sec = (sec + 1) % 60
            seconds(*[0 if i == sec else 1 for i in range(64)])
            last_sec_update = now

            LED.value(1)
            led_on_time = now

        if LED.value() == 1 and time.ticks_diff(now, led_on_time) > led_duration:
            LED.value(0)

        if time.ticks_diff(now, last_min_update) >= 60000:
            minute = (minute + 1) % 60
            minutes(*[0 if i == minute else 1 for i in range(64)])
            last_min_update = now

        if time.ticks_diff(now, last_hour_update) >= 3600000:
            hour = (hour + 1) % 12
            hours(*[0 if i == hour else 1 for i in range(16)])
            last_hour_update = now

        light = get_light_level()
        if light < LIGHT_THRESHOLD:
            set_pwm_all(PWM_DIM)
        else:
            set_pwm_all(PWM_BRIGHT)

        time.sleep(0.01) 


        
        
        

boot_animation_chase()
fast_forward_boot(10) # <-- 10 Seconds
run_clock()
```
