# Widget SDK Expression Syntax

This document describes the expression syntax used in widget definitions for dynamic data binding and conditional rendering.

## Overview

Expressions allow widgets to bind to telemetry data and perform dynamic calculations. They are evaluated at render time for each frame of video export or during live playback preview.

## Basic Binding Syntax

### Simple Binding

Use double curly braces to bind to a data path:

```json
{
  "content": "{{speed.value}}"
}
```

### Nested Path Access

Access nested properties using dot notation:

```json
{
  "content": "{{autopilot.state}}",
  "color": "{{location.coordinates.latitude}}"
}
```

## Available Data Paths

### Speed & Motion

| Path | Type | Description |
|------|------|-------------|
| `speed.value` | Double | Current speed value |
| `speed.formatted` | String | Speed with formatting applied |
| `speed.unit` | String | "mph" or "km/h" |
| `speed.raw` | Double | Speed in m/s (unconverted) |

### Throttle & Brake

| Path | Type | Description |
|------|------|-------------|
| `throttle.percent` | Double | 0-100 throttle position |
| `throttle.normalized` | Double | 0-1 normalized value |
| `brake.applied` | Bool | Brake pedal pressed |
| `brake.percent` | Double | Brake pressure (if available) |

### Steering

| Path | Type | Description |
|------|------|-------------|
| `steering.angle` | Double | Steering angle in degrees |
| `steering.normalized` | Double | -1 to 1 normalized value |
| `steering.direction` | String | "left", "right", or "center" |

### Autopilot

| Path | Type | Description |
|------|------|-------------|
| `autopilot.state` | String | "fsd", "autosteer", "tacc", "none" |
| `autopilot.active` | Bool | Any autopilot mode active |
| `autopilot.color` | String | Hex color for current state |

### Turn Signals

| Path | Type | Description |
|------|------|-------------|
| `signals.left` | Bool | Left blinker active |
| `signals.right` | Bool | Right blinker active |
| `signals.hazards` | Bool | Both signals active |

### Gear

| Path | Type | Description |
|------|------|-------------|
| `gear.state` | String | "P", "R", "N", "D" |
| `gear.color` | String | Hex color for gear state |

### Compass & Heading

| Path | Type | Description |
|------|------|-------------|
| `heading.degrees` | Double | 0-360 heading in degrees |
| `heading.cardinal` | String | "N", "NE", "E", etc. |
| `heading.rotation` | Double | Rotation for compass needle |

### G-Force

| Path | Type | Description |
|------|------|-------------|
| `gforce.x` | Double | Lateral G-force |
| `gforce.y` | Double | Longitudinal G-force |
| `gforce.z` | Double | Vertical G-force |
| `gforce.magnitude` | Double | Total G-force magnitude |

### Location

| Path | Type | Description |
|------|------|-------------|
| `location.latitude` | Double | GPS latitude |
| `location.longitude` | Double | GPS longitude |
| `location.name` | String | Reverse geocoded location |
| `location.formatted` | String | Formatted coordinates |

### Time

| Path | Type | Description |
|------|------|-------------|
| `time.timestamp` | Date | Current timestamp |
| `time.formatted` | String | Formatted time string |
| `time.date` | String | Formatted date string |

### Third-Party Data

| Path | Type | Description |
|------|------|-------------|
| `battery.percent` | Double | Battery level 0-100 |
| `battery.range` | Double | Estimated range |
| `battery.charging` | Bool | Currently charging |
| `temperature.inside` | Double | Cabin temperature |
| `temperature.outside` | Double | Exterior temperature |
| `temperature.unit` | String | "C" or "F" |

---

## Operators

### Nil Coalescing

Provide a default value when the binding is nil:

```json
{
  "content": "{{speed.formatted ?? '--'}}"
}
```

```json
{
  "content": "{{location.name ?? 'Unknown Location'}}"
}
```

### Ternary Operator

Conditional value based on a boolean expression:

