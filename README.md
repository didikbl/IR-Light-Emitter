# Build a 38 kHz IR Light Emitter for IR Receivers

## Introduction

Imagine you want to build an **infrared beam barrier** to detect when an object crosses a path.

Naturally, you need two circuits:

* An **IR transmitter**
* An **IR receiver**

For the receiver, a **photodiode** might be the first component that comes to mind. Technically, a photodiode can detect infrared light.

But there is a problem.

A simple photodiode cannot distinguish between the IR signal from our transmitter and infrared radiation coming from the **sun, lamps, and other sources**. The system could therefore produce false detections.

So how can we make the receiver detect **only our IR signal**?

This is where **integrated infrared receivers with built-in filtering** come in.

These receivers combine:

* A photodiode
* Amplification
* Filtering
* Signal processing

in a single package.

Their internal filtering is designed to detect infrared signals modulated at a specific **carrier frequency**, making them much less sensitive to continuous or unwanted infrared sources such as sunlight and ambient lighting.

Different receivers are designed for different carrier frequencies:

| Receiver       | Approximate Carrier Frequency |
| -------------- | ----------------------------: |
| TSOP36         |                        36 kHz |
| TSOP38         |                        38 kHz |
| Other variants |  30 kHz, 40 kHz, 56 kHz, etc. |

This also imposes a requirement on our transmitter.

We cannot simply turn the IR LED ON and OFF. We need to **modulate the emitted infrared light at the carrier frequency expected by the receiver**.

In this project, we are using a **38 kHz IR receiver**, so the transmitter must generate a **38 kHz carrier**.

We will therefore build a simple **38 kHz IR emitter** designed to work with 38 kHz IR receivers such as the TSOP38.

---

# What You'll Learn

By the end of this tutorial, you will be able to:

* Understand how an IR emitter built around **two astable oscillators** works.
* Calculate and select component values according to the type of IR receiver being used.
* Build and test the complete IR emitter circuit.
* Understand how a low-frequency oscillator can generate **bursts** of a high-frequency IR carrier.

---

# Step 1 — Understand the Circuit

The IR emitter is built around **two NE555 astable oscillators**.

```text
             ┌─────────────────────┐
             │  Oscillator 1       │
             │  ~10 Hz             │
             │  Burst Generator    │
             └──────────┬──────────┘
                        │
                        │ RESET
                        ▼
             ┌─────────────────────┐
             │  Oscillator 2       │
             │  ~38 kHz            │
             │  IR Carrier         │
             └──────────┬──────────┘
                        │
                        ▼
                     IR LED
```

The first oscillator runs at approximately **10 Hz** and controls the second oscillator.

The second oscillator generates the **38 kHz carrier signal**.

It is controlled through its **RESET pin** and only generates the 38 kHz signal when the output of the first oscillator is HIGH.

As a result, the IR LED does not emit a continuous 38 kHz signal. Instead, it emits:

```text
38 kHz bursts → pause → 38 kHz bursts → pause → ...
```

The first oscillator therefore acts as a **burst generator**, while the second oscillator generates the **38 kHz carrier** required by the IR receiver.

### Why Use Bursts?

The TSOP38 is designed to detect **bursts of modulated IR**, rather than a continuous 38 kHz signal.

Continuous transmission can cause its **automatic gain control (AGC)** and interference suppression mechanisms to reduce or reject the signal, preventing the receiver from responding correctly.

---

# Step 2 — Dimension the Two Astable Oscillators

## 2.1 — 10 Hz Astable Oscillator

The first oscillator is built around:

* `R1`
* `R2`
* `C1`

This RC network determines the oscillation frequency.

The period is:

```text
T = TON + TOFF
```

where:

* `TON` = time during which the output is HIGH
* `TOFF` = time during which the output is LOW

The duty cycle is:

```text
D = TON / T
```

For this project, we want a duty cycle close to **50%**.

For a standard NE555 astable configuration:

```text
TON  = 0.69 × (R1 + R2) × C
TOFF = 0.69 × R2 × C
```

Therefore:

```text
T = 0.69 × (R1 + 2R2) × C
```

Since:

```text
f = 1 / T
```

we want:

```text
f = 10 Hz
```

Therefore:

```text
T = 1 / 10
T = 0.1 s
```

### Selecting R1 and C

Let's choose:

```text
C1 = 10 µF = 0.00001 F
R1 = 1 kΩ = 1000 Ω
```

We choose `R1 = 1 kΩ` to obtain a duty cycle close to 50%.

### Calculating R2

Starting from:

```text
T = 0.69 × (R1 + 2R2) × C
```

we obtain:

```text
R2 = [T / (0.69 × C) - R1] / 2
```

Substituting:

```text
R2 = [0.1 / (0.69 × 0.00001) - 1000] / 2
```

```text
R2 ≈ 6746 Ω
```

So we can use the standard:

```text
R2 = 6.8 kΩ
```

### Final 10 Hz Oscillator Values

| Component |  Value |
| --------- | -----: |
| R1        |   1 kΩ |
| R2        | 6.8 kΩ |
| C1        |  10 µF |

This gives a frequency of approximately **9.9 Hz**, with a duty cycle of about **53%**, which is close to the 50% target.

---

# 2.2 — 38 kHz Astable Oscillator

The second NE555 generates the **38 kHz carrier** used to drive the IR LED.

We use the same astable configuration:

```text
T = 0.69 × (R1 + 2R2) × C
```

We want:

