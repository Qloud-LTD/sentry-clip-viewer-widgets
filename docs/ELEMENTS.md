# Widget SDK Element Types Reference

This document describes all available element types that can be used in widget definitions. Each element type has specific properties for positioning, styling, and rendering.

## Overview

Widgets are composed of elements arranged in a tree structure. Elements can be:
- **Primitives**: text, icon, rect, circle, arc, path
- **Containers**: group (with layout modes)
- **References**: widget (embeds another widget)
- **Special**: map (interactive map view)

---

## Common Properties

These properties are available on all element types:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `id` | String | auto | Unique identifier for the element |
| `visible` | Bool/Expression | `true` | Whether element is rendered |
| `opacity` | Double/Expression | `1.0` | Element opacity (0-1) |
| `rotation` | Double/Expression | `0` | Rotation in degrees |
| `scale` | Double/Expression | `1.0` | Scale factor |

---

## text

Renders text content with styling options.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `content` | String/Expression | required | Text to display |
| `position` | Position | `{x: 0, y: 0}` | Text position |
| `anchor` | Anchor | `topLeading` | Alignment anchor point |
| `style` | TextStyle | {} | Text styling options |

### TextStyle Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `fontSize` | Double/Expression | `17` | Font size in points |
| `fontWeight` | String | `regular` | Weight: ultraLight, thin, light, regular, medium, semibold, bold, heavy, black |
| `fontDesign` | String | `default` | Design: default, rounded, serif, monospaced |
| `color` | Color/Expression | `#FFFFFF` | Text color |
| `minimumScaleFactor` | Double | `1.0` | Min scale for text fitting (0-1) |
| `lineLimit` | Int | `nil` | Maximum number of lines |
| `alignment` | String | `leading` | Text alignment: leading, center, trailing |

### Examples

**Basic Text:**
```json
{
  "type": "text",
  "content": "{{speed.formatted ?? '--'}}",
  "position": { "x": 50, "y": 20 },
  "style": {
    "fontSize": 48,
    "fontWeight": "bold",
    "color": "#FFFFFF"
  }
}
```

**Speed Display with Unit:**
```json
{
  "type": "text",
  "content": "{{speed.formatted ?? '--'}} {{speed.unit}}",
  "anchor": "center",
  "style": {
    "fontSize": "{{size.height * 0.6}}",
    "fontWeight": "bold",
    "fontDesign": "rounded",
    "color": "{{autopilot.active ? '#00FF00' : '#FFFFFF'}}",
    "minimumScaleFactor": 0.5
  }
}
```

**Multi-line Location:**
```json
{
  "type": "text",
  "content": "{{location.name ?? 'Unknown'}}\n{{time.formatted}}",
  "style": {
    "fontSize": 14,
    "fontWeight": "medium",
    "color": "#CCCCCC",
    "lineLimit": 2,
    "alignment": "leading"
  }
}
```

---

## icon

Renders SF Symbol icons with styling.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | String/Expression | required | SF Symbol name |
| `size` | Double/Expression | `24` | Icon size in points |
| `color` | Color/Expression | `#FFFFFF` | Icon color |
| `rotation` | Double/Expression | `0` | Rotation in degrees |
| `weight` | String | `regular` | Symbol weight |
| `renderingMode` | String | `monochrome` | Mode: monochrome, hierarchical, palette, multicolor |

### Examples

**Basic Icon:**
```json
{
  "type": "icon",
  "name": "speedometer",
  "size": 32,
  "color": "#FFFFFF"
}
```

**Steering Wheel with Rotation:**
```json
{
  "type": "icon",
  "name": "steeringwheel",
  "size": "{{size.width * 0.8}}",
  "color": "{{autopilot.color}}",
  "rotation": "{{steering.angle | clamp:-180,180}}"
}
```

**Turn Signal Arrow:**
```json
{
  "type": "icon",
  "name": "arrowtriangle.left.fill",
  "size": 24,
  "color": "{{signals.left ? '#00FF00' : '#333333'}}",
  "visible": true
}
```

**Gear Indicator:**
```json
{
  "type": "icon",
  "name": "{{gear.state == 'P' ? 'p.circle.fill' : (gear.state == 'R' ? 'r.circle.fill' : (gear.state == 'N' ? 'n.circle.fill' : 'd.circle.fill'))}}",
  "size": 28,
  "color": "{{gear.color}}"
}
```

