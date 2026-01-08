# Data Bindings Reference

This document provides a complete reference for all available data bindings in the declarative widget system.

## Table of Contents

- [Overview](#overview)
- [Binding Syntax](#binding-syntax)
- [Speed and Motion](#speed-and-motion)
- [Throttle and Brake](#throttle-and-brake)
- [Steering](#steering)
- [Autopilot](#autopilot)
- [Turn Signals](#turn-signals)
- [Gear](#gear)
- [Navigation](#navigation)
- [G-Force](#g-force)
- [Location](#location)
- [Time](#time)
- [Third-Party Data](#third-party-data)
- [Widget Options](#widget-options)
- [Filters](#filters)
- [Operators](#operators)
- [Functions](#functions)
- [Conditional Expressions](#conditional-expressions)

---

## Overview

Bindings connect your widget to live telemetry data. When the vehicle data changes, bound values update automatically.

### Data Sources

| Source | Description | Update Rate |
|--------|-------------|-------------|
| SEI Metadata | Embedded in dashcam video | 30 fps |
| Third-Party APIs | Tessie, TeslaMate, TeslaScope | ~60 seconds |
| Widget Options | User-configured settings | On change |

---

## Binding Syntax

### Basic Binding

Wrap a binding path in double curly braces:

```json
{
  "content": "{{speed.formatted}}"
}
```

### Inline with Text

Mix bindings with static text:

```json
{
  "content": "Speed: {{speed.formatted}} {{speed.unit}}"
}
```

### In Style Properties

Use bindings for dynamic colors:

```json
{
  "style": {
    "color": "{{autopilot.color}}"
  }
}
```

### With Expressions

Combine bindings with operators:

```json
{
  "content": "{{speed.value > 100 ? 'FAST' : 'Normal'}}"
}
```

---

## Speed and Motion

Data from vehicle speed sensors.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `speed.value` | `number` | Raw speed in m/s | `29.06` |
| `speed.formatted` | `string` | Speed in user's unit (no unit suffix) | `"65"` |
| `speed.withUnit` | `string` | Speed with unit suffix | `"65 mph"` |
| `speed.unit` | `string` | Unit symbol | `"mph"` or `"km/h"` |

### Usage Examples

**Digital speedometer:**
```json
{
  "type": "text",
  "content": "{{speed.formatted}}",
  "style": { "fontSize": "60%h", "fontWeight": "bold" }
}
```

**Speed with conditional color:**
```json
{
  "type": "text",
  "content": "{{speed.formatted}}",
  "style": {
    "color": "{{speed.value > 35 ? '#FF0000' : '#00FF00'}}"
  }
}
```

**Speed gauge arc:**
```json
{
  "type": "arc",
  "startAngle": "-135",
  "endAngle": "{{speed.value | map:0,160,-135,135}}"
}
```

---

## Throttle and Brake

Pedal position data from the vehicle.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `throttle.percent` | `number` | Throttle position 0-100 | `45.5` |
| `throttle.formatted` | `string` | Formatted with percent sign | `"45%"` |
| `brake.applied` | `boolean` | Whether brake pedal is pressed | `true` |

### Usage Examples

**Throttle bar:**
```json
{
  "type": "rect",
  "size": {
    "width": "20px",
    "height": "{{throttle.percent}}%h"
  },
  "fill": "{{brake.applied ? '#FF0000' : '#00FF00'}}"
}
```

**Brake indicator:**
```json
{
  "type": "icon",
  "name": "brake.light",
  "visible": "{{brake.applied}}",
  "style": { "color": "#FF0000" }
}
```

**Combined throttle/brake display:**
```json
{
  "type": "text",
  "content": "{{brake.applied ? 'B' : throttle.formatted}}",
  "style": {
    "color": "{{brake.applied ? '#FF0000' : '#00FF00'}}"
  }
}
```

---

## Steering

Steering wheel angle data.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `steering.angle` | `number` | Steering angle in degrees (+ = right, - = left) | `-12.5` |
| `steering.formatted` | `string` | Formatted angle with degree symbol | `"-12.5deg"` |

### Usage Examples

**Rotating steering wheel icon:**
```json
{
  "type": "icon",
  "name": "steeringwheel",
  "rotation": "{{steering.angle}}",
  "style": { "color": "{{autopilot.color}}" }
}
```

**Steering angle display:**
```json
{
  "type": "text",
  "content": "{{steering.angle | abs | round}}deg",
  "style": { "fontSize": "20%h" }
}
```

**Direction indicator:**
```json
{
  "type": "text",
  "content": "{{steering.angle > 5 ? 'R' : (steering.angle < -5 ? 'L' : '-')}}"
}
```

---

## Autopilot

Tesla Autopilot and FSD engagement status.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `autopilot.state` | `string` | Current autopilot state | `"fullSelfDriving"` |
| `autopilot.color` | `string` | Color hex for current state | `"#0066FF"` |
| `autopilot.engaged` | `boolean` | Whether any autopilot mode is active | `true` |

### Autopilot States

| State | Color | Description |
|-------|-------|-------------|
| `fullSelfDriving` | `#0066FF` (Blue) | FSD is active |
| `autosteer` | `#00CC00` (Green) | Autosteer is active |
| `trafficAwareCruise` | `#FF9900` (Orange) | TACC is active |
| `available` | `#FFFFFF` (White) | Available but not engaged |
| `disabled` | `#FFFFFF` (White) | Not available |

### Usage Examples

**Autopilot status indicator:**
```json
{
  "type": "icon",
  "name": "steeringwheel",
  "style": { "color": "{{autopilot.color}}" }
}
```

**Autopilot status text:**
```json
{
  "type": "text",
  "content": "{{autopilot.state == 'fullSelfDriving' ? 'FSD' : (autopilot.state == 'autosteer' ? 'AS' : (autopilot.state == 'trafficAwareCruise' ? 'TACC' : ''))}}",
  "visible": "{{autopilot.engaged}}"
}
```

**Engagement glow effect:**
```json
{
  "type": "circle",
  "radius": "45%",
  "fill": "{{autopilot.engaged ? autopilot.color : 'clear'}}",
  "style": { "opacity": 0.3 }
}
```

---

## Turn Signals

Blinker/indicator status.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `blinker.left` | `boolean` | Left turn signal active | `true` |
| `blinker.right` | `boolean` | Right turn signal active | `false` |

### Usage Examples

**Left turn signal:**
```json
{
  "type": "icon",
  "name": "arrowtriangle.left.fill",
  "visible": "{{blinker.left}}",
  "style": { "color": "#00FF00" }
}
```

**Right turn signal:**
```json
{
  "type": "icon",
  "name": "arrowtriangle.right.fill",
  "visible": "{{blinker.right}}",
  "style": { "color": "#00FF00" }
}
```

**Hazard indicator (both signals):**
```json
{
  "type": "icon",
  "name": "exclamationmark.triangle.fill",
  "visible": "{{blinker.left && blinker.right}}",
  "style": { "color": "#FF9900" }
}
```

---

## Gear

Current gear position.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `gear.letter` | `string` | Single letter gear indicator | `"D"` |
| `gear.name` | `string` | Full gear name | `"Drive"` |
| `gear.color` | `string` | Color hex for current gear | `"#00CC00"` |

### Gear Values

| Letter | Name | Color |
|--------|------|-------|
| `P` | Park | `#007AFF` (Blue) |
| `R` | Reverse | `#FF3B30` (Red) |
| `N` | Neutral | `#FF9500` (Orange) |
| `D` | Drive | `#00CC00` (Green) |

### Usage Examples

**Gear indicator:**
```json
{
  "type": "text",
  "content": "{{gear.letter}}",
  "style": {
    "fontSize": "30%h",
    "fontWeight": "bold",
    "color": "{{gear.color}}"
  }
}
```

**Gear badge with background:**
```json
{
  "type": "group",
  "layout": "zstack",
  "children": [
    {
      "type": "circle",
      "radius": "15%",
      "fill": "{{gear.color}}"
    },
    {
      "type": "text",
      "content": "{{gear.letter}}",
      "style": { "color": "white", "fontWeight": "bold" }
    }
  ]
}
```

---

## Navigation

Heading and direction data.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `heading.degrees` | `number` | Heading in degrees (0-360, 0 = North) | `247.5` |
| `heading.cardinal` | `string` | Cardinal/intercardinal direction | `"WSW"` |
| `heading.formatted` | `string` | Formatted heading with degree symbol | `"247deg"` |

### Cardinal Directions

| Range | Cardinal |
|-------|----------|
| 337.5 - 22.5 | N |
| 22.5 - 67.5 | NE |
| 67.5 - 112.5 | E |
| 112.5 - 157.5 | SE |
| 157.5 - 202.5 | S |
| 202.5 - 247.5 | SW |
| 247.5 - 292.5 | W |
| 292.5 - 337.5 | NW |

### Usage Examples

**Compass display:**
```json
{
  "type": "text",
  "content": "{{heading.cardinal}}",
  "style": { "fontSize": "20%h", "color": "cyan" }
}
```

**Rotating compass icon:**
```json
{
  "type": "icon",
  "name": "location.north.fill",
  "rotation": "{{0 - heading.degrees}}",
  "style": { "color": "cyan" }
}
```

**Heading in degrees:**
```json
{
  "type": "text",
  "content": "{{heading.degrees | round}}deg"
}
```

---

## G-Force

Acceleration data from vehicle sensors.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `gforce.x` | `number` | Lateral G-force (+ = right, - = left) | `0.25` |
| `gforce.y` | `number` | Longitudinal G-force (+ = forward, - = back) | `-0.15` |
| `gforce.magnitude` | `number` | Combined G-force magnitude | `0.29` |

### Usage Examples

**G-force meter dot position:**
```json
{
  "type": "circle",
  "radius": "5%",
  "position": {
    "x": "{{50 + (gforce.x | clamp:-1,1) * 40}}%",
    "y": "{{50 - (gforce.y | clamp:-1,1) * 40}}%"
  },
  "fill": "#00FF00"
}
```

**G-force magnitude display:**
```json
{
  "type": "text",
  "content": "{{gforce.magnitude | round:2}}G"
}
```

**G-force colored by intensity:**
```json
{
  "type": "circle",
  "fill": "{{gforce.magnitude > 0.5 ? '#FF0000' : (gforce.magnitude > 0.3 ? '#FFAA00' : '#00FF00')}}"
}
```

---

## Location

GPS coordinates and reverse-geocoded location.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `location.lat` | `number` | Latitude | `37.7749` |
| `location.lon` | `number` | Longitude | `-122.4194` |
| `location.name` | `string` | Reverse-geocoded address | `"San Francisco, CA"` |
| `location.formatted` | `string` | Formatted coordinates | `"37.77490, -122.41940"` |

### Usage Examples

**Location display:**
```json
{
  "type": "text",
  "content": "{{location.name ?? location.formatted ?? 'No GPS'}}",
  "style": { "fontSize": "14px" }
}
```

**Coordinates:**
```json
{
  "type": "text",
  "content": "{{location.lat | round:4}}, {{location.lon | round:4}}"
}
```

**GPS status indicator:**
```json
{
  "type": "icon",
  "name": "{{location.lat ? 'location.fill' : 'location.slash'}}",
  "style": {
    "color": "{{location.lat ? '#00FF00' : '#FF0000'}}"
  }
}
```

---

## Time

Timestamp data from the recording.

### Available Bindings

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `timestamp.iso` | `string` | ISO 8601 formatted timestamp | `"2025-01-08T14:30:00Z"` |
| `timestamp.time` | `string` | Time only (respects timeFormat option) | `"14:30:00"` |
| `timestamp.date` | `string` | Date only (respects dateFormat option) | `"2025-01-08"` |

### Usage Examples

**Time display:**
```json
{
  "type": "text",
  "content": "{{timestamp.time}}",
  "style": { "fontDesign": "monospaced" }
}
```

**Date and time:**
```json
{
  "type": "text",
  "content": "{{timestamp.date}} {{timestamp.time}}"
}
```

---

## Third-Party Data

Data from connected third-party services (Tessie, TeslaMate, TeslaScope).

### Battery and Range

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `battery.percent` | `number` | Battery level 0-100 | `78.5` |
| `battery.range` | `number` | Estimated range in miles | `245.3` |
| `battery.formatted` | `string` | Formatted battery percentage | `"78%"` |

### Temperature

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `temp.inside` | `number` | Interior temperature | `21.5` |
| `temp.outside` | `number` | Exterior temperature | `15.2` |
| `temp.insideFormatted` | `string` | Formatted inside temp | `"21.5deg"` |
| `temp.outsideFormatted` | `string` | Formatted outside temp | `"15.2deg"` |

### Other Data

| Binding | Type | Description | Example Value |
|---------|------|-------------|---------------|
| `power.value` | `number` | Power in kW (+ = consuming, - = regen) | `-15.5` |
| `odometer.value` | `number` | Odometer in miles | `12345.6` |
| `elevation.value` | `number` | Elevation in meters | `156.2` |

### Usage Examples

**Battery display:**
```json
{
  "type": "text",
  "content": "{{battery.formatted ?? '--'}}",
  "style": {
    "color": "{{battery.percent < 20 ? '#FF0000' : (battery.percent < 50 ? '#FFAA00' : '#00FF00')}}"
  }
}
```

**Temperature display:**
```json
{
  "type": "group",
  "layout": "hstack",
  "spacing": "10px",
  "children": [
    {
      "type": "text",
      "content": "In: {{temp.insideFormatted ?? '--'}}"
    },
    {
      "type": "text",
      "content": "Out: {{temp.outsideFormatted ?? '--'}}"
    }
  ]
}
```

**Power indicator:**
```json
{
  "type": "text",
  "content": "{{power.value | round}}kW",
  "style": {
    "color": "{{power.value < 0 ? '#00FF00' : '#FF9900'}}"
  }
}
```

---

## Widget Options

Access user-configured widget options.

### Available Option Bindings

| Binding | Type | Description |
|---------|------|-------------|
| `options.watermarkText` | `string` | Custom watermark text |
| `options.watermarkPosition` | `string` | Watermark position (`topLeft`, `topRight`) |
| `options.speedUnit` | `string` | Speed unit (`mph`, `kmh`) |
| `options.speedUnit.symbol` | `string` | Unit symbol (`"mph"`, `"km/h"`) |
| `options.speedUnit.displayName` | `string` | Full unit name |
| `options.mapStyle` | `string` | Map style |
| `options.mapZoom` | `string` | Map zoom level |
| `options.showMapTrail` | `boolean` | Show trail on map |
| `options.showMapHeading` | `boolean` | Show heading indicator |
| `options.gForceShowTrail` | `boolean` | Show G-force trail |
| `options.gForceScale` | `string` | G-force scale |
| `options.timeFormat` | `string` | Time format |
| `options.showSeconds` | `boolean` | Show seconds in time |
| `options.dateFormat` | `string` | Date format |
| `options.temperatureUnit` | `string` | Temperature unit |

### Usage Examples

**Conditional unit display:**
```json
{
  "type": "text",
  "content": "{{options.speedUnit.symbol}}"
}
```

**Custom watermark:**
```json
{
  "type": "text",
  "content": "{{options.watermarkText ?? ''}}",
  "visible": "{{options.watermarkText}}"
}
```

---

## Filters

Filters transform binding values using pipe syntax: `{{value | filter:args}}`

### Available Filters

| Filter | Arguments | Description | Example |
|--------|-----------|-------------|---------|
| `format` | format type | Format numbers | `{{speed.value \| format:integer}}` |
| `default` | fallback value | Provide fallback for nil | `{{location.name \| default:'No GPS'}}` |
| `abs` | none | Absolute value | `{{steering.angle \| abs}}` |
| `round` | [precision] | Round to N decimals | `{{speed.value \| round:1}}` |
| `floor` | none | Round down | `{{speed.value \| floor}}` |
| `ceil` | none | Round up | `{{speed.value \| ceil}}` |
| `clamp` | min,max | Clamp to range | `{{gforce.x \| clamp:-1,1}}` |
| `map` | inMin,inMax,outMin,outMax | Linear interpolation | `{{speed.value \| map:0,160,-135,135}}` |
| `sign` | none | Add +/- prefix | `{{power.value \| sign}}` |
| `uppercase` | none | Convert to uppercase | `{{gear.name \| uppercase}}` |
| `lowercase` | none | Convert to lowercase | `{{gear.name \| lowercase}}` |

### Format Types

| Type | Description | Example |
|------|-------------|---------|
| `integer` | Whole number | `65` |
| `decimal:N` | N decimal places | `65.12` |

### Usage Examples

**Formatted speed:**
```json
{
  "content": "{{speed.value | format:integer}}"
}
```

**Clamped G-force:**
```json
{
  "content": "{{gforce.x | clamp:-1,1 | round:2}}"
}
```

**Arc angle mapping:**
```json
{
  "endAngle": "{{speed.value | map:0,160,-135,135}}"
}
```

**Fallback value:**
```json
{
  "content": "{{location.name | default:'Unknown Location'}}"
}
```

---

## Operators

### Arithmetic Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `+` | Addition | `{{speed.value + 10}}` |
| `-` | Subtraction | `{{100 - speed.value}}` |
| `*` | Multiplication | `{{speed.value * 2.237}}` |
| `/` | Division | `{{speed.value / 3.6}}` |
| `%` | Modulo | `{{heading.degrees % 90}}` |

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `==` | Equal | `{{gear.letter == 'D'}}` |
| `!=` | Not equal | `{{gear.letter != 'P'}}` |
| `>` | Greater than | `{{speed.value > 100}}` |
| `<` | Less than | `{{battery.percent < 20}}` |
| `>=` | Greater or equal | `{{speed.value >= 65}}` |
| `<=` | Less or equal | `{{battery.percent <= 50}}` |

### Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `&&` | Logical AND | `{{blinker.left && blinker.right}}` |
| `\|\|` | Logical OR | `{{blinker.left \|\| blinker.right}}` |
| `!` | Logical NOT | `{{!autopilot.engaged}}` |

### Null Coalescing

| Operator | Description | Example |
|----------|-------------|---------|
| `??` | Nil fallback | `{{location.name ?? 'No GPS'}}` |

---

## Functions

Whitelisted functions for use in expressions.

### Available Functions

| Function | Arguments | Description | Example |
|----------|-----------|-------------|---------|
| `min(a, b, ...)` | 2+ numbers | Minimum value | `{{min(speed.value, 100)}}` |
| `max(a, b, ...)` | 2+ numbers | Maximum value | `{{max(speed.value, 0)}}` |
| `abs(n)` | 1 number | Absolute value | `{{abs(steering.angle)}}` |
| `round(n, [precision])` | 1-2 numbers | Round to precision | `{{round(speed.value, 1)}}` |
| `floor(n)` | 1 number | Round down | `{{floor(speed.value)}}` |
| `ceil(n)` | 1 number | Round up | `{{ceil(speed.value)}}` |
| `clamp(value, min, max)` | 3 numbers | Clamp to range | `{{clamp(gforce.x, -1, 1)}}` |

### Usage Examples

**Clamped display:**
```json
{
  "content": "{{clamp(speed.value, 0, 160)}}"
}
```

**Rounded magnitude:**
```json
{
  "content": "{{round(gforce.magnitude, 2)}}G"
}
```

---

## Conditional Expressions

### Ternary Operator

```
{{condition ? trueValue : falseValue}}
```

**Examples:**

```json
{
  "content": "{{speed.value > 100 ? 'FAST' : 'Normal'}}"
}
```

```json
{
  "style": {
    "color": "{{brake.applied ? '#FF0000' : '#00FF00'}}"
  }
}
```

### Nested Ternary

```json
{
  "content": "{{speed.value > 100 ? 'Fast' : (speed.value > 50 ? 'Normal' : 'Slow')}}"
}
```

### Conditional Bindings Object

For complex conditions, use the conditional bindings syntax:

```json
{
  "bindings": {
    "speedColor": {
      "conditions": [
        { "when": "{{speed.value > 120}}", "value": "#FF0000" },
        { "when": "{{speed.value > 80}}", "value": "#FFAA00" },
        { "when": "{{speed.value > 40}}", "value": "#FFFF00" }
      ],
      "default": "#00FF00"
    }
  }
}
```

Then reference in elements:

```json
{
  "style": {
    "color": "{{speedColor}}"
  }
}
```

---

## Visibility Expressions

Control element visibility with the `visible` property:

```json
{
  "type": "icon",
  "name": "brake.light",
  "visible": "{{brake.applied}}"
}
```

**Common visibility patterns:**

```json
{
  "visible": "{{speed.value > 0}}"
}
```

```json
{
  "visible": "{{autopilot.engaged}}"
}
```

```json
{
  "visible": "{{battery.percent != null}}"
}
```

```json
{
  "visible": "{{blinker.left || blinker.right}}"
}
```

---

## Expression Limits

For security and performance, expressions have the following limits:

| Limit | Value |
|-------|-------|
| Maximum expression length | 500 characters |
| Maximum nesting depth | 10 levels |
| Maximum operators | 20 per expression |

---

## Best Practices

1. **Use fallbacks**: Always provide defaults for optional data
   ```json
   "{{location.name | default:'--'}}"
   ```

2. **Clamp numeric values**: Prevent layout issues with unexpected values
   ```json
   "{{gforce.x | clamp:-1,1}}"
   ```

3. **Check for nil in visibility**: Hide elements when data is unavailable
   ```json
   "visible": "{{battery.percent != null}}"
   ```

4. **Use appropriate precision**: Round values for cleaner display
   ```json
   "{{speed.value | round:1}}"
   ```

5. **Prefer filters over complex expressions**: Filters are more readable
   ```json
   // Prefer:
   "{{speed.value | map:0,160,-135,135}}"

   // Over:
   "{{-135 + (speed.value / 160) * 270}}"
   ```

6. **Test edge cases**: Consider zero, negative, and nil values
   ```json
   "{{power.value < 0 ? 'Regen' : 'Using'}}"
   ```
