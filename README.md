# EFR32 Touchlink Reset

A standalone Python utility for resetting one nearby Zigbee Light Link (ZLL) / Touchlink device through a Silicon Labs EFR32 / EmberZNet coordinator.

It is designed as a workaround for cases where normal Zigbee2MQTT Touchlink scanning does not receive Scan Responses from an EFR32 coordinator.

## What it does

The script uses Ember MFG library mode to receive raw Touchlink Scan Responses and performs this targeted sequence for the first matching device:

```text
Scan Request -> Scan Response -> Identify Request -> wait 2 seconds
-> Reset-to-Factory-New Request
```

The reset is an IEEE-addressed inter-PAN unicast. Scan, Identify, and Reset reuse the same random transaction ID.

## Requirements

- Python 3.10 or later
- A Touchlink-capable EFR32 / EmberZNet coordinator reachable via EZSP serial or TCP
- `bellows`

Install the only Python dependency:

```sh
python3 -m pip install bellows
```

## Before running

1. Stop Zigbee2MQTT, ZHA, or any other process using the coordinator.
2. Verify the coordinator has exactly one owner: this script.
3. Put the intended light within approximately 10 cm of the coordinator.
4. Turn off or move other Touchlink-capable devices out of range.
5. Use a known Touchlink channel whenever possible. Scanning every channel resets the first responder, which is destructive.

## Configuration

Edit the user-default block at the top of `touchlink_reset.py` when needed:

- `DEFAULT_DEVICE`: EZSP TCP or serial endpoint
- `DEFAULT_BAUDRATE`: serial baud rate
- `DEFAULT_SCAN_WAIT_SECONDS`: response timeout per channel

The validated example for an MR5U endpoint was:

```sh
python3 touchlink_reset.py --channel 20
```

Use another endpoint when required:

```sh
python3 touchlink_reset.py tcp://COORDINATOR_IP:6638 --channel 20
```

## After the reset

1. Start Zigbee2MQTT again.
2. Enable Permit Join.
3. Power-cycle the light if it does not join immediately.

Some devices indicate pairing for only a short time after power is restored. A successful rejoin is the practical confirmation that the reset succeeded.

## Notes and limitations

- This is destructive: it erases the target's Zigbee network state.
- The NCP accepting an outbound frame does not prove that the target accepted it. Check the visible Identify/reset behavior and rejoin.
- The script resets the first valid Touchlink Scan Response on the selected channel. It is not an interactive multi-device selection tool.
- The protocol frame serializer is included in the script, so it does not depend on project-local helper files.

## License and attribution

This project is licensed under GPL-3.0-or-later. The raw legacy ZLL inter-PAN frame structure is derived from the GPL-licensed `hue-thief-skyconnect` project by John Neerdael and related upstream work. See `LICENSE`.

Generated and validated on 2026-07-12 with GPT-5.6-terra.