```text
f = 38 kHz
  = 38000 Hz
```

Therefore:

```text
T = 1 / 38000
T ≈ 0.0000263 s
```

### Selecting C2 and R5

Let's choose:

```text
C2 = 1 nF = 10⁻⁹ F
R5 = 1 kΩ = 1000 Ω
```

The required value of `R6` is calculated from:

```text
R6 = [T / (0.69 × C2) - R5] / 2
```

Using the target period gives a resistance in the approximately **18–20 kΩ** range.

Therefore, a **20 kΩ potentiometer** can be used.

This allows the carrier frequency to be adjusted precisely to **38 kHz**.

### Final 38 kHz Oscillator Values

| Component |               Value |
| --------- | ------------------: |
| R5        |                1 kΩ |
| R6        | 20 kΩ potentiometer |
| C2        |                1 nF |

> **Note:** The potentiometer allows the carrier frequency to be adjusted during testing using an oscilloscope.

---

# Step 3 — Dimension the IR LED Current-Limiting Resistor

The emitter uses **one IR LED**.

We want the LED to emit a strong infrared signal to increase the operating range.

A series resistor is therefore used to limit the LED current.

Using Kirchhoff's Voltage Law:

```text
VCC - VLED - VR = 0
```

Therefore:

```text
R = (VCC - VLED) / ILED
```

For:

```text
VCC  = 12 V
VLED = 2 V
ILED = 20 mA = 0.02 A
```

we obtain:

```text
R = (12 - 2) / 0.02
R = 500 Ω
```

A resistor close to this value can be selected.

The design uses approximately:

```text
RLED = 490 Ω
```

### Resistor Power

The resistor power is:

```text
P = R × I²
```

Using approximately 20 mA:

```text
P = 490 × (0.02)²
P ≈ 0.196 W
```

A **1/2 W (0.5 W)** resistor therefore provides a suitable power rating.

### Final LED Configuration

| Parameter             |   Value |
| --------------------- | ------: |
| IR LED                |       1 |
| Series resistor       |   490 Ω |
| Target LED current    | ≈ 20 mA |
| Resistor power rating |   1/2 W |
| Supply voltage        |    12 V |

---

# Step 4 — Build the Circuit

## Components

| Quantity | Component                   |
| -------: | --------------------------- |
|        2 | NE555 timer ICs             |
|        1 | IR LED                      |
|        2 | 1 kΩ resistors              |
|        1 | 6.8 kΩ resistor             |
|        1 | 20 kΩ potentiometer         |
|        1 | 10 µF capacitor             |
|        1 | 1 nF capacitor              |
|        1 | 490 Ω, 1/2 W resistor       |
|        1 | 12 V DC power supply        |
|        — | Breadboard and jumper wires |

The complete circuit schematic and final prototype can be added below:

```text
[Insert schematic image here]
```

```text
[Insert prototype image here]
```

---

# Step 5 — Test the Circuit

## 1. Check the Oscillation Frequencies

Connect the outputs of the two oscillators to an **oscilloscope**.

Verify that:

* The first oscillator generates approximately **10 Hz**.
* The second oscillator generates a carrier close to **38 kHz**.
* The potentiometer can be adjusted to obtain precisely **38 kHz**.

The 38 kHz carrier should appear as **short bursts controlled by the first oscillator**, with pauses between them.

Conceptually:

```text
10 Hz burst control:

    ┌───────┐       ┌───────┐       ┌───────┐
    │       │       │       │       │       │
────┘       └───────┘       └───────┘       └──

38 kHz carrier:

  /\/\/\/\/\          /\/\/\/\/\          /\/\/\/\/\
 /\/\/\/\/\/\        /\/\/\/\/\/\        /\/\/\/\/\/\
```

The result is a **38 kHz carrier transmitted in bursts**.

---

## 2. Check the IR LED

Point the IR LED toward a **smartphone camera**.

Most smartphone cameras can detect infrared light, allowing you to see the LED flashing when the emitter is operating.

The LED should appear to **flash periodically**, confirming that the infrared carrier is being transmitted in bursts.

> The smartphone camera confirms that the IR LED is emitting infrared light, but it does not verify that the carrier frequency is exactly 38 kHz. Use an oscilloscope for frequency verification.

---

# What's Next?

The same **38 kHz IR emitter** can be reused in several different receiver projects.

Possible applications include:

### Automatic Handwashing System

An IR beam barrier can detect the presence of a user's hands and activate the water flow automatically.

### Presence Detector

An IR-based system can detect when an object or person crosses the infrared beam.

### Remote-Controlled Lamp with a CD4017

An IR receiver can be combined with a **CD4017** to control a lamp remotely.

The advantage is that the **same 38 kHz IR emitter can be reused across all these projects**.

Only the receiver and processing circuit need to change depending on the application.

```text
                    38 kHz IR Emitter
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
       Handwashing     Presence      Remote Lamp
        System         Detector      + CD4017
```

---

# Project Summary

This project demonstrates how two simple **NE555 astable oscillators** can be combined to create a modulated infrared transmitter.

```text
10 Hz Oscillator
      │
      │ Burst Control
      ▼
38 kHz Oscillator
      │
      │ IR Carrier
      ▼
   IR LED
      │
      ▼
38 kHz IR Receiver
```

The first oscillator determines **when the IR signal is transmitted**, while the second oscillator determines the **carrier frequency**.

This separation makes the circuit simple to understand, calculate, build, and reuse with different IR receiver applications.
