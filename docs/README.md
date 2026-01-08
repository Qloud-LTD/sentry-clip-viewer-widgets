# Widget SDK Documentation

## Overview

The declarative widget system allows you to create custom telemetry overlays using JSON definitions. Unlike SwiftUI widgets (which require compiled code), declarative widgets are defined entirely in JSON and loaded at runtime.

### Key Differences from SwiftUI

| Aspect | SwiftUI Widgets | Declarative Widgets |
|--------|-----------------|---------------------|
| Definition | Swift code | JSON files |
| Compilation | Required | None (runtime loading) |
| Distribution | App update | Widget packs (bundled or external) |
| Complexity | Full programming | Limited, safe expressions |
| App Store | Full review | Sandboxed, non-Turing-complete |

### Benefits

- **Community Packs**: Share and distribute widget collections without app updates
- **Easy to Create**: No Swift knowledge required - just JSON
- **Safe Execution**: Expression parser is intentionally limited for App Store compliance
- **Live Preview**: Changes visible immediately in the overlay editor
- **Composable**: Widgets can embed other widgets for complex layouts

---

## Quick Start

### 1. Widget JSON Structure

Every widget requires these core properties:

```json
{
  "$schema": "widget-v1.json",
  "id": "mypack.speedometer",
  "version": "1.0.0",
  "category": "speed",
  "name": "My Speedometer",
  "canvas": {
    "minWidth": 0.10,
    "minHeight": 0.10
  },
  "elements": [
    { "type": "text", "content": "{{speed.formatted}}" }
  ]
}
```

### 2. Simple Example Widget

A digital speed display with unit:

```json
{
  "$schema": "widget-v1.json",
  "id": "example.speedWithUnit",
  "version": "1.0.0",
  "category": "speed",
  "name": "Speed with Unit",
  "description": "Shows speed value with unit label",
  "author": "Your Name",

  "canvas": {
    "minWidth": 0.08,
    "minHeight": 0.06
  },

  "elements": [
    {
      "type": "group",
      "layout": "vstack",
      "spacing": "2%h",
      "alignment": "center",
      "children": [
        {
          "type": "text",
          "content": "{{speed.formatted ?? '--'}}",
          "style": {
            "fontSize": "50%h",
            "fontWeight": "bold",
            "color": "#FFFFFF"
          }
        },
        {
          "type": "text",
          "content": "{{speed.unit}}",
          "style": {
            "fontSize": "20%h",
            "color": "rgba(255,255,255,0.7)"
          }
        }
      ]
    }
  ]
}
```

### 3. How Widgets Are Loaded

1. **App Startup**: `WidgetPackManager` scans `Resources/WidgetPacks/` for manifest files
2. **Manifest Parsing**: Each `manifest.json` declares which widgets are in the pack
3. **On-Demand Loading**: Individual widget JSON files are parsed when needed
4. **Validation**: `WidgetParser` validates structure and binding expressions
5. **Rendering**: `DeclarativeWidgetRenderer` converts definitions to SwiftUI views

---

## Widget Pack Structure

Widget packs organize related widgets together:

```
Resources/WidgetPacks/
├── core-widgets/
│   ├── manifest.json          # Pack manifest (required)
│   ├── speed/
│   │   ├── digital.json       # Widget definition
│   │   ├── analog.json
│   │   └── compact.json
│   ├── gear/
│   │   ├── letter.json
│   │   └── colored.json
│   ├── autopilot/
│   │   └── dot.json
│   └── compound/
│       └── tesla-hud.json     # Compound widget using refs
├── racing-pack/
│   ├── manifest.json
│   └── ...
```

### Manifest File

```json
{
  "$schema": "widget-pack-v1.json",
  "id": "core-widgets",
  "version": "1.0.0",
  "name": "Core Widgets",
  "description": "Built-in widget collection",
  "author": "Sentry Clip Viewer",

  "compatibility": {
    "minAppVersion": "3.0.0",
    "schemaVersion": "1.0"
  },

  "widgets": [
    { "file": "speed/digital.json", "category": "speed" },
    { "file": "speed/analog.json", "category": "speed" },
    { "file": "gear/letter.json", "category": "gear" },
    { "file": "compound/tesla-hud.json", "category": "compound" }
  ],

  "presets": []
}
```