```json
{
  "color": "{{autopilot.active ? '#00FF00' : '#FFFFFF'}}"
}
```

```json
{
  "content": "{{brake.applied ? 'BRAKE' : ''}}"
}
```

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `==` | Equal | `{{gear.state == 'D'}}` |
| `!=` | Not equal | `{{autopilot.state != 'none'}}` |
| `>` | Greater than | `{{speed.value > 60}}` |
| `<` | Less than | `{{battery.percent < 20}}` |
| `>=` | Greater or equal | `{{throttle.percent >= 100}}` |
| `<=` | Less or equal | `{{gforce.magnitude <= 1.0}}` |

### Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `&&` | Logical AND | `{{autopilot.active && speed.value > 0}}` |
| `\|\|` | Logical OR | `{{signals.left \|\| signals.right}}` |
| `!` | Logical NOT | `{{!brake.applied}}` |

### Arithmetic Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `+` | Addition | `{{speed.value + 5}}` |
| `-` | Subtraction | `{{100 - battery.percent}}` |
| `*` | Multiplication | `{{throttle.percent * 0.01}}` |
| `/` | Division | `{{heading.degrees / 360}}` |
| `%` | Modulo | `{{heading.degrees % 90}}` |

---

## Functions

### Math Functions

| Function | Description | Example |
|----------|-------------|---------|
| `min(a, b)` | Minimum of two values | `{{min(speed.value, 120)}}` |
| `max(a, b)` | Maximum of two values | `{{max(throttle.percent, 0)}}` |
| `abs(x)` | Absolute value | `{{abs(steering.angle)}}` |
| `round(x)` | Round to nearest integer | `{{round(speed.value)}}` |
| `floor(x)` | Round down | `{{floor(battery.percent)}}` |
| `ceil(x)` | Round up | `{{ceil(gforce.magnitude)}}` |
| `clamp(x, min, max)` | Clamp value to range | `{{clamp(throttle.percent, 0, 100)}}` |

### Examples

```json
{
  "rotation": "{{clamp(steering.angle, -180, 180)}}",
  "width": "{{max(throttle.percent, 5)}}%",
  "value": "{{round(speed.value)}}"
}
```

---

## Filters

Filters transform values using pipe syntax: `{{value | filter}}` or `{{value | filter:arg1,arg2}}`

### Available Filters

| Filter | Arguments | Description | Example |
|--------|-----------|-------------|---------|
| `format` | pattern | Format string/number | `{{speed.value \| format:'%.1f'}}` |
| `default` | value | Fallback for nil | `{{location.name \| default:'--'}}` |
| `abs` | - | Absolute value | `{{steering.angle \| abs}}` |
| `round` | decimals | Round to decimals | `{{speed.value \| round:1}}` |
| `clamp` | min, max | Clamp to range | `{{throttle.percent \| clamp:0,100}}` |
| `map` | in1,in2,out1,out2 | Map value range | `{{throttle.percent \| map:0,100,0,1}}` |
| `sign` | - | Get sign (-1, 0, 1) | `{{steering.angle \| sign}}` |
| `uppercase` | - | Convert to uppercase | `{{gear.state \| uppercase}}` |
| `lowercase` | - | Convert to lowercase | `{{autopilot.state \| lowercase}}` |

### Filter Chaining

Multiple filters can be chained:

```json
{
  "content": "{{speed.value | round:0 | default:'--'}}"
}
```

```json
{
  "height": "{{throttle.percent | clamp:0,100 | map:0,100,0,50}}px"
}
```

---

## Complex Expression Examples

### Conditional Coloring

```json
{
  "color": "{{speed.value > 100 ? '#FF0000' : (speed.value > 60 ? '#FFFF00' : '#00FF00')}}"
}
```

### Dynamic Content

```json
{
  "content": "{{speed.formatted ?? '--'}} {{speed.unit}}"
}
```

### Autopilot Status Color

```json
{
  "color": "{{autopilot.state == 'fsd' ? '#0066FF' : (autopilot.state == 'autosteer' ? '#00FF00' : (autopilot.state == 'tacc' ? '#FF9900' : '#FFFFFF'))}}"
}
```

