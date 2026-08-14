# Reproducing the results

## Prerequisites

- An NVIDIA GPU with at least 6 GiB. A 4 GiB card will not hold two
  concurrent halves sessions.
- TensorFlow with accelerator support. On Windows this means WSL2 — the
  primary measurement host runs the server inside Ubuntu under WSL2 while the
  phone connects across the LAN.
- The three repositories cloned **side by side** — the backend currently
  reaches the multisensory code as a sibling directory.
- Checkpoints downloaded, placed, and hash-verified. See
  [CHECKPOINTS.md](CHECKPOINTS.md).

## Running

1. Start the server. It binds `:50051` in plaintext on the local network.
2. Confirm the health check reports the models you expect. If one engine
   failed to load, the server still starts and says so — that is deliberate,
   not a silent failure.
3. On the phone, set the server address in-app. It persists across runs.
4. Pick a mode and start a session.

## Reproducing the measurements

The load and stress harness is built from the project's own generated stubs,
not a general-purpose tool, so it measures the server through its real
protocol with no internal injection points. Every run records a code
fingerprint and the resolved YAML targets alongside its results.

Two things to hold constant if you want comparable numbers:

- **One server process, no restarts across the campaign.** GPU residency
  climbs through a session series; restarting resets it and makes the memory
  figures meaningless.
- **The same environment.** Numbers from E-M and E-D are not interchangeable.
  See [ENVIRONMENTS.md](ENVIRONMENTS.md).

## The determinism check

The deterministic replay harness pushes a fixed clip through the wire protocol
and compares separation against a no-streaming reference. It exists so that
refactoring can be proven not to have changed the audio path: matching to four
decimal places across a loop restructure is what licensed that change.

If you refactor anything on the audio path, capture replay output before and
after and diff it. A difference in the last decimal place is a finding, not
noise to be tolerated.