---

## Key Concepts

### Canvas Sizing

The canvas defines the widget's size constraints:

```json
"canvas": {
  "aspectRatio": 1.0,      // Width/height ratio (1.0 = square)
  "minWidth": 0.10,        // Minimum 10% of parent width
  "minHeight": 0.10,       // Minimum 10% of parent height
  "maxWidth": 0.50,        // Maximum 50% of parent width
  "backgroundColor": "#000000",
  "cornerRadius": "5%"
}
```

- Values are fractions (0.0-1.0) representing percentage of parent container
- Use `aspectRatio` for fixed proportions (gauges, circles)
- Omit constraints for flexible sizing

### Element Types

| Type | Description | Key Properties |
|------|-------------|----------------|
| `text` | Text display | `content`, `style` |
| `icon` | SF Symbol | `name`, `color`, `rotation` |
| `rect` | Rectangle | `fill`, `stroke`, `cornerRadius` |
| `circle` | Circle | `radius`, `fill`, `stroke` |
| `arc` | Arc segment | `startAngle`, `endAngle`, `radius` |
| `path` | SVG-like path | `d` (path data) |
| `group` | Container | `layout`, `children`, `spacing` |
| `widget` | Embed widget | `ref` (widget ID) |

### Bindings

Bindings connect widget elements to live telemetry data using `{{path}}` syntax:

```json
"content": "{{speed.formatted}}"
"color": "{{autopilot.color}}"
"rotation": "{{steering.angle}}"
"visible": "{{brake.applied}}"
```

**Null Coalescing:**
```json
"content": "{{speed.formatted ?? '--'}}"
```

**Common Binding Paths:**

| Category | Path | Type | Description |
|----------|------|------|-------------|
| Speed | `speed.value` | number | Raw speed in m/s |
| | `speed.formatted` | string | e.g., "65" |
| | `speed.withUnit` | string | e.g., "65 mph" |
| | `speed.unit` | string | "mph" or "km/h" |
| Throttle | `throttle.percent` | number | 0-100 |
| | `brake.applied` | boolean | true when braking |
| Steering | `steering.angle` | number | Degrees (-450 to 450) |
| | `steering.formatted` | string | e.g., "-12°" |
| Autopilot | `autopilot.state` | string | "fsd", "autosteer", "tacc", etc. |
| | `autopilot.color` | string | Hex color for state |
| | `autopilot.engaged` | boolean | Any autopilot active |
| Gear | `gear.letter` | string | "P", "R", "N", "D" |
| | `gear.color` | string | Hex color for gear |
| Heading | `heading.degrees` | number | 0-360 |
| | `heading.cardinal` | string | "N", "NE", "E", etc. |
| G-Force | `gforce.x` | number | Lateral G |
| | `gforce.y` | number | Longitudinal G |
| Turn Signals | `blinker.left` | boolean | Left signal active |
| | `blinker.right` | boolean | Right signal active |
| Battery | `battery.percent` | number | 0-100 |
| | `battery.range` | number | Estimated range |
| Temperature | `temp.inside` | number | Interior temp |
| | `temp.outside` | number | Exterior temp |
| Location | `location.lat` | number | Latitude |
| | `location.lon` | number | Longitude |
| | `location.name` | string | Reverse geocoded address |
| Timestamp | `timestamp.time` | string | "HH:mm:ss" |
| | `timestamp.date` | string | "yyyy-MM-dd" |

### Conditional Values

For dynamic styling based on conditions:

```json
"bindings": {
  "dotColor": {
    "conditions": [
      { "when": "{{autopilot.state == 'fsd'}}", "value": "#0066FF" },
      { "when": "{{autopilot.state == 'autosteer'}}", "value": "#00CC00" },
      { "when": "{{autopilot.state == 'tacc'}}", "value": "#FF9900" }
    ],
    "default": "#FFFFFF"
  }
}
```