### Throttle Bar Height

```json
{
  "height": "{{throttle.percent | clamp:0,100}}%"
}
```

### Battery Warning Color

```json
{
  "color": "{{battery.percent < 20 ? '#FF0000' : (battery.percent < 40 ? '#FFFF00' : '#00FF00')}}"
}
```

### Steering Wheel Rotation

```json
{
  "rotation": "{{steering.angle | clamp:-540,540}}"
}
```

### Compass Needle Rotation

```json
{
  "rotation": "{{-heading.degrees}}"
}
```

### G-Force Dot Position

```json
{
  "x": "{{50 + (gforce.x | clamp:-1,1 | map:-1,1,-40,40)}}%",
  "y": "{{50 - (gforce.y | clamp:-1,1 | map:-1,1,-40,40)}}%"
}
```

---

## Security Limits

Expressions are evaluated in a sandboxed environment with strict limits to prevent abuse.

### Expression Length

- **Maximum length**: 500 characters per expression
- Expressions exceeding this limit will be truncated and may produce errors

### Nesting Depth

- **Maximum nesting**: 10 levels
- Deeply nested ternary operators or function calls beyond this limit will fail

### Prohibited Operations

The following are **NOT** allowed in expressions:

| Prohibited | Reason |
|------------|--------|
| Loops (`for`, `while`) | Prevents infinite loops |
| Function definitions | No custom function creation |
| Variable declarations | Expressions are stateless |
| Object/array literals | Use predefined data paths |
| Regular expressions | Security risk |
| String interpolation | Use concatenation instead |
| Comments | Not supported in expressions |

### Whitelisted Operators Only

Only the operators documented above are permitted. Any unrecognized operators will cause the expression to fail.

### Error Handling

When an expression fails to evaluate:
1. The widget falls back to the `default` value if specified
2. If no default, displays "--" or empty string
3. Error is logged for debugging (debug builds only)

---

## Best Practices

### 1. Always Provide Defaults

```json
{
  "content": "{{speed.formatted ?? '--'}} {{speed.unit ?? 'mph'}}"
}
```

### 2. Keep Expressions Simple

**Good:**
```json
{
  "color": "{{autopilot.active ? '#00FF00' : '#FFFFFF'}}"
}
```

**Avoid:**
```json
{
  "color": "{{(autopilot.state == 'fsd' && speed.value > 0) ? (battery.percent > 50 ? '#0066FF' : '#003399') : ((autopilot.state == 'autosteer' && !brake.applied) ? '#00FF00' : (autopilot.state == 'tacc' ? (speed.value > 60 ? '#FF9900' : '#FFCC00') : '#FFFFFF'))}}"
}
```

### 3. Use Appropriate Precision

```json
{
  "content": "{{speed.value | round:0}}",
  "latitude": "{{location.latitude | round:6}}"
}
```

### 4. Validate Ranges

```json
{
  "height": "{{throttle.percent | clamp:0,100}}%",
  "rotation": "{{steering.angle | clamp:-540,540}}"
}
```

### 5. Test Edge Cases

Always verify expressions handle:
- Nil/missing data
- Zero values
- Negative numbers
- Maximum values

---

## Debugging Expressions

### Console Logging (Debug Builds)

In debug builds, expression evaluation errors are logged to the console:

```
[Expression] Error evaluating "{{speed.valuee}}": Unknown path "speed.valuee"
[Expression] Error evaluating "{{1/0}}": Division by zero
```

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Unknown path" | Typo in data path | Check spelling, verify path exists |
| "Type mismatch" | Wrong type for operator | Ensure types match (number vs string) |
| "Division by zero" | Dividing by zero | Add guard: `{{x != 0 ? y/x : 0}}` |
| "Expression too long" | Over 500 characters | Split into multiple expressions |
| "Max depth exceeded" | Too many nested operations | Simplify expression |
