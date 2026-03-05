# Uge 15 — IoT & Raspberry Pi

Denne uge introducerer Internet of Things (IoT) — en af de hurtigst voksende teknologitrends. Vi ser på Raspberry Pi som en embedded computing-platform, SenseHAT-modulet med dets sensorer og LED-matrix, og GPIO-programmering med Python.

---

## 1. IoT — Internet of Things

### Hvad er IoT?

**Internet of Things (IoT)** refererer til et netværk af fysiske enheder (objekter) der er forbundet til internettet og kan indsamle og udveksle data.

"Tingene" i IoT er alt fra køleskabe og termostateter til fabriksmaskiner, biler og medicinske enheder.

```
Sensor/Aktuator
    ↓  (data)
Embedded Device (Pi, Arduino, ESP32)
    ↓  (WiFi/4G/LoRa)
Cloud/Server
    ↓  (API)
Dashboard / App / Analyse
```

### IoT Komponenter

| Komponent        | Rolle                                              | Eksempel                         |
|------------------|----------------------------------------------------|----------------------------------|
| **Sensor**       | Måler fysisk størrelse, omsætter til digitalt signal | Temperatursensor, bevægelsessensor |
| **Aktuator**     | Udfører handling baseret på digitalt signal         | Motor, LED, relæ                |
| **Mikrocontroller** | Lille computer til enkel behandling             | Arduino, ESP32                  |
| **Single-Board Computer** | Lille computer med fuldt OS              | Raspberry Pi                    |
| **Gateway**      | Forbinder IoT-enheder til internet                  | WiFi-router med MQTT-broker      |
| **Cloud platform** | Gemmer og analyserer data                        | AWS IoT, Azure IoT Hub          |

### IoT Anvendelser

| Område            | Eksempler                                                  |
|-------------------|------------------------------------------------------------|
| Smart hjem        | Termostat (Nest), smart pærer, døralarmer                  |
| Industri (IIoT)   | Maskinsundhedsovervågning, produktionsstyring              |
| Sundhed           | Smartwatches, insulinpumper, fjernpatientovervågning       |
| Landbrug          | Jordfugtighedssensorer, automatisk vanding                 |
| Transport         | Sporings-GPS i pakker, autonome køretøjer                  |
| Smart city        | Intelligent trafiklys, affaldsspande der rapporterer fylde |

---

## 2. Embedded Systems

### Hvad er embedded systems?

Et **embedded system** er et computersystem designet til at udføre **en specifik dedikeret funktion** indlejret i en større enhed. Det er i modsætning til en almen computer der kan køre mange programmer.

```
Eksempler:
- Mikrobølgeovnens controller
- Bilens ABS-system
- Et digitalt ur
- En smart termostat
- En Raspberry Pi konfigureret til at måle temperatur
```

### Embedded vs. General Purpose

| Aspekt          | Embedded System                | General Purpose (PC)          |
|-----------------|-------------------------------|-------------------------------|
| Formål          | Specifik opgave                | Mange opgaver                 |
| OS              | Oftest intet eller RTOS        | Windows, Linux, macOS         |
| Strøm           | Lavt forbrug (mA)              | Højt forbrug (Watt)           |
| Interface       | Sensorer, GPIO, LCD            | Tastatur, skærm, mus          |
| Pris            | Lavt (kr.)                    | Højt (kr.)                   |
| Eksempel        | Arduino, ESP32                 | Laptop, desktop               |

---

## 3. Raspberry Pi

### Hvad er Raspberry Pi?

Raspberry Pi er en **single-board computer (SBC)** — en lille, billig computer på ét printkort. Det ligner en embedded controller, men kører et fuldt Linux-baseret OS (Raspberry Pi OS, baseret på Debian).

```
+------------------------------------------+
|           Raspberry Pi 4 Model B         |
|                                           |
|  +-------+  +------+  USB USB  HDMI HDMI |
|  |  CPU  |  | RAM  |  [ ] [ ]  [==] [==] |
|  | BCM   |  | 4GB  |                     |
|  | 2711  |  +------+  LAN                 |
|  +-------+           [====]               |
|                                           |
|  GPIO Pins (40 pin)                       |
|  [====================================]   |
|                                           |
|  microSD             USB-C Power          |
|  [=]                 [=]                  |
+------------------------------------------+
```

### Raspberry Pi Hardware specifikationer (Pi 4B)

| Komponent    | Specifikation                                        |
|--------------|------------------------------------------------------|
| CPU          | Broadcom BCM2711, 4x ARM Cortex-A72 @ 1.5GHz        |
| RAM          | 1GB, 2GB, 4GB eller 8GB LPDDR4                      |
| Lager        | microSD (ingen intern SSD/HDD)                       |
| GPIO         | 40-pin GPIO header                                   |
| USB          | 2x USB 3.0, 2x USB 2.0                              |
| Video        | 2x micro-HDMI (op til 4K)                           |
| Netværk      | Gigabit Ethernet, WiFi 802.11ac, Bluetooth 5.0       |
| Strøm        | 5V via USB-C                                         |
| Pris         | ca. 350-700 kr.                                     |