Then reference: `"fill": "{{dotColor}}"`

### Percentage-Based Sizing

Dimensions support multiple unit types:

| Suffix | Meaning | Example |
|--------|---------|---------|
| `%h` | Percentage of height | `"60%h"` = 60% of container height |
| `%w` | Percentage of width | `"30%w"` = 30% of container width |
| `%` | Percentage of min(width, height) | `"50%"` = 50% of smaller dimension |
| `px` | Pixels (use sparingly) | `"16px"` = 16 pixels |

Examples:
```json
"fontSize": "60%h"       // Font scales with height
"radius": "45%"          // Circle scales proportionally
"spacing": "4%w"         // Spacing relative to width
"strokeWidth": "2px"     // Fixed 2-pixel stroke
```

### Expressions

The expression parser supports limited operations for safety:

**Arithmetic:**
```json
"{{speed.value * 2.237}}"     // m/s to mph
"{{heading.degrees + 180}}"   // Flip heading
```

**Comparison:**
```json
"{{speed.value > 100}}"       // Boolean result
"{{gear.letter == 'D'}}"      // String comparison
```

**Ternary:**
```json
"{{brake.applied ? 'B' : throttle.formatted}}"
```

**Functions:**
```json
"{{abs(steering.angle)}}"
"{{round(speed.value, 1)}}"
"{{clamp(throttle.percent, 0, 100)}}"
"{{min(speed.value, 200)}}"
"{{max(gforce.x, gforce.y)}}"
```

**Filters (pipe syntax):**
```json
"{{speed.value | round:0}}"           // Round to integer
"{{steering.angle | abs}}"            // Absolute value
"{{speed.value | clamp:0,200}}"       // Clamp to range
"{{steering.angle | sign}}"           // Add +/- prefix
"{{heading.cardinal | uppercase}}"    // "NE" -> "NE"
```

### Visibility Control

Elements can be conditionally shown:

```json
{
  "type": "icon",
  "name": "exclamationmark.triangle",
  "visible": "{{brake.applied}}",
  "color": "#FF0000"
}
```

### Embedding Widgets

Compound widgets can embed other widgets:

```json
{
  "type": "widget",
  "ref": "speed.speedDigital",
  "size": { "width": "30%", "height": "100%" },
  "optionOverrides": {
    "speedUnit": "mph"
  }
}
```

### Groups and Layouts

Groups organize child elements:

```json
{
  "type": "group",
  "layout": "hstack",        // "hstack", "vstack", or "zstack"
  "spacing": "4%w",          // Space between children
  "alignment": "center",     // "leading", "center", "trailing", etc.
  "children": [
    { "type": "text", "content": "Speed:" },
    { "type": "text", "content": "{{speed.formatted}}" }
  ]
}
```

---

## Complete Example: Analog Gauge

```json
{
  "$schema": "widget-v1.json",
  "id": "speed.analog",
  "version": "1.0.0",
  "category": "speed",
  "name": "Analog Gauge",
  "description": "Circular speedometer with needle",

  "canvas": {
    "aspectRatio": 1.0,
    "minWidth": 0.10,
    "minHeight": 0.10
  },

  "options": {
    "maxSpeed": {
      "type": "number",
      "default": 160,
      "label": "Maximum Speed"
    }
  },

  "elements": [
    {
      "type": "arc",
      "center": { "x": "50%", "y": "50%" },
      "radius": "46%",
      "startAngle": 135,
      "endAngle": 405,
      "stroke": "rgba(255,255,255,0.3)",
      "strokeWidth": "2%"
    },
    {
      "type": "circle",
      "position": { "x": "50%", "y": "50%" },
      "radius": "5%",
      "fill": "#FFFFFF"
    },
    {
      "type": "text",
      "content": "{{speed.formatted ?? '--'}}",
      "position": { "x": "50%", "y": "72%" },
      "anchor": "center",
      "style": {
        "fontSize": "15%h",
        "fontWeight": "bold",
        "color": "#FFFFFF"
      }
    },
    {
      "type": "text",
      "content": "{{speed.unit}}",
      "position": { "x": "50%", "y": "82%" },
      "anchor": "center",
      "style": {
        "fontSize": "8%h",
        "color": "rgba(255,255,255,0.7)"
      }
    }
  ]
}
```

