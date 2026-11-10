# Basic Electronics Lab (BE Lab 3D)

A virtual electronics bench that runs entirely in your browser. You wire real circuits on a breadboard, drive them with a power supply and function generator, and measure them with an oscilloscope and a multimeter, all backed by a live circuit simulator. Wire something badly and the part burns out, just as it would on a real bench.

**Live demo: https://sinless-rush.github.io/electronics-lab/**

It is a single file, `index.html`, with no build step and no install.

## Purpose

Introductory electronics labs usually depend on scarce equipment, fragile parts and supervised lab time. This project gives students a place to practise the same experiments at home, on a phone or a laptop, and to make mistakes safely. It is meant to sit alongside a lab manual, not replace the bench: the guided experiments follow a standard first-year "Basic Electronics" syllabus (CRO measurements, Lissajous figures, diode characteristics, rectifiers, and BJT and JFET amplifiers).

What it aims to teach:

- How a breadboard is actually connected, and why two legs in one column are shorted.
- How to read an oscilloscope: volts/div, time/div, peak, period, frequency, phase.
- How real parts fail: overheated resistors, LEDs without series resistors, reversed electrolytic capacitors, overloaded transistors.
- What the numbers in a lab manual should look like (ripple factors, bias voltages, gains), so you can tell when your own readings are off.

## Quick start

### Use it online

Open https://sinless-rush.github.io/electronics-lab/ in your browser. Nothing to download or install. Pick an experiment from the dropdown in the header, or stay on **Free bench**.

### Run it on your own computer

1. Get the files, using either method:
   - **Download:** on the repository page, choose **Code → Download ZIP**, then unzip it.
   - **Clone:** `git clone https://github.com/sinless-rush/electronics-lab.git`
2. Open the folder and double-click `index.html`. It opens in your default browser, with no install or build step.
3. Pick an experiment from the dropdown in the header, or stay on **Free bench**.