### Raspberry Pi OS

```bash
# System info
uname -a             # Kernel version
cat /etc/os-release  # OS version
vcgencmd measure_temp # CPU temperatur

# Pakker
sudo apt update
sudo apt install python3-pip

# Python
python3 --version
pip3 install requests
```

---

## 4. GPIO — General Purpose Input/Output

**GPIO** er 40 stifter (pins) på Raspberry Pi der kan bruges til at forbinde elektronik.

### GPIO Pin Layout (40-pin)

```
+---+---+
| 1 | 2 |   1=3.3V       2=5V
| 3 | 4 |   3=GPIO2(SDA) 4=5V
| 5 | 6 |   5=GPIO3(SCL) 6=GND
| 7 | 8 |   7=GPIO4      8=GPIO14(TX)
| 9 |10 |   9=GND        10=GPIO15(RX)
|11 |12 |   11=GPIO17    12=GPIO18
|13 |14 |   13=GPIO27    14=GND
...
|39 |40 |   39=GND       40=GPIO21
+---+---+
```

**Pin typer:**
- **3.3V** — strøm ud (3.3V)
- **5V** — strøm ud (5V)
- **GND** — jord (ground)
- **GPIO** — programmerbar digital ind/ud

### GPIO med Python (RPi.GPIO)

```python
import RPi.GPIO as GPIO
import time

# Brug BCM pin-nummerering
GPIO.setmode(GPIO.BCM)

LED_PIN = 17  # GPIO17

# Sæt LED-pin som output
GPIO.setup(LED_PIN, GPIO.OUT)

# Blink LED 5 gange
for i in range(5):
    GPIO.output(LED_PIN, GPIO.HIGH)  # Tænd LED
    time.sleep(0.5)
    GPIO.output(LED_PIN, GPIO.LOW)   # Sluk LED
    time.sleep(0.5)

# Ryd op (vigtigt!)
GPIO.cleanup()
```

### Læs input (knap)

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
KNAP_PIN = 27

GPIO.setup(KNAP_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)

print("Tryk på knappen...")
while True:
    if GPIO.input(KNAP_PIN) == GPIO.LOW:  # Knap trykket
        print("Knap trykket!")
        time.sleep(0.2)  # Debounce
    time.sleep(0.1)
```

---

## 5. SenseHAT

**SenseHAT** er et tilbehørsmodul (HAT = Hardware Attached on Top) til Raspberry Pi. Det indeholder:

```
+------------------------------+
|         SenseHAT             |
|                              |
|  [8x8 RGB LED Matrix]        |
|                              |
|  Sensorer:                   |
|  - Temperatur                |
|  - Luftfugtighed             |
|  - Lufttryk (barometer)      |
|  - Gyroskop                  |
|  - Accelerometer             |
|  - Magnetometer (kompas)     |
|                              |
|  5-knaps joystick            |
+------------------------------+
```

### SenseHAT installation

```bash
sudo apt install sense-hat
pip3 install sense-hat
```

### SenseHAT — LED Matrix

```python
from sense_hat import SenseHat
import time

sense = SenseHat()

# Sæt en enkelt pixel (x, y, (R, G, B))
sense.set_pixel(0, 0, (255, 0, 0))   # Rød pixel øverst til venstre
sense.set_pixel(7, 7, (0, 0, 255))   # Blå pixel nederst til højre

# Fyld hele matrix med én farve
sense.clear(0, 255, 0)   # Grøn
time.sleep(1)
sense.clear()             # Sluk

# Vis tekst (ruller hen over skærmen)
sense.show_message("Hej verden!", text_colour=(255, 255, 0))

# Vis bogstav
sense.show_letter("A", text_colour=(0, 255, 0))

# Tilpasset 8x8 billede (liste med 64 RGB-tuples)
r = (255, 0, 0)   # Rød
g = (0, 255, 0)   # Grøn
b = (0, 0, 255)   # Blå
e = (0, 0, 0)     # Slukket

billede = [
    e, e, e, e, e, e, e, e,
    e, r, e, e, e, e, r, e,
    e, e, r, e, e, r, e, e,
    e, e, e, r, r, e, e, e,
    e, e, e, r, r, e, e, e,
    e, e, r, e, e, r, e, e,
    e, r, e, e, e, e, r, e,
    e, e, e, e, e, e, e, e,
]
sense.set_pixels(billede)
```

### SenseHAT — Sensorer

```python
from sense_hat import SenseHat