---

## rect

Renders a rectangle shape.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `size` | Size | required | Width and height |
| `position` | Position | `{x: 0, y: 0}` | Rectangle position |
| `fill` | Color/Expression | `nil` | Fill color |
| `stroke` | Color/Expression | `nil` | Stroke color |
| `strokeWidth` | Double/Expression | `1` | Stroke line width |
| `cornerRadius` | Double/Expression | `0` | Corner radius |

### Examples

**Background Box:**
```json
{
  "type": "rect",
  "size": { "width": 100, "height": 40 },
  "fill": "#000000",
  "opacity": 0.6,
  "cornerRadius": 8
}
```

**Throttle Bar Background:**
```json
{
  "type": "rect",
  "size": { "width": 28, "height": 80 },
  "fill": "#333333",
  "cornerRadius": 4
}
```

**Throttle Bar Fill:**
```json
{
  "type": "rect",
  "size": {
    "width": 24,
    "height": "{{throttle.percent | clamp:0,100 | map:0,100,0,76}}"
  },
  "position": { "x": 2, "y": "{{78 - (throttle.percent | clamp:0,100 | map:0,100,0,76)}}" },
  "fill": "{{brake.applied ? '#FF0000' : '#00FF00'}}",
  "cornerRadius": 2
}
```

**Border Only:**
```json
{
  "type": "rect",
  "size": { "width": 120, "height": 60 },
  "stroke": "#FFFFFF",
  "strokeWidth": 2,
  "cornerRadius": 12,
  "fill": null
}
```

---

## circle

Renders a circle shape.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `radius` | Double/Expression | required | Circle radius |
| `center` | Position | `{x: 0, y: 0}` | Center position (alternative to position) |
| `position` | Position | `nil` | Top-left position |
| `fill` | Color/Expression | `nil` | Fill color |
| `stroke` | Color/Expression | `nil` | Stroke color |
| `strokeWidth` | Double/Expression | `1` | Stroke line width |

### Examples

**G-Force Background:**
```json
{
  "type": "circle",
  "radius": 40,
  "center": { "x": 50, "y": 50 },
  "fill": "#000000",
  "opacity": 0.5,
  "stroke": "#FFFFFF",
  "strokeWidth": 1.5
}
```

**G-Force Dot:**
```json
{
  "type": "circle",
  "radius": 6,
  "center": {
    "x": "{{50 + (gforce.x | clamp:-1,1) * 35}}",
    "y": "{{50 - (gforce.y | clamp:-1,1) * 35}}"
  },
  "fill": "#FF0000"
}
```

**Autopilot Status Dot:**
```json
{
  "type": "circle",
  "radius": 8,
  "fill": "{{autopilot.color}}",
  "stroke": "#FFFFFF",
  "strokeWidth": 1
}
```

---

## arc

Renders an arc or partial circle.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `center` | Position | required | Arc center point |
| `radius` | Double/Expression | required | Arc radius |
| `startAngle` | Double/Expression | `0` | Start angle in degrees |
| `endAngle` | Double/Expression | `360` | End angle in degrees |
| `clockwise` | Bool | `true` | Draw direction |
| `fill` | Color/Expression | `nil` | Fill color |
| `stroke` | Color/Expression | `nil` | Stroke color |
| `strokeWidth` | Double/Expression | `1` | Stroke line width |
| `lineCap` | String | `butt` | Line cap: butt, round, square |

### Examples

**Speed Gauge Arc:**
```json
{
  "type": "arc",
  "center": { "x": 60, "y": 60 },
  "radius": 50,
  "startAngle": 135,
  "endAngle": 405,
  "stroke": "#333333",
  "strokeWidth": 8,
  "lineCap": "round"
}
```

**Speed Value Arc:**
```json
{
  "type": "arc",
  "center": { "x": 60, "y": 60 },
  "radius": 50,
  "startAngle": 135,
  "endAngle": "{{135 + (speed.value | clamp:0,160 | map:0,160,0,270)}}",
  "stroke": "#00FF00",
  "strokeWidth": 8,
  "lineCap": "round"
}
```

