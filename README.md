# Keyling protocol

The protocol a phone uses to drive a Keyling dongle over Bluetooth LE. `bridge.proto` is the
contract: its comments describe the GATT service, framing, flow control, abort and every message.

- `bridge.proto`: the schema (proto3).
- `bridge.options`: nanopb size limits, used by the firmware's C code generation.
- `design/tokens.json`: the shared look (see below).

## Using it

Projects include this repository as a git submodule and generate code from it:

```sh
git submodule add https://github.com/keyling-hq/protocol.git proto
git clone --recurse-submodules <project>          # or: git submodule update --init
git submodule update --remote proto               # pick up a newer protocol, then regenerate
```

The submodule pins the exact protocol commit each project was built against.

## Compatibility rules

- Fields and messages are only ever added; numbers are never reused or renumbered.
- A dongle answers requests it doesn't support with `RESULT_INVALID`; clients check
  `Hello.capabilities` before using a request family.
- `oneof` field numbers are grouped: 1-9 core, 10-29 input, 30-49 system, 50+ on-device apps.
- `Hello.protocol_version` changes only for incompatible changes to framing or flow control.

## Design tokens

`design/tokens.json` holds the values the dongle's Candy theme and the hub app share, so both
look like one product: the palette, the color of each on-device app (by the app id the dongle
reports), the status LED's colors, corner radii and the fonts. It uses the
[Design Tokens Format](https://www.designtokens.org/) (`$value`, `$type`; colors as sRGB hex),
which tools such as Style Dictionary read.

Projects generate their constants from it rather than copying values: the firmware's
`tools/gen_tokens.py` writes C headers, and its CI fails when they are out of date. Tokens are
only added or changed together with both users; a renamed or removed token breaks their builds.

## License

MIT, see `LICENSE`. Clients for any password manager or app, open or closed source, may implement
this protocol. The firmware and Android app are GPL-3.0-only.