sense = SenseHat()

# Temperatur (fra to sensorer)
temp_cpu = sense.get_temperature()          # Fra CPU-sensor
temp_humidity = sense.get_temperature_from_humidity()
temp_pressure = sense.get_temperature_from_pressure()

print(f"Temperatur: {temp_cpu:.1f} °C")

# Luftfugtighed
humidity = sense.get_humidity()
print(f"Luftfugtighed: {humidity:.1f} %")

# Lufttryk
pressure = sense.get_pressure()
print(f"Lufttryk: {pressure:.1f} hPa")

# Gyroskop (rotationshastighed i grader/sek)
gyro = sense.get_gyroscope_raw()
print(f"Gyro X: {gyro['x']:.2f}, Y: {gyro['y']:.2f}, Z: {gyro['z']:.2f}")

# Accelerometer (acceleration i G)
accel = sense.get_accelerometer_raw()
print(f"Accel X: {accel['x']:.2f}, Y: {accel['y']:.2f}, Z: {accel['z']:.2f}")

# Orientering (pitch, roll, yaw i grader)
orientation = sense.get_orientation()
print(f"Pitch: {orientation['pitch']:.1f}, Roll: {orientation['roll']:.1f}, Yaw: {orientation['yaw']:.1f}")

# Magnetometer (kompass)
mag = sense.get_compass_raw()
compass = sense.get_compass()
print(f"Kompas: {compass:.0f} grader")
```

### SenseHAT — Joystick

```python
from sense_hat import SenseHat
import time

sense = SenseHat()

print("Flyt joysticket...")

while True:
    for event in sense.stick.get_events():
        # event.direction: "up", "down", "left", "right", "middle"
        # event.action: "pressed", "released", "held"

        if event.action == "pressed":
            if event.direction == "up":
                print("Opad!")
                sense.clear(0, 255, 0)
            elif event.direction == "down":
                print("Nedad!")
                sense.clear(255, 0, 0)
            elif event.direction == "middle":
                print("Midt (tryk)!")
                sense.clear()

    time.sleep(0.01)
```

---

## 6. IoT Data til REST API

Raspberry Pi kan sende sensordata til et REST API:

```python
from sense_hat import SenseHat
import requests
import time

sense = SenseHat()
API_URL = "http://192.168.1.100:8080/api/sensordata"
INTERVAL = 30  # Sekunder

def laes_sensordata():
    return {
        "temperatur": round(sense.get_temperature(), 2),
        "luftfugtighed": round(sense.get_humidity(), 2),
        "lufttryk": round(sense.get_pressure(), 2),
        "timestamp": time.strftime("%Y-%m-%dT%H:%M:%S")
    }

def send_data(data):
    try:
        response = requests.post(
            API_URL,
            json=data,
            timeout=10,
            headers={"Content-Type": "application/json"}
        )
        if response.status_code == 201:
            print(f"Data sendt: {data['temperatur']}°C")
        else:
            print(f"Fejl: HTTP {response.status_code}")
    except requests.exceptions.ConnectionError:
        print("Ingen forbindelse til server")

# Hoved-løkke
print("Starter IoT datalogging...")
while True:
    data = laes_sensordata()
    send_data(data)
    time.sleep(INTERVAL)
```

---

## 7. Opsummering

| Emne              | Nogleinformation                                              |
|-------------------|---------------------------------------------------------------|
| IoT               | Fysiske enheder forbundet til internet, data indsamling       |
| Embedded System   | Dedikeret computer til specifik funktion (Arduino, ESP32)    |
| Raspberry Pi      | Single-board computer med fuldt Linux OS, 40 GPIO pins        |
| GPIO              | Programmerbare pins til sensorer/aktuatorer                  |
| SenseHAT          | Udvidelsesmodul med 8x8 LED matrix, temp/luftfugtighed/tryk  |
| SenseHAT sensorer | get_temperature(), get_humidity(), get_pressure(), get_orientation() |
| LED matrix        | set_pixel(), set_pixels(), show_message(), clear()           |
| IoT + REST        | Sensordata sendes til API via requests.post()                 |

---

## Opgaver til selvstudium

1. Hvad er forskellen på en embedded system og en Raspberry Pi?
2. Tegn et simpelt IoT-system der måler temperatur og sender data til en server. Inkludér: sensor, Raspberry Pi, netværk, server, database, dashboard.
3. Hvad er GPIO, og hvad bruges det til på Raspberry Pi?
4. Skriv Python-kode der læser temperatur fra SenseHAT og viser den på LED-matrixen som farvet baggrund: grøn hvis under 25°C, gul 25-30°C, rød over 30°C.
5. Forklar forskellen på en SenseHAT sensor og en GPIO-tilsluttet sensor.