**Compass Background:**
```json
{
  "type": "arc",
  "center": { "x": 25, "y": 25 },
  "radius": 22,
  "startAngle": 0,
  "endAngle": 360,
  "stroke": "#666666",
  "strokeWidth": 2
}
```

---

## path

Renders a custom SVG path.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `d` | String | required | SVG path data |
| `fill` | Color/Expression | `nil` | Fill color |
| `stroke` | Color/Expression | `nil` | Stroke color |
| `strokeWidth` | Double/Expression | `1` | Stroke line width |
| `fillRule` | String | `nonZero` | Fill rule: nonZero, evenOdd |
| `lineCap` | String | `butt` | Line cap: butt, round, square |
| `lineJoin` | String | `miter` | Line join: miter, round, bevel |

### SVG Path Commands

| Command | Parameters | Description |
|---------|------------|-------------|
| `M` | x y | Move to (absolute) |
| `m` | dx dy | Move to (relative) |
| `L` | x y | Line to (absolute) |
| `l` | dx dy | Line to (relative) |
| `H` | x | Horizontal line (absolute) |
| `h` | dx | Horizontal line (relative) |
| `V` | y | Vertical line (absolute) |
| `v` | dy | Vertical line (relative) |
| `C` | x1 y1 x2 y2 x y | Cubic bezier |
| `S` | x2 y2 x y | Smooth cubic |
| `Q` | x1 y1 x y | Quadratic bezier |
| `T` | x y | Smooth quadratic |
| `A` | rx ry rot large sweep x y | Arc |
| `Z` | - | Close path |

### Examples

**Triangle (Compass North):**
```json
{
  "type": "path",
  "d": "M 25 5 L 30 20 L 20 20 Z",
  "fill": "#FF0000"
}
```

**Custom Needle:**
```json
{
  "type": "path",
  "d": "M 50 10 L 53 50 L 50 55 L 47 50 Z",
  "fill": "#FFFFFF",
  "rotation": "{{heading.degrees}}"
}
```

**Arrow Shape:**
```json
{
  "type": "path",
  "d": "M 0 10 L 20 10 L 20 5 L 30 15 L 20 25 L 20 20 L 0 20 Z",
  "fill": "{{signals.right ? '#00FF00' : '#333333'}}"
}
```

**Brake Disc Icon:**
```json
{
  "type": "path",
  "d": "M 20 10 A 10 10 0 1 1 20 10.001 M 15 12 L 15 18 M 20 11 L 20 19 M 25 12 L 25 18",
  "stroke": "#FFFFFF",
  "strokeWidth": 2,
  "fill": "{{brake.applied ? '#FF0000' : 'none'}}"
}
```

---

## group

Container that arranges child elements with layout.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `layout` | String | `zstack` | Layout mode: hstack, vstack, zstack |
| `spacing` | Double/Expression | `0` | Space between children |
| `alignment` | String | `center` | Child alignment |
| `padding` | Padding | `0` | Internal padding |
| `children` | Array | `[]` | Child elements |

### Alignment Values

**For hstack:**
- `top`, `center`, `bottom`

**For vstack:**
- `leading`, `center`, `trailing`

**For zstack:**
- `topLeading`, `top`, `topTrailing`
- `leading`, `center`, `trailing`
- `bottomLeading`, `bottom`, `bottomTrailing`

### Examples

**Horizontal Speed Display:**
```json
{
  "type": "group",
  "layout": "hstack",
  "spacing": 8,
  "alignment": "bottom",
  "children": [
    {
      "type": "text",
      "content": "{{speed.formatted ?? '--'}}",
      "style": { "fontSize": 48, "fontWeight": "bold" }
    },
    {
      "type": "text",
      "content": "{{speed.unit}}",
      "style": { "fontSize": 18, "color": "#AAAAAA" }
    }
  ]
}
```

**Vertical Info Stack:**
```json
{
  "type": "group",
  "layout": "vstack",
  "spacing": 4,
  "alignment": "leading",
  "padding": { "horizontal": 12, "vertical": 8 },
  "children": [
    {
      "type": "text",
      "content": "{{location.name ?? 'Unknown'}}",
      "style": { "fontSize": 14, "fontWeight": "semibold" }
    },
    {
      "type": "text",
      "content": "{{time.formatted}}",
      "style": { "fontSize": 12, "color": "#888888" }
    }
  ]
}
```

