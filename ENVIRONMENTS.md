# Reference environments

Every measured number in this project names the environment it came from.
Numbers from different environments are not comparable.

| Id | Specification | Role |
|---|---|---|
| **E-M** | WSL2 Ubuntu on Windows 10 Pro 19045 · GTX 1660 Ti 6 GiB (driver 610.88) · Python 3.12.13 · TensorFlow 2.21.0 (GPU) · torch 2.11.0+cu128 · grpcio 1.83.0 | Primary measurement environment. All headline server figures were measured here, on one server with no restarts and all three models resident. |
| **E-D** | Windows Pro (build 26200) · i7-9750H · 16 GiB RAM · GTX 1650 4 GiB (driver 595.97) · Python 3.11.9 · torch 2.6.0+cu124 · **no TensorFlow** | Development and unit tests. Also the environment that proved the registry tolerates a missing framework, and that a 4 GiB card will not hold two halves sessions. |
| **E-A** | Team Android handset | Client on real hardware. The capture probes compile but have not been run — zero devices measured. |
| **E-U** | Windows 11 · Gradle 8.13 · AGP 8.13.2 · JBR 21 · Robolectric 4.16 @ SDK 35 | Client unit tests on the JVM. All mobile figures came from here. |

## Reading these correctly

**E-U measures logic, not hardware.** Robolectric substitutes the Android
framework, so its camera and microphone are stand-ins. Anything that depends
on what a real device actually delivers — plane strides, supported sample
rates, real frame cadence — cannot be settled there, and is marked
unmeasured rather than assumed.

**E-A has no results yet.** The device matrix is empty by design, not by
omission. When a device is run, its row belongs here.
