# Widget Definition Schema Reference

This document provides the complete JSON schema reference for creating declarative widgets in Sentry Clip Viewer.

## Table of Contents

- [Overview](#overview)
- [Widget Definition](#widget-definition)
- [Canvas Configuration](#canvas-configuration)
- [Options Definition](#options-definition)
- [Bindings Section](#bindings-section)
- [Elements Array](#elements-array)
- [Element Types](#element-types)
- [Style Properties](#style-properties)
- [Dimension Values](#dimension-values)
- [Color Values](#color-values)

---

## Overview

Widgets are defined in JSON files with a `.widget.json` extension. Each widget is self-contained and declarative, meaning the JSON describes what to render rather than how to render it.

### Minimal Example

```json
{
  "$schema": "widget-schema-v1",
  "id": "my-widget.simple",
  "version": "1.0.0",
  "category": "custom",
  "name": "Simple Speed",
  "canvas": {
    "aspectRatio": 2.0
  },
  "elements": [
    {
      "type": "text",
      "content": "{{speed.formatted}}",
      "style": {
        "fontSize": "50%h",
        "fontWeight": "bold",
        "color": "white"
      }
    }
  ]
}
```

---

## Widget Definition

The root object of a widget JSON file.

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `$schema` | `string` | No | Schema version identifier (e.g., `"widget-schema-v1"`) |
| `id` | `string` | Yes | Unique widget identifier using dot notation (e.g., `"speed.digital"`) |
| `version` | `string` | Yes | Semantic version (e.g., `"1.0.0"` or `"1.0"`) |
| `category` | `string` | Yes | Widget category for organization (e.g., `"speed"`, `"navigation"`, `"compound"`) |
| `name` | `string` | Yes | Human-readable display name |
| `description` | `string` | No | Brief description of the widget |
| `author` | `string` | No | Widget author name or identifier |
| `canvas` | `object` | Yes | Canvas configuration (see below) |
| `options` | `object` | No | Configurable options map (see below) |
| `bindings` | `object` | No | Pre-computed bindings for optimization |
| `elements` | `array` | Yes | Array of element definitions |

### ID Naming Convention

Widget IDs should follow a hierarchical naming pattern:
- `category.variant` - e.g., `speed.digital`, `speed.gauge`, `speed.minimal`
- `category.subcategory.variant` - e.g., `navigation.compass.simple`

---

## Canvas Configuration

Defines the widget's container and sizing behavior.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `aspectRatio` | `number` | `null` | Fixed width/height ratio (e.g., `2.0` = twice as wide as tall) |
| `minWidth` | `number` | `null` | Minimum width as percentage of parent (0.0-1.0) |
| `minHeight` | `number` | `null` | Minimum height as percentage of parent (0.0-1.0) |
| `maxWidth` | `number` | `null` | Maximum width as percentage of parent (0.0-1.0) |
| `maxHeight` | `number` | `null` | Maximum height as percentage of parent (0.0-1.0) |
| `backgroundColor` | `string` | `null` | Background color (hex, named, or binding) |
| `cornerRadius` | `string` | `null` | Corner radius for the canvas background |

### Example

```json
{
  "canvas": {
    "aspectRatio": 1.5,
    "minWidth": 0.15,
    "minHeight": 0.1,
    "backgroundColor": "rgba(0, 0, 0, 0.5)",
    "cornerRadius": "8px"
  }
}
```

---

## Options Definition

Define user-configurable options for your widget.

### Option Object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `type` | `string` | Yes | One of: `"string"`, `"number"`, `"boolean"`, `"enum"`, `"color"` |
| `default` | `any` | No | Default value for the option |
| `label` | `string` | No | Human-readable label for UI |
| `values` | `array` | For enum | Array of valid values for enum type |
| `min` | `number` | No | Minimum value for number type |
| `max` | `number` | No | Maximum value for number type |

### Option Types

| Type | Description | Example Default |
|------|-------------|-----------------|
| `string` | Text value | `"MPH"` |
| `number` | Numeric value | `100` |
| `boolean` | True/false toggle | `true` |
| `enum` | Selection from list | `"standard"` |
| `color` | Color picker | `"#FFFFFF"` |

### Example

```json
{
  "options": {
    "showUnit": {
      "type": "boolean",
      "default": true,
      "label": "Show Speed Unit"
    },
    "accentColor": {
      "type": "color",
      "default": "#00FF00",
      "label": "Accent Color"
    },
    "displayMode": {
      "type": "enum",
      "values": ["minimal", "standard", "detailed"],
      "default": "standard",
      "label": "Display Mode"
    },
    "maxValue": {
      "type": "number",
      "default": 160,
      "min": 100,
      "max": 300,
      "label": "Maximum Speed"
    }
  }
}
```

### Accessing Options in Elements

Use `options.optionName` in binding expressions:

```json
{
  "type": "text",
  "content": "{{options.showUnit ? speed.withUnit : speed.formatted}}",
  "style": {
    "color": "{{options.accentColor}}"
  }
}
```

---

## Bindings Section

The `bindings` section allows pre-computing expressions for reuse and optimization. This is optional but useful for complex widgets.

```json
{
  "bindings": {
    "speedColor": {
      "conditions": [
        { "when": "{{speed.value > 120}}", "value": "#FF0000" },
        { "when": "{{speed.value > 80}}", "value": "#FFAA00" }
      ],
      "default": "#00FF00"
    },
    "displayText": "{{speed.formatted}} {{speed.unit}}"
  }
}
```

### Conditional Bindings

For values that depend on conditions:

| Property | Type | Description |
|----------|------|-------------|
| `conditions` | `array` | Array of condition objects |
| `conditions[].when` | `string` | Expression that evaluates to boolean |
| `conditions[].value` | `string` | Value to use if condition is true |
| `default` | `string` | Value if no conditions match |

---

## Elements Array

The `elements` array contains all visual components of the widget. Elements are rendered in order (first element at back, last at front for ZStack layouts).

### Common Element Properties

All element types share these properties:

| Property | Type | Description |
|----------|------|-------------|
| `type` | `string` | Required. Element type (see below) |
| `id` | `string` | Optional identifier for referencing |
| `position` | `object` | Position within parent (`{ "x": "50%", "y": "50%" }`) |
| `size` | `object` | Size specification (`{ "width": "100%", "height": "50%h" }`) |
| `anchor` | `string` | Anchor point for positioning (see below) |
| `style` | `object` | Style properties (see Style section) |
| `visible` | `string` | Visibility expression (e.g., `"{{speed.value > 0}}"`) |

### Anchor Points

| Value | Description |
|-------|-------------|
| `topLeft` | Top-left corner |
| `top` | Top center |
| `topRight` | Top-right corner |
| `left` | Center left |
| `center` | Center (default) |
| `right` | Center right |
| `bottomLeft` | Bottom-left corner |
| `bottom` | Bottom center |
| `bottomRight` | Bottom-right corner |

---

## Element Types

### Text Element

Displays text with optional binding expressions.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"text"` | Required |
| `content` | `string` | Text content or binding expression |

#### Style Properties for Text

| Property | Type | Description |
|----------|------|-------------|
| `fontSize` | `string` | Font size (e.g., `"50%h"`, `"24px"`) |
| `fontWeight` | `string` | Weight: `ultralight`, `thin`, `light`, `regular`, `medium`, `semibold`, `bold`, `heavy`, `black` |
| `fontDesign` | `string` | Design: `default`, `rounded`, `monospaced`, `serif` |
| `color` | `string` | Text color |
| `textAlignment` | `string` | Alignment: `leading`, `center`, `trailing` |
| `minimumScaleFactor` | `number` | Minimum scale for text shrinking (0.0-1.0) |
| `lineLimit` | `number` | Maximum number of lines |

#### Example

```json
{
  "type": "text",
  "content": "{{speed.formatted}}",
  "position": { "x": "50%", "y": "40%" },
  "style": {
    "fontSize": "60%h",
    "fontWeight": "bold",
    "fontDesign": "rounded",
    "color": "white"
  }
}
```

---

### Icon Element

Displays an SF Symbol icon.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"icon"` | Required |
| `name` | `string` | SF Symbol name (e.g., `"speedometer"`, `"steeringwheel"`) |
| `rotation` | `string` | Rotation in degrees (can be binding) |
| `fill` | `string` | Icon fill color (alternative to style.color) |

#### Example

```json
{
  "type": "icon",
  "name": "steeringwheel",
  "rotation": "{{steering.angle}}",
  "position": { "x": "50%", "y": "50%" },
  "size": { "width": "80%", "height": "80%" },
  "style": {
    "color": "{{autopilot.color}}"
  }
}
```

---

### Rect Element

Draws a rectangle.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"rect"` | Required |
| `fill` | `string` | Fill color |
| `stroke` | `string` | Stroke color |
| `strokeWidth` | `string` | Stroke width |
| `cornerRadius` | `string` | Corner radius |

#### Example

```json
{
  "type": "rect",
  "position": { "x": "0%", "y": "0%" },
  "size": { "width": "100%", "height": "100%" },
  "fill": "rgba(0, 0, 0, 0.6)",
  "cornerRadius": "10px"
}
```

---

### Circle Element

Draws a circle.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"circle"` | Required |
| `radius` | `string` | Circle radius |
| `center` | `object` | Center point (`{ "x": "50%", "y": "50%" }`) |
| `fill` | `string` | Fill color |
| `stroke` | `string` | Stroke color |
| `strokeWidth` | `string` | Stroke width |

#### Example

```json
{
  "type": "circle",
  "radius": "40%",
  "fill": "clear",
  "stroke": "white",
  "strokeWidth": "2px"
}
```

---

### Arc Element

Draws an arc segment.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"arc"` | Required |
| `startAngle` | `string` | Start angle in degrees (0 = top) |
| `endAngle` | `string` | End angle in degrees |
| `stroke` | `string` | Stroke color |
| `strokeWidth` | `string` | Stroke width |

#### Example

```json
{
  "type": "arc",
  "startAngle": "-135",
  "endAngle": "{{speed.value | map:0,160,-135,135}}",
  "size": { "width": "90%", "height": "90%" },
  "stroke": "#00FF00",
  "strokeWidth": "8px"
}
```

---

### Path Element

Draws a custom SVG-like path.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"path"` | Required |
| `d` | `string` | SVG path data (M, L, H, V, Z commands) |
| `fill` | `string` | Fill color |
| `stroke` | `string` | Stroke color |
| `strokeWidth` | `string` | Stroke width |

#### Supported Path Commands

| Command | Description | Example |
|---------|-------------|---------|
| `M x,y` | Move to absolute | `M 50,50` |
| `m dx,dy` | Move to relative | `m 10,10` |
| `L x,y` | Line to absolute | `L 100,100` |
| `l dx,dy` | Line to relative | `l 50,50` |
| `H x` | Horizontal line to | `H 100` |
| `h dx` | Horizontal line by | `h 50` |
| `V y` | Vertical line to | `V 100` |
| `v dy` | Vertical line by | `v 50` |
| `Z` | Close path | `Z` |

#### Example

```json
{
  "type": "path",
  "d": "M 50,10 L 90,90 L 10,90 Z",
  "size": { "width": "100%", "height": "100%" },
  "fill": "yellow",
  "stroke": "orange",
  "strokeWidth": "2px"
}
```

---

### Group Element

Container for grouping child elements with layout control.

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"group"` | Required |
| `layout` | `string` | Layout type: `hstack`, `vstack`, `zstack` |
| `spacing` | `string` | Spacing between children |
| `alignment` | `string` | Alignment: `leading`, `trailing`, `center`, `top`, `bottom`, etc. |
| `children` | `array` | Array of child elements |

#### Layout Types

| Layout | Description |
|--------|-------------|
| `hstack` | Horizontal stack (left to right) |
| `vstack` | Vertical stack (top to bottom) |
| `zstack` | Overlay stack (back to front) |

#### Example

```json
{
  "type": "group",
  "layout": "vstack",
  "spacing": "4px",
  "alignment": "center",
  "children": [
    {
      "type": "text",
      "content": "{{speed.formatted}}",
      "style": { "fontSize": "40%h", "fontWeight": "bold" }
    },
    {
      "type": "text",
      "content": "{{speed.unit}}",
      "style": { "fontSize": "20%h", "color": "gray" }
    }
  ]
}
```

---

### Widget Element

Embeds another widget (for composition).

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"widget"` | Required |
| `ref` | `string` | Widget ID to embed (e.g., `"speed.digital"`) |
| `optionOverrides` | `object` | Override embedded widget's options |

#### Example

```json
{
  "type": "widget",
  "ref": "speed.digital",
  "position": { "x": "25%", "y": "50%" },
  "size": { "width": "50%", "height": "100%" },
  "optionOverrides": {
    "showUnit": false,
    "accentColor": "#FF0000"
  }
}
```

---

### Image Element

Displays an image (placeholder implementation).

| Property | Type | Description |
|----------|------|-------------|
| `type` | `"image"` | Required |

Note: Image elements currently render as gray placeholders. Full image support is planned for future versions.

---

## Style Properties

Complete reference for the `style` object.

### Text Styles

| Property | Type | Values |
|----------|------|--------|
| `fontSize` | `string` | Dimension value |
| `fontWeight` | `string` | `ultralight`, `thin`, `light`, `regular`, `medium`, `semibold`, `bold`, `heavy`, `black` |
| `fontDesign` | `string` | `default`, `rounded`, `monospaced`, `serif` |
| `color` | `string` | Color value |
| `textAlignment` | `string` | `leading`, `center`, `trailing` |
| `minimumScaleFactor` | `number` | 0.0 - 1.0 |
| `lineLimit` | `number` | Integer |

### Shape Styles

| Property | Type | Description |
|----------|------|-------------|
| `fill` | `string` | Fill color |
| `stroke` | `string` | Stroke color |
| `strokeWidth` | `string` | Stroke width |
| `cornerRadius` | `string` | Corner radius for rectangles |

### Layout & Effects

| Property | Type | Description |
|----------|------|-------------|
| `padding` | `object/string` | Padding values |
| `opacity` | `number` | 0.0 - 1.0 |
| `blur` | `number` | Blur radius |
| `rotation` | `string` | Rotation in degrees |
| `scale` | `number` | Scale factor |
| `backgroundColor` | `string` | Background color |
| `backgroundCornerRadius` | `string` | Background corner radius |

### Shadow

| Property | Type | Description |
|----------|------|-------------|
| `shadow.color` | `string` | Shadow color |
| `shadow.radius` | `string` | Blur radius |
| `shadow.x` | `string` | Horizontal offset |
| `shadow.y` | `string` | Vertical offset |

### Padding

Padding can be specified as a single value or per-side:

```json
{
  "style": {
    "padding": "10px"
  }
}
```

```json
{
  "style": {
    "padding": {
      "top": "10px",
      "bottom": "10px",
      "leading": "16px",
      "trailing": "16px"
    }
  }
}
```

---

## Dimension Values

Dimension values support multiple units for flexible sizing.

### Units

| Unit | Syntax | Description |
|------|--------|-------------|
| Percentage of height | `60%h` | Relative to container height |
| Percentage of width | `30%w` | Relative to container width |
| Percentage of minimum | `50%` | Relative to smaller dimension |
| Pixels | `100px` | Fixed pixel value |
| Plain number | `50` | Treated as percentage of minimum |

### Examples

```json
{
  "size": {
    "width": "100%w",
    "height": "50%h"
  }
}
```

```json
{
  "style": {
    "fontSize": "40%h",
    "strokeWidth": "3px"
  }
}
```

### Resolution

When resolved:
- `60%h` on a 100px tall container = 60px
- `30%w` on a 200px wide container = 60px
- `50%` on a 100x200 container = 50px (uses smaller dimension)
- `100px` = 100px regardless of container

---

## Color Values

### Named Colors

| Name | Value |
|------|-------|
| `white` | #FFFFFF |
| `black` | #000000 |
| `red` | System red |
| `green` | System green |
| `blue` | System blue |
| `orange` | System orange |
| `yellow` | System yellow |
| `purple` | System purple |
| `pink` | System pink |
| `gray` / `grey` | System gray |
| `cyan` | System cyan |
| `clear` | Transparent |

### Hex Colors

| Format | Example |
|--------|---------|
| 3-digit hex | `#FFF` (expands to #FFFFFF) |
| 6-digit hex | `#FF5500` |
| 8-digit hex (with alpha) | `#FF550080` |

### RGB/RGBA

```json
{
  "color": "rgb(255, 85, 0)"
}
```

```json
{
  "color": "rgba(255, 85, 0, 0.5)"
}
```

### Dynamic Colors via Bindings

```json
{
  "style": {
    "color": "{{autopilot.color}}"
  }
}
```

```json
{
  "style": {
    "color": "{{speed.value > 100 ? '#FF0000' : '#00FF00'}}"
  }
}
```

---

## Complete Example

A complete speed gauge widget demonstrating multiple features:

```json
{
  "$schema": "widget-schema-v1",
  "id": "speed.gauge-arc",
  "version": "1.0.0",
  "category": "speed",
  "name": "Arc Speed Gauge",
  "description": "Speed displayed with an animated arc indicator",
  "author": "Sentry Clip Viewer",
  "canvas": {
    "aspectRatio": 1.2,
    "minWidth": 0.15,
    "minHeight": 0.12
  },
  "options": {
    "maxSpeed": {
      "type": "number",
      "default": 160,
      "min": 100,
      "max": 300,
      "label": "Maximum Speed"
    },
    "showUnit": {
      "type": "boolean",
      "default": true,
      "label": "Show Unit"
    },
    "arcColor": {
      "type": "color",
      "default": "#00FF00",
      "label": "Arc Color"
    }
  },
  "elements": [
    {
      "type": "rect",
      "size": { "width": "100%", "height": "100%" },
      "fill": "rgba(0, 0, 0, 0.6)",
      "cornerRadius": "10%"
    },
    {
      "type": "arc",
      "startAngle": "-135",
      "endAngle": "135",
      "size": { "width": "85%", "height": "85%" },
      "position": { "x": "50%", "y": "50%" },
      "stroke": "rgba(255, 255, 255, 0.2)",
      "strokeWidth": "8px"
    },
    {
      "type": "arc",
      "startAngle": "-135",
      "endAngle": "{{speed.value | map:0,options.maxSpeed,-135,135}}",
      "size": { "width": "85%", "height": "85%" },
      "position": { "x": "50%", "y": "50%" },
      "stroke": "{{options.arcColor}}",
      "strokeWidth": "8px"
    },
    {
      "type": "group",
      "layout": "vstack",
      "spacing": "2px",
      "position": { "x": "50%", "y": "55%" },
      "children": [
        {
          "type": "text",
          "content": "{{speed.formatted}}",
          "style": {
            "fontSize": "35%h",
            "fontWeight": "bold",
            "fontDesign": "rounded",
            "color": "white"
          }
        },
        {
          "type": "text",
          "content": "{{speed.unit}}",
          "visible": "{{options.showUnit}}",
          "style": {
            "fontSize": "15%h",
            "color": "rgba(255, 255, 255, 0.7)"
          }
        }
      ]
    }
  ]
}
```

---

## Widget Pack Manifest

Widgets are organized into packs with a manifest file (`pack.json`).

### Manifest Structure

| Property | Type | Description |
|----------|------|-------------|
| `$schema` | `string` | Schema version |
| `id` | `string` | Pack identifier |
| `version` | `string` | Pack version |
| `name` | `string` | Display name |
| `description` | `string` | Pack description |
| `author` | `string` | Pack author |
| `compatibility` | `object` | App version requirements |
| `widgets` | `array` | List of widget references |
| `presets` | `array` | List of preset layouts |

### Example Pack Manifest

```json
{
  "$schema": "widget-pack-v1",
  "id": "custom-widgets",
  "version": "1.0.0",
  "name": "My Custom Widgets",
  "description": "A collection of custom telemetry widgets",
  "author": "Your Name",
  "compatibility": {
    "minAppVersion": "3.0.0",
    "schemaVersion": "1.0"
  },
  "widgets": [
    { "file": "widgets/speed-gauge.widget.json", "category": "speed" },
    { "file": "widgets/compass.widget.json", "category": "navigation" }
  ],
  "presets": [
    {
      "file": "presets/minimal.preset.json",
      "name": "Minimal",
      "description": "A minimal overlay layout"
    }
  ]
}
```

---

## Validation Rules

The widget parser enforces these validation rules:

1. **Widget ID**: Must not be empty
2. **Version**: Must match semantic versioning pattern (`X.Y` or `X.Y.Z`)
3. **Elements**: At least one element required
4. **Text elements**: Must have `content` property
5. **Icon elements**: Must have `name` property
6. **Group elements**: Must have non-empty `children` array
7. **Widget elements**: Must have `ref` property
8. **Path elements**: Must have `d` property
9. **Binding expressions**: Must have balanced `{{` and `}}` delimiters
10. **Expression length**: Maximum 500 characters per expression

---

## Security Limits

The expression parser is intentionally limited for security:

| Limit | Value |
|-------|-------|
| Maximum expression length | 500 characters |
| Maximum nesting depth | 10 levels |
| Maximum operators per expression | 20 |

Only whitelisted functions are available: `min`, `max`, `abs`, `round`, `floor`, `ceil`, `clamp`

---

## Best Practices

1. **Use semantic IDs**: Follow `category.variant` naming (e.g., `speed.minimal`, `speed.detailed`)
2. **Provide options**: Make widgets configurable for user customization
3. **Use percentage units**: Prefer `%h` and `%w` over `px` for responsive sizing
4. **Add visibility conditions**: Hide elements when data is unavailable
5. **Test aspect ratios**: Ensure widgets look good at various sizes
6. **Include fallbacks**: Use `| default:value` filter for optional data
7. **Document options**: Add meaningful labels to all options