**Layered ZStack:**
```json
{
  "type": "group",
  "layout": "zstack",
  "alignment": "center",
  "children": [
    {
      "type": "circle",
      "radius": 40,
      "fill": "#000000",
      "opacity": 0.6
    },
    {
      "type": "icon",
      "name": "steeringwheel",
      "size": 50,
      "rotation": "{{steering.angle}}"
    }
  ]
}
```

**Tesla HUD Layout:**
```json
{
  "type": "group",
  "layout": "hstack",
  "spacing": 16,
  "alignment": "center",
  "padding": { "horizontal": 16, "vertical": 8 },
  "children": [
    { "type": "widget", "ref": "throttle-bar" },
    { "type": "widget", "ref": "gear-indicator" },
    { "type": "widget", "ref": "turn-signal-left" },
    { "type": "widget", "ref": "speed-display" },
    { "type": "widget", "ref": "turn-signal-right" },
    { "type": "widget", "ref": "compass" },
    { "type": "widget", "ref": "steering-wheel" }
  ]
}
```

---

## widget

Embeds another widget by reference.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ref` | String | required | Widget identifier |
| `size` | Size | widget default | Override widget size |
| `optionOverrides` | Object | `{}` | Override widget options |

### Examples

**Basic Widget Reference:**
```json
{
  "type": "widget",
  "ref": "speed-digital"
}
```

**Widget with Size Override:**
```json
{
  "type": "widget",
  "ref": "map-standard",
  "size": { "width": 200, "height": 150 }
}
```

**Widget with Option Overrides:**
```json
{
  "type": "widget",
  "ref": "map-standard",
  "size": { "width": 180, "height": 180 },
  "optionOverrides": {
    "mapStyle": "satellite",
    "zoomLevel": "street",
    "showsTrail": true
  }
}
```

**Compound Widget Assembly:**
```json
{
  "type": "group",
  "layout": "hstack",
  "spacing": 12,
  "children": [
    { "type": "widget", "ref": "battery-level" },
    { "type": "widget", "ref": "temperature-display" }
  ]
}
```

---

## map

Renders an interactive map view with vehicle position.

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `style` | String | `standard` | Map style: standard, satellite, dark, hybrid |
| `position` | Position | `{x: 0, y: 0}` | Map position |
| `size` | Size | required | Map dimensions |
| `options` | MapOptions | `{}` | Map configuration |

### MapOptions Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `showsUserLocation` | Bool | `true` | Show vehicle marker |
| `showsHeading` | Bool | `true` | Show heading indicator |
| `zoomLevel` | String | `neighborhood` | Zoom: street, neighborhood, city |
| `showsTrail` | Bool | `false` | Show breadcrumb trail |
| `trailLength` | Int | `50` | Number of trail points |
| `trailColor` | Color | `#00FF00` | Trail line color |
| `markerColor` | Color | `#FF0000` | Vehicle marker color |
| `rotateWithHeading` | Bool | `false` | Rotate map with vehicle |

### Examples

**Basic Map:**
```json
{
  "type": "map",
  "style": "standard",
  "size": { "width": 150, "height": 150 },
  "options": {
    "showsUserLocation": true,
    "showsHeading": true,
    "zoomLevel": "neighborhood"
  }
}
```

**Satellite Map with Trail:**
```json
{
  "type": "map",
  "style": "satellite",
  "size": { "width": 200, "height": 200 },
  "options": {
    "showsUserLocation": true,
    "showsHeading": true,
    "showsTrail": true,
    "trailLength": 100,
    "trailColor": "#00FFFF",
    "zoomLevel": "street"
  }
}
```

**Dark Map (Night Mode):**
```json
{
  "type": "map",
  "style": "dark",
  "size": { "width": 180, "height": 180 },
  "options": {
    "showsUserLocation": true,
    "showsHeading": true,
    "zoomLevel": "neighborhood",
    "markerColor": "#FFFFFF"
  }
}
```

**Mini Map (Corner Widget):**
```json
{
  "type": "map",
  "style": "standard",
  "position": { "x": 10, "y": 10 },
  "size": { "width": 120, "height": 120 },
  "options": {
    "showsUserLocation": true,
    "showsHeading": false,
    "zoomLevel": "city"
  }
}
```

---

## Position & Size Reference

### Position Object

