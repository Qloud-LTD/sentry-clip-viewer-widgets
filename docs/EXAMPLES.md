# Widget SDK Examples

This document provides complete, working JSON examples for common widget patterns. Each example demonstrates a specific technique or pattern you can use when creating custom widgets.

## Table of Contents

1. [Simple Text Widget](#1-simple-text-widget) - Speed display with nil coalescing
2. [Conditional Color Widget](#2-conditional-color-widget) - Gear indicator with color based on state
3. [Icon with Rotation](#3-icon-with-rotation) - Steering wheel that rotates with angle
4. [Vertical Bar Widget](#4-vertical-bar-widget) - Throttle bar with dynamic fill height
5. [Circular Gauge](#5-circular-gauge) - Arc-based speedometer with needle
6. [Compound Layout](#6-compound-layout) - HStack/VStack combining multiple elements
7. [Widget Composition](#7-widget-composition) - Using "widget" element to embed other widgets
8. [Map Widget](#8-map-widget) - Map element with configuration options

---

## 1. Simple Text Widget

The simplest widget type displays telemetry data as text. This example shows a speed display with nil coalescing to handle missing data.

**Techniques demonstrated:**
- Basic text element
- `{{data ?? fallback}}` nil coalescing syntax
- Percentage-based sizing (`60%h` = 60% of widget height)
- Font styling (weight, design)
- Anchor positioning

```json
{
  "$schema": "widget-v1.json",
  "id": "speed.digital",
  "version": "1.0.0",
  "category": "speed",
  "name": "Digital Speed",
  "description": "Large digital speed display",
  "author": "Your Name",

  "canvas": {
    "minWidth": 0.06,
    "minHeight": 0.06
  },

  "elements": [
    {
      "type": "text",
      "content": "{{speed.formatted ?? '--'}}",
      "position": { "x": "50%", "y": "50%" },
      "anchor": "center",
      "style": {
        "fontSize": "60%h",
        "fontWeight": "bold",
        "fontDesign": "rounded",
        "color": "#FFFFFF",
        "minimumScaleFactor": 0.5
      }
    }
  ]
}
```

**Key Points:**
- `{{speed.formatted ?? '--'}}` - If `speed.formatted` is nil/unavailable, display `--`
- `fontSize: "60%h"` - Font size is 60% of the widget's height
- `anchor: "center"` - The text is centered on the position point
- `minimumScaleFactor: 0.5` - Text can shrink to 50% if needed to fit

---

## 2. Conditional Color Widget

Use conditional bindings to change colors based on data values. This gear indicator shows different colors for each gear.

**Techniques demonstrated:**
- Pre-computed bindings with conditions
- Circle element with dynamic fill
- Layered elements (circle behind text)
- Fixed aspect ratio canvas

```json
{
  "$schema": "widget-v1.json",
  "id": "gear.colored",
  "version": "1.0.0",
  "category": "gear",
  "name": "Gear Colored Circle",
  "description": "Gear letter in a color-coded circle",
  "author": "Your Name",

  "canvas": {
    "aspectRatio": 1.0,
    "minWidth": 0.04,
    "minHeight": 0.04
  },

  "bindings": {
    "circleColor": {
      "conditions": [
        { "when": "{{gear.letter == 'D'}}", "value": "#00CC00" },
        { "when": "{{gear.letter == 'R'}}", "value": "#FF3B30" },
        { "when": "{{gear.letter == 'P'}}", "value": "#007AFF" },
        { "when": "{{gear.letter == 'N'}}", "value": "#FF9500" }
      ],
      "default": "#8E8E93"
    }
  },

  "elements": [
    {
      "type": "circle",
      "radius": "40%",
      "position": { "x": "50%", "y": "50%" },
      "fill": "{{circleColor}}"
    },
    {
      "type": "text",
      "content": "{{gear.letter ?? '-'}}",
      "position": { "x": "50%", "y": "50%" },
      "anchor": "center",
      "style": {
        "fontSize": "40%h",
        "fontWeight": "bold",
        "fontDesign": "rounded",
        "color": "#FFFFFF"
      }
    }
  ]
}
```

**Key Points:**
- `bindings` section pre-computes values based on conditions
- Conditions are evaluated in order; first match wins
- `{{circleColor}}` references the computed binding
- Elements are drawn in order (circle first, then text on top)

---

## 3. Icon with Rotation

SF Symbols can be rotated dynamically based on telemetry data. This steering wheel rotates with the actual steering angle.

**Techniques demonstrated:**
- Icon element with SF Symbol
- Dynamic rotation from data
- Autopilot-aware coloring
- Glow/shadow effects

```json
{
  "$schema": "widget-v1.json",
  "id": "steering.wheel",
  "version": "1.0.0",
  "category": "steering",
  "name": "Steering Wheel",
  "description": "Rotating steering wheel icon with autopilot color",
  "author": "Your Name",

  "canvas": {
    "aspectRatio": 1.0,
    "minWidth": 0.05,
    "minHeight": 0.05
  },

  "elements": [
    {
      "type": "icon",
      "name": "steeringwheel",
      "size": "70%h",
      "position": { "x": "50%", "y": "50%" },
      "anchor": "center",
      "color": "{{autopilot.color}}",
      "rotation": "{{steering.angle}}",
      "shadow": {
        "color": "{{autopilot.color}}",
        "opacity": 0.5,
        "radius": 2
      }
    }
  ]
}
```

**Key Points:**
- `name: "steeringwheel"` - Uses SF Symbol "steeringwheel"
- `rotation: "{{steering.angle}}"` - Rotates with steering angle in degrees
- `color: "{{autopilot.color}}"` - Built-in binding for autopilot state color
- `shadow` creates a glow effect matching the autopilot color

---

## 4. Vertical Bar Widget

A dynamic bar that fills based on throttle percentage. Changes color based on throttle/brake state.

**Techniques demonstrated:**
- Complex conditional bindings
- Stacked layers with ZStack group
- Dynamic height from percentage
- Conditional glow effects
- Multiple binding references

```json
{
  "$schema": "widget-v1.json",
  "id": "throttle.combined",
  "version": "1.0.0",
  "category": "throttle",
  "name": "Combined Throttle/Brake Bar",
  "description": "Single vertical bar showing throttle (green) or brake (red)",
  "author": "Your Name",

  "canvas": {
    "aspectRatio": 0.4,
    "minWidth": 0.025,
    "minHeight": 0.08
  },

  "bindings": {
    "fillColor": {
      "conditions": [
        { "when": "{{brake.applied && throttle.percent > 0}}", "value": "#FFCC00" },
        { "when": "{{brake.applied}}", "value": "#FF3B30" },
        { "when": "{{throttle.percent > 0}}", "value": "#34C759" }
      ],
      "default": "#8E8E93"
    },
    "fillHeight": {
      "conditions": [
        { "when": "{{brake.applied}}", "value": "100%" }
      ],
      "default": "{{throttle.percent}}%"
    },
    "displayText": {
      "conditions": [
        { "when": "{{brake.applied && throttle.percent <= 0}}", "value": "B" },
        { "when": "{{throttle.percent >= 100}}", "value": "F" },
        { "when": "{{throttle.percent > 0 || brake.applied}}", "value": "{{throttle.percent | int}}" }
      ],
      "default": ""
    },
    "glowEnabled": {
      "conditions": [
        { "when": "{{throttle.percent >= 100 && !brake.applied}}", "value": true }
      ],
      "default": false
    }
  },

  "elements": [
    {
      "type": "group",
      "layout": "zstack",
      "alignment": "bottom",
      "children": [
        {
          "type": "rect",
          "size": { "width": "70%", "height": "85%" },
          "position": { "x": "50%", "y": "50%" },
          "anchor": "center",
          "cornerRadius": "8%w",
          "fill": "rgba(128, 128, 128, 0.3)"
        },
        {
          "type": "rect",
          "size": { "width": "70%", "height": "{{fillHeight}}" },
          "maxHeight": "85%",
          "position": { "x": "50%", "y": "92.5%" },
          "anchor": "bottomCenter",
          "cornerRadius": "8%w",
          "fill": "{{fillColor}}",
          "shadow": {
            "enabled": "{{glowEnabled}}",
            "color": "#34C759",
            "radius": "15%w",
            "opacity": 0.8
          }
        },
        {
          "type": "text",
          "content": "{{displayText}}",
          "position": { "x": "50%", "y": "12%" },
          "anchor": "topCenter",
          "style": {
            "fontSize": "28%w",
            "fontWeight": "bold",
            "fontDesign": "rounded",
            "color": "#FFFFFF",
            "shadow": {
              "color": "#000000",
              "radius": 1,
              "y": 1
            }
          }
        }
      ]
    }
  ]
}
```

**Key Points:**
- `layout: "zstack"` - Layers children on top of each other
- `anchor: "bottomCenter"` - Fill bar grows from the bottom
- `{{throttle.percent | int}}` - Pipe filter to format as integer
- `shadow.enabled` can be conditionally controlled
- Use `&&`, `||`, `!` for boolean logic in conditions

---

## 5. Circular Gauge

An analog-style gauge with arc, tickmarks, and rotating needle.

**Techniques demonstrated:**
- Arc element for dial background
- Tickmarks element for scale markers
- Needle element that rotates based on value
- Configurable options (maxSpeed)
- Multiple layered elements

```json
{
  "$schema": "widget-v1.json",
  "id": "speed.analog",
  "version": "1.0.0",
  "category": "speed",
  "name": "Analog Gauge",
  "description": "Circular analog speedometer with needle",
  "author": "Your Name",

  "canvas": {
    "aspectRatio": 1.0,
    "minWidth": 0.10,
    "minHeight": 0.10
  },

  "options": {
    "maxSpeed": {
      "type": "number",
      "default": 160,
      "label": "Maximum Speed",
      "description": "Maximum speed value on the gauge"
    }
  },

  "elements": [
    {
      "type": "arc",
      "id": "dialBackground",
      "center": { "x": "50%", "y": "50%" },
      "radius": "46%",
      "startAngle": 135,
      "endAngle": 405,
      "stroke": "rgba(255,255,255,0.3)",
      "strokeWidth": "2%"
    },
    {
      "type": "tickmarks",
      "id": "ticks",
      "center": { "x": "50%", "y": "50%" },
      "innerRadius": "34%",
      "outerRadius": "44%",
      "startAngle": 135,
      "endAngle": 405,
      "count": 17,
      "majorEvery": 4,
      "majorWidth": "2%",
      "minorWidth": "1%",
      "color": "rgba(255,255,255,0.7)"
    },
    {
      "type": "needle",
      "id": "speedNeedle",
      "center": { "x": "50%", "y": "50%" },
      "length": "38%",
      "width": "3%",
      "color": "#FF3B30",
      "value": "{{speed.value ?? 0}}",
      "minValue": 0,
      "maxValue": "{{options.maxSpeed ?? 160}}",
      "startAngle": 135,
      "endAngle": 405
    },
    {
      "type": "circle",
      "id": "centerCap",
      "radius": "5%",
      "position": { "x": "50%", "y": "50%" },
      "fill": "#FFFFFF"
    },
    {
      "type": "text",
      "id": "speedReadout",
      "content": "{{speed.formatted ?? '--'}}",
      "position": { "x": "50%", "y": "72%" },
      "anchor": "center",
      "style": {
        "fontSize": "15%h",
        "fontWeight": "bold",
        "fontDesign": "rounded",
        "color": "#FFFFFF"
      }
    },
    {
      "type": "text",
      "id": "unitLabel",
      "content": "{{speed.unit}}",
      "position": { "x": "50%", "y": "82%" },
      "anchor": "center",
      "style": {
        "fontSize": "8%h",
        "fontWeight": "medium",
        "color": "rgba(255,255,255,0.7)"
      }
    }
  ]
}
```

**Key Points:**
- Angles use 0 = right, 90 = down convention (clockwise)
- 135 to 405 creates a 270-degree arc (typical gauge sweep)
- `needle` element automatically maps value to angle range
- `options` section defines user-configurable settings
- `{{options.maxSpeed ?? 160}}` accesses the option with fallback

---

## 6. Compound Layout

Use HStack and VStack groups to arrange multiple elements in rows and columns.

**Techniques demonstrated:**
- HStack horizontal layout
- VStack vertical layout
- Nested layout groups
- Spacing and alignment
- Relative sizing within groups

```json
{
  "$schema": "widget-v1.json",
  "id": "info.compact",
  "version": "1.0.0",
  "category": "info",
  "name": "Compact Info Panel",
  "description": "Compact info panel with speed, gear, and compass",
  "author": "Your Name",

  "canvas": {
    "minWidth": 0.20,
    "minHeight": 0.08,
    "backgroundColor": "rgba(0,0,0,0.5)",
    "cornerRadius": "8"
  },

  "elements": [
    {
      "type": "group",
      "layout": "hstack",
      "spacing": "5%w",
      "alignment": "center",
      "padding": { "horizontal": "4%w", "vertical": "4%h" },
      "size": { "width": "100%", "height": "100%" },
      "children": [
        {
          "type": "group",
          "layout": "vstack",
          "spacing": "2%h",
          "alignment": "leading",
          "size": { "width": "40%", "height": "100%" },
          "children": [
            {
              "type": "text",
              "content": "{{speed.formatted ?? '--'}}",
              "style": {
                "fontSize": "45%h",
                "fontWeight": "bold",
                "fontDesign": "rounded",
                "color": "#FFFFFF"
              }
            },
            {
              "type": "text",
              "content": "{{speed.unit}}",
              "style": {
                "fontSize": "20%h",
                "fontWeight": "medium",
                "color": "rgba(255,255,255,0.7)"
              }
            }
          ]
        },
        {
          "type": "text",
          "content": "{{gear.letter ?? '-'}}",
          "style": {
            "fontSize": "50%h",
            "fontWeight": "bold",
            "color": "{{gear.color}}"
          }
        },
        {
          "type": "group",
          "layout": "vstack",
          "spacing": "2%h",
          "alignment": "trailing",
          "size": { "width": "30%", "height": "100%" },
          "children": [
            {
              "type": "icon",
              "name": "location.north.fill",
              "size": "25%h",
              "color": "#00D4FF",
              "rotation": "{{compass.heading}}"
            },
            {
              "type": "text",
              "content": "{{compass.cardinal ?? 'N'}}",
              "style": {
                "fontSize": "18%h",
                "fontWeight": "medium",
                "color": "#00D4FF"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

**Key Points:**
- `layout: "hstack"` arranges children horizontally
- `layout: "vstack"` arranges children vertically
- `spacing` controls gap between children
- `alignment` controls cross-axis alignment: `leading`, `center`, `trailing`
- `padding` adds internal spacing around group content
- Groups can be nested for complex layouts

---

## 7. Widget Composition

The most powerful pattern: embed existing widgets inside a compound widget using the `widget` element type.

**Techniques demonstrated:**
- `type: "widget"` element
- Reference widgets by ID
- Size allocation for embedded widgets
- Mixed layouts (hstack with vstack inside)

```json
{
  "$schema": "widget-v1.json",
  "id": "compound.teslaHUD",
  "version": "1.0.0",
  "category": "compound",
  "name": "Tesla HUD",
  "description": "Standard Tesla-style HUD with speed, gear, and autopilot",
  "author": "Your Name",

  "canvas": {
    "minWidth": 0.30,
    "minHeight": 0.06
  },

  "elements": [
    {
      "type": "group",
      "layout": "hstack",
      "spacing": "4%w",
      "alignment": "center",
      "size": { "width": "100%", "height": "100%" },
      "children": [
        {
          "type": "widget",
          "ref": "gear.gearColored",
          "size": { "width": "10%", "height": "80%" }
        },
        {
          "type": "widget",
          "ref": "turnSignal.turnSignalArrows",
          "size": { "width": "12%", "height": "50%" }
        },
        {
          "type": "widget",
          "ref": "speed.speedDigital",
          "size": { "width": "30%", "height": "100%" }
        },
        {
          "type": "widget",
          "ref": "autopilot.autopilotDot",
          "size": { "width": "8%", "height": "50%" }
        },
        {
          "type": "widget",
          "ref": "steering.steeringWheel",
          "size": { "width": "12%", "height": "80%" }
        }
      ]
    }
  ]
}
```

**Key Points:**
- `type: "widget"` embeds another widget definition
- `ref` uses the widget's `id` field (e.g., `gear.gearColored`)
- `size` controls how much space the widget occupies
- The referenced widget scales to fit the allocated space
- Options from the embedded widget can be overridden

### Advanced Composition Example

Nested layouts with widget composition:

```json
{
  "$schema": "widget-v1.json",
  "id": "compound.racingHUD",
  "version": "1.0.0",
  "category": "compound",
  "name": "Racing HUD",
  "description": "Racing-style HUD with speed, g-force, and throttle/brake",
  "author": "Your Name",

  "canvas": {
    "minWidth": 0.35,
    "minHeight": 0.10
  },

  "elements": [
    {
      "type": "group",
      "layout": "hstack",
      "spacing": "5%w",
      "alignment": "center",
      "size": { "width": "100%", "height": "100%" },
      "children": [
        {
          "type": "widget",
          "ref": "gForce.gForceCircle",
          "size": { "width": "25%", "height": "90%" }
        },
        {
          "type": "group",
          "layout": "vstack",
          "spacing": "4%h",
          "alignment": "center",
          "size": { "width": "35%", "height": "100%" },
          "children": [
            {
              "type": "widget",
              "ref": "speed.speedDigital",
              "size": { "width": "100%", "height": "65%" }
            },
            {
              "type": "widget",
              "ref": "gear.gearLetter",
              "size": { "width": "40%", "height": "30%" }
            }
          ]
        },
        {
          "type": "widget",
          "ref": "throttle.throttleSplit",
          "size": { "width": "15%", "height": "90%" }
        }
      ]
    }
  ]
}
```

---

## 8. Map Widget

The map element displays an Apple Maps view with the vehicle's current position.

**Techniques demonstrated:**
- Map element type
- Style options (standard, satellite, dark, hybrid)
- User-configurable options
- Corner radius for rounded appearance

```json
{
  "$schema": "widget-v1.json",
  "id": "map.standard",
  "version": "1.0.0",
  "category": "map",
  "name": "Standard Map",
  "description": "Standard Apple Maps view with vehicle position marker",
  "author": "Your Name",

  "canvas": {
    "aspectRatio": 1.0,
    "minWidth": 0.1,
    "minHeight": 0.1
  },

  "options": {
    "zoomLevel": {
      "type": "enum",
      "values": ["street", "neighborhood", "city"],
      "default": "neighborhood",
      "label": "Zoom Level",
      "description": "Map zoom level"
    },
    "showsUserLocation": {
      "type": "boolean",
      "default": true,
      "label": "Show Location Marker",
      "description": "Display marker at vehicle position"
    },
    "showsHeading": {
      "type": "boolean",
      "default": true,
      "label": "Show Heading",
      "description": "Display heading direction indicator"
    }
  },

  "elements": [
    {
      "type": "map",
      "id": "mainMap",
      "style": "standard",
      "position": { "x": "50%", "y": "50%" },
      "size": { "width": "100%", "height": "100%" },
      "cornerRadius": "5%",
      "options": {
        "showsUserLocation": "{{options.showsUserLocation ?? true}}",
        "showsHeading": "{{options.showsHeading ?? true}}",
        "zoomLevel": "{{options.zoomLevel ?? 'neighborhood'}}"
      }
    }
  ]
}
```

**Key Points:**
- `style` can be: `standard`, `satellite`, `dark`, `hybrid`
- `zoomLevel` controls the map scale
- Map automatically centers on GPS coordinates from telemetry
- Heading indicator shows vehicle direction
- Maps are cached for performance during export

### Dark Map Variant

```json
{
  "$schema": "widget-v1.json",
  "id": "map.dark",
  "version": "1.0.0",
  "category": "map",
  "name": "Dark Map",
  "description": "Dark-themed map for better visibility on night footage",
  "author": "Your Name",

  "canvas": {
    "aspectRatio": 1.0,
    "minWidth": 0.1,
    "minHeight": 0.1
  },

  "elements": [
    {
      "type": "map",
      "id": "mainMap",
      "style": "dark",
      "position": { "x": "50%", "y": "50%" },
      "size": { "width": "100%", "height": "100%" },
      "cornerRadius": "8%",
      "options": {
        "showsUserLocation": true,
        "showsHeading": true,
        "zoomLevel": "neighborhood"
      }
    }
  ]
}
```

---

## Available Data Bindings

Here are the telemetry data bindings you can use:

### Speed
- `{{speed.value}}` - Speed in m/s (raw)
- `{{speed.formatted}}` - Formatted with user's preferred unit
- `{{speed.unit}}` - "MPH" or "KM/H"

### Gear
- `{{gear.letter}}` - "P", "D", "R", or "N"
- `{{gear.color}}` - Color based on gear state

### Steering
- `{{steering.angle}}` - Steering angle in degrees (-450 to +450)

### Throttle/Brake
- `{{throttle.percent}}` - Throttle position 0-100
- `{{brake.applied}}` - Boolean, true if braking

### Autopilot
- `{{autopilot.state}}` - "fsd", "autosteer", "tacc", "none"
- `{{autopilot.color}}` - Color based on state (blue, green, orange, white)

### Turn Signals
- `{{turnSignal.left}}` - Boolean
- `{{turnSignal.right}}` - Boolean

### Compass
- `{{compass.heading}}` - Heading in degrees (0-360)
- `{{compass.cardinal}}` - "N", "NE", "E", etc.

### G-Force
- `{{gforce.x}}` - Lateral acceleration (g)
- `{{gforce.y}}` - Longitudinal acceleration (g)
- `{{gforce.z}}` - Vertical acceleration (g)
- `{{gforce.magnitude}}` - Combined magnitude

### GPS
- `{{gps.latitude}}` - Latitude in degrees
- `{{gps.longitude}}` - Longitude in degrees

### Timestamp
- `{{timestamp.formatted}}` - Formatted timestamp
- `{{timestamp.date}}` - Date portion
- `{{timestamp.time}}` - Time portion

---

## Binding Filters

Format data using pipe filters:

```
{{value | filter}}
{{value | filter:'format'}}
```

Available filters:
- `int` - Round to integer
- `format:'%.1f'` - Printf-style formatting
- `abs` - Absolute value
- `upper` - Uppercase string
- `lower` - Lowercase string

Examples:
```
{{speed.value | int}}           → "65"
{{gforce.x | format:'%.2f'}}    → "0.35"
{{throttle.percent | int}}      → "75"
```

---

## Best Practices

1. **Use semantic IDs**: `category.widgetName` format (e.g., `speed.digital`)

2. **Provide defaults**: Always use nil coalescing `{{data ?? fallback}}`

3. **Set minimum sizes**: Ensure widgets don't become too small to read

4. **Use percentage units**: Makes widgets scale properly at different sizes

5. **Layer elements correctly**: Background elements first, foreground last

6. **Test at multiple sizes**: Verify widgets look good small and large

7. **Document options**: Include clear labels and descriptions

8. **Prefer composition**: Reuse existing widgets via `type: "widget"`
