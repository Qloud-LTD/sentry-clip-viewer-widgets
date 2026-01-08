# Sentry Clip Viewer - Widget Packs

Declarative widget definitions for [Sentry Clip Viewer](https://apps.apple.com/app/sentry-clip-viewer), the Tesla dashcam viewer app for iOS and Mac.

## Overview

This repository contains:
- **Widget SDK Documentation** - Complete reference for creating custom widgets
- **Core Widgets Pack** - 77 built-in widgets included with the app
- **Example Widgets** - Standalone examples to learn from

## Widget SDK Documentation

| Document | Description |
|----------|-------------|
| [README](docs/README.md) | Overview & quick start guide |
| [SCHEMA](docs/SCHEMA.md) | Complete JSON schema reference |
| [BINDINGS](docs/BINDINGS.md) | Data binding reference (telemetry paths) |
| [EXPRESSIONS](docs/EXPRESSIONS.md) | Expression syntax & operators |
| [ELEMENTS](docs/ELEMENTS.md) | Element type reference |
| [EXAMPLES](docs/EXAMPLES.md) | Example gallery with explanations |

## Core Widgets Pack

The `packs/core-widgets/` directory contains all 77 built-in widgets organized by category:

| Category | Widgets | Description |
|----------|---------|-------------|
| speed | 7 | Digital, analog, compact speed displays |
| gear | 4 | Gear indicator variants |
| throttle | 9 | Throttle/brake bars and pedals |
| steering | 4 | Steering wheel with angle |
| autopilot | 4 | Autopilot status indicators |
| compass | 4 | Heading/direction displays |
| turn-signal | 4 | Blinker indicators |
| location | 5 | GPS location displays |
| gforce | 5 | G-force meters |
| map | 6 | Map widget variants |
| battery | 5 | Battery level displays |
| temperature | 4 | Temperature displays |
| timestamp | 5 | Date/time displays |
| watermark | 1 | Custom text watermark |
| compound | 10 | HUD layouts combining multiple widgets |

## Quick Start

### Widget Structure

```json
{
  "$schema": "widget-v1.json",
  "id": "my-pack.my-widget",
  "version": "1.0.0",
  "category": "speed",
  "name": "My Speed Widget",
  "canvas": { "minWidth": 0.06, "minHeight": 0.06 },
  "elements": [
    {
      "type": "text",
      "content": "{{speed.formatted}}",
      "position": { "x": "50%", "y": "50%" },
      "anchor": "center",
      "style": {
        "fontSize": "60%h",
        "fontWeight": "bold",
        "color": "#FFFFFF"
      }
    }
  ]
}
```

### Element Types

- `text` - Display text with data bindings
- `icon` - SF Symbols (Apple's system icons)
- `rect` - Rectangle shapes
- `circle` - Circle shapes
- `arc` - Arc/gauge shapes
- `line` - Line segments
- `path` - SVG-like paths
- `group` - Container with layout (hstack/vstack/zstack)
- `widget` - Embed another widget

### Data Bindings

Use `{{path}}` syntax to bind telemetry data:

- `{{speed.formatted}}` - Speed value (e.g., "65")
- `{{speed.withUnit}}` - Speed with unit (e.g., "65 mph")
- `{{gear.letter}}` - Gear (P/R/N/D)
- `{{steering.angle}}` - Steering angle in degrees
- `{{autopilot.state}}` - Autopilot mode
- `{{gforce.x}}`, `{{gforce.y}}` - G-force values

See [BINDINGS.md](docs/BINDINGS.md) for the complete list.

## Releases

Widget pack updates are published as [GitHub Releases](../../releases). Each release includes:
- Version tag (e.g., `v1.0.0`)
- Changelog
- Downloadable `core-widgets.zip`

## License

MIT License - See [LICENSE](LICENSE) for details.

## Contributing

Contributions are welcome! Please read the SDK documentation before submitting widgets.

1. Fork this repository
2. Create your widget in a new branch
3. Test with the app (if possible)
4. Submit a Pull Request

## Related

- [Sentry Clip Viewer on App Store](https://apps.apple.com/app/sentry-clip-viewer)
- [Main App Repository](https://github.com/sioakim/sentryclipviewer) (private)