```json
{
  "x": 100,
  "y": 50
}
```

Can use expressions:
```json
{
  "x": "{{size.width * 0.5}}",
  "y": "{{size.height * 0.1}}"
}
```

### Size Object

```json
{
  "width": 100,
  "height": 50
}
```

Can use expressions:
```json
{
  "width": "{{size.width * 0.3}}",
  "height": "{{size.height * 0.2}}"
}
```

### Padding Object

**Uniform:**
```json
{
  "padding": 8
}
```

**Asymmetric:**
```json
{
  "padding": {
    "top": 8,
    "bottom": 8,
    "leading": 12,
    "trailing": 12
  }
}
```

**Shorthand:**
```json
{
  "padding": {
    "horizontal": 12,
    "vertical": 8
  }
}
```

### Anchor Values

| Value | Description |
|-------|-------------|
| `topLeading` | Top-left corner |
| `top` | Top center |
| `topTrailing` | Top-right corner |
| `leading` | Center-left |
| `center` | Center |
| `trailing` | Center-right |
| `bottomLeading` | Bottom-left corner |
| `bottom` | Bottom center |
| `bottomTrailing` | Bottom-right corner |

---

## Color Reference

### Formats

| Format | Example | Description |
|--------|---------|-------------|
| Hex 6 | `#FF0000` | RGB (red) |
| Hex 8 | `#FF0000FF` | RGBA (red, opaque) |
| Hex 3 | `#F00` | Short RGB |
| Named | `red` | CSS color name |

### Expression Colors

```json
{
  "color": "{{autopilot.state == 'fsd' ? '#0066FF' : '#FFFFFF'}}"
}
```

### Opacity

Colors can have opacity applied separately:
```json
{
  "fill": "#000000",
  "opacity": 0.6
}
```

Or inline (8-digit hex):
```json
{
  "fill": "#00000099"
}
```

---

## Complete Widget Example

A full Tesla HUD widget definition:

```json
{
  "id": "tesla-hud",
  "name": "Tesla HUD",
  "version": "1.0",
  "minSize": { "width": 300, "height": 80 },
  "preferredAspectRatio": 4.0,
  "root": {
    "type": "group",
    "layout": "zstack",
    "children": [
      {
        "type": "rect",
        "size": { "width": "100%", "height": "100%" },
        "fill": "#000000",
        "opacity": 0.6,
        "cornerRadius": 12
      },
      {
        "type": "group",
        "layout": "hstack",
        "spacing": 16,
        "alignment": "center",
        "padding": { "horizontal": 16, "vertical": 8 },
        "children": [
          {
            "type": "widget",
            "ref": "throttle-combined"
          },
          {
            "type": "group",
            "layout": "zstack",
            "children": [
              {
                "type": "circle",
                "radius": 18,
                "fill": "{{gear.color}}",
                "opacity": 0.3
              },
              {
                "type": "text",
                "content": "{{gear.state ?? 'P'}}",
                "style": {
                  "fontSize": 20,
                  "fontWeight": "bold",
                  "color": "{{gear.color}}"
                }
              }
            ]
          },
          {
            "type": "icon",
            "name": "arrowtriangle.left.fill",
            "size": 20,
            "color": "{{signals.left ? '#00FF00' : '#333333'}}"
          },
          {
            "type": "group",
            "layout": "vstack",
            "spacing": 0,
            "alignment": "center",
            "children": [
              {
                "type": "text",
                "content": "{{speed.formatted ?? '--'}}",
                "style": {
                  "fontSize": 36,
                  "fontWeight": "bold",
                  "fontDesign": "rounded"
                }
              },
              {
                "type": "text",
                "content": "{{speed.unit ?? 'mph'}}",
                "style": {
                  "fontSize": 12,
                  "color": "#888888"
                }
              }
            ]
          },
          {
            "type": "icon",
            "name": "arrowtriangle.right.fill",
            "size": 20,
            "color": "{{signals.right ? '#00FF00' : '#333333'}}"
          },
          {
            "type": "widget",
            "ref": "compass-dial"
          },
          {
            "type": "icon",
            "name": "steeringwheel",
            "size": 32,
            "color": "{{autopilot.color}}",
            "rotation": "{{steering.angle | clamp:-180,180}}"
          }
        ]
      }
    ]
  }
}
```