If the page opens as plain text or the 3D bench stays blank, start a small local server in the project folder and open the address it prints:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000` in your browser.

Requirements:

- A recent version of Chrome, Firefox, Safari or Edge, with WebGL enabled. Phones and tablets work too.
- An internet connection when the page loads, because Three.js r128 is fetched from cdnjs. Everything else is inline.

## Overview of features

### 3D bench

- A 3D lab scene with the breadboard, oscilloscope, DC power supply, function generator, multimeter and a centre-tapped transformer.
- Orbit, pan and zoom with mouse or touch. Pinch to zoom, use the **Orbit/Pan** button on touch screens, and **Reset view** to come back.
- Hover over holes, wires, parts or equipment for a description. Click any instrument for an info card that explains what it does and how to use it.
- LEDs glow in proportion to the current through them. Burnt parts turn black and smoke. The breadboard tints when the supply is shorted.

### Breadboard wiring panel

- A 30-column breadboard with five-hole strips (a to e, f to j) and power rails, plus ten front-panel terminals: PSU +, FG +, CH1, CH2, DMM V, two GND, FG2 +, and the transformer ends AC1 and AC2.
- Click two points to place a wire or a two-leg part. Transistors take three clicks (collector, base, emitter, or drain, gate, source) and a hint bar tells you which leg comes next.
- Parts must span two different strips. Placing both legs in one strip is rejected with an explanation.
- A **Remove** tool deletes wires and parts, and **Clear bench** starts over.

### Parts

| Part | Options |
| --- | --- |
| Wire | Six rotating colours |
| Resistor | 100 Ω to 470 kΩ (12 standard values), rated 0.25 W |
| Capacitor | 33 nF, 100 nF, 1 µF, 10 µF, 100 µF; the first hole clicked is the + leg |
| LED | Red, 30 mA limit |
| Diode | 1N4001-style rectifier diode |
| Zener | 5.1 V or 12 V, 1.3 W |
| BJT | NPN transistor |
| JFET | N-channel, IDSS 5.5 mA, VP −2.2 V |

### Instruments

- **DC power supply:** 0 to 30 V, adjustable current limit from 0.01 to 3 A, output on/off. It switches into constant-current mode when the limit is reached and warns you about a short circuit.
- **Function generator:** sine, square and triangle at 1 Hz to 200 kHz, up to 20 V peak, with a 50 Ω output resistance. A second sine output (FG2) with its own frequency and amplitude supports Lissajous work.
- **Oscilloscope (CRO):** two channels, 10 mV to 20 V per division, 20 µs to 100 ms per division, and an X-Y mode. Readouts under the screen give mean, ripple factor, Vpp, Vp, Vrms (sine) and frequency.
- **Digital multimeter:** DC V and AC V (RMS of the varying part), which together give the ripple of a rectifier output. Current measurement is not available yet.
- **Transformer:** a centre-tapped 11-0-11 V, 50 Hz source for rectifier experiments (about ±15.6 V peak at each end).

### Guided experiments

Each experiment builds its circuit for you, sets the instruments, and shows numbered steps with expected values.

| Experiment | What you do |
| --- | --- |
| Free bench | Build anything; try to break things on purpose |
| 1A | Measure DC, peak, RMS, period and frequency of a signal on the CRO |
| 1B | Lissajous patterns and finding an unknown frequency |
| 1C | Phase angle from the X-Y ellipse, using an R-C network |
| 2 | Forward-bias characteristics of a PN junction diode |
| 4 | Full-wave rectifier: ripple factor, filter capacitor, half-wave comparison |
| 5 | BJT common-emitter amplifier: bias point, gain, phase inversion, clipping |
| 6 | JFET common-source amplifier: self-bias and gain |

### Failure simulation

Parts are checked every simulation frame and fail when they exceed their ratings. A message explains what went wrong and how to avoid it.

| Part | Fails when |
| --- | --- |
| Resistor | Dissipates more than 0.25 W |
| LED | Carries more than 30 mA |
| Diode | Carries more than 1 A |
| Zener | Dissipates more than 1.3 W |
| Electrolytic capacitor (1 µF and up) | Connected backwards, or charged past 25 V |
| BJT / JFET | Carries more than 150 mA |

Press **Replace burnt parts** to reset them without rebuilding the circuit.

## How it works

The simulator is written from scratch in plain JavaScript:

- **Nodal analysis.** The breadboard is reduced to electrical nodes and solved as a linear system with Gaussian elimination and partial pivoting.
- **Newton iteration** handles the non-linear devices: diodes, LEDs, Zeners (with a breakdown term), an Ebers-Moll style BJT and a square-law JFET with channel-length modulation.
- **Backward-Euler integration** models capacitors, so charging, filtering and AC coupling behave correctly over time.
- **Real-source behaviour.** The supply has current limiting and the function generator has a 50 Ω output, so loaded circuits differ slightly from ideal textbook values, as they would on a real bench.
- Each frame solves 1200 time steps spanning the scope's time base, then draws the trace on a canvas. That canvas is also used as a texture on the 3D oscilloscope screen.

Rendering uses Three.js for the 3D scene, SVG for the wiring panel, and Canvas 2D for the scope and the instrument displays.

## Interface and accessibility

- Responsive layout: two columns on wide screens, a single stacked column on phones, with a horizontally scrollable wiring panel on narrow screens.
- Automatic light and dark themes, following the system setting.
- Larger touch targets on touch screens, and animations are switched off when the system asks for reduced motion.

## Known limitations

- The BJT has no junction capacitance, so high-frequency roll-off (fH) is not simulated.
- A second adjustable supply (dual supply) is not available, so the BJT and JFET output and transfer curves from the manuals cannot be measured yet.
- The multimeter measures voltage only.
- Component values are limited to the lists in the part selector.

## Project structure

```
index.html   # the entire app: styles, markup and JavaScript
```

Inside `index.html` the script is organised in labelled sections: circuit solver, oscilloscope, breadboard wiring panel, instrument controls, guided experiments, and the 3D bench.

## Contributing

Issues and pull requests are welcome. Good places to start are the items under Known limitations, new guided experiments (add an entry to the `GD` object), and new part types. Because there is no build step, you can test a change by refreshing the page in your browser.

## License

No license has been chosen yet. Add a `LICENSE` file to state how others may use this project.