---

## Style Properties

### Text Styling

```json
"style": {
  "fontSize": "60%h",
  "fontWeight": "bold",           // ultralight, thin, light, regular, medium, semibold, bold, heavy, black
  "fontDesign": "rounded",        // default, rounded, monospaced, serif
  "color": "#FFFFFF",
  "textAlignment": "center",      // leading, center, trailing
  "minimumScaleFactor": 0.5,      // Allow text shrinking
  "lineLimit": 1
}
```

### Shape Styling

```json
"fill": "#FF0000",
"stroke": "#FFFFFF",
"strokeWidth": "2%",
"cornerRadius": "10%"
```

### Effects

```json
"style": {
  "opacity": 0.8,
  "blur": 2.0,
  "rotation": "45",
  "scale": 1.5,
  "shadow": {
    "color": "#000000",
    "radius": "4",
    "x": "2",
    "y": "2"
  }
}
```

### Colors

Supported formats:
- Named: `"white"`, `"red"`, `"blue"`, etc.
- Hex: `"#FF0000"`, `"#FF0000FF"` (with alpha)
- RGB: `"rgb(255, 0, 0)"`
- RGBA: `"rgba(255, 0, 0, 0.5)"`

---

## Widget Options

Widgets can define configurable options:

```json
"options": {
  "maxSpeed": {
    "type": "number",
    "default": 160,
    "label": "Maximum Speed",
    "min": 50,
    "max": 300
  },
  "showUnit": {
    "type": "boolean",
    "default": true,
    "label": "Show Unit"
  },
  "style": {
    "type": "enum",
    "default": "digital",
    "label": "Display Style",
    "values": ["digital", "analog", "minimal"]
  },
  "accentColor": {
    "type": "color",
    "default": "#FF0000",
    "label": "Accent Color"
  }
}
```

Access options in bindings: `"{{options.maxSpeed}}"`

---

## Security Limits

The expression parser enforces these limits:

| Limit | Value |
|-------|-------|
| Max expression length | 500 characters |
| Max nesting depth | 10 levels |
| Max operators | 20 per expression |

Only whitelisted functions are allowed: `min`, `max`, `abs`, `round`, `floor`, `ceil`, `clamp`.

---

## Related Documentation

- [SCHEMA.md](./SCHEMA.md) - Complete JSON schema reference
- [BINDINGS.md](./BINDINGS.md) - All available data bindings
- [EXPRESSIONS.md](./EXPRESSIONS.md) - Expression syntax and operators
- [ELEMENTS.md](./ELEMENTS.md) - Element type reference
- [EXAMPLES.md](./EXAMPLES.md) - Widget examples and patterns

---

## File Locations

| File | Purpose |
|------|---------|
| `Services/DeclarativeWidgets/Schema/WidgetDefinition.swift` | Core data models |
| `Services/DeclarativeWidgets/Schema/ElementDefinition.swift` | Element types |
| `Services/DeclarativeWidgets/Schema/BindingExpression.swift` | Binding paths |
| `Services/DeclarativeWidgets/Parser/WidgetParser.swift` | JSON parsing |
| `Services/DeclarativeWidgets/Parser/ExpressionParser.swift` | Expression evaluation |
| `Services/DeclarativeWidgets/Renderer/DeclarativeWidgetRenderer.swift` | SwiftUI rendering |
| `Services/WidgetPacks/WidgetPackManager.swift` | Pack management |
| `Resources/WidgetPacks/` | Widget JSON files |
