---
linktitle: Smooth-Step Gradient Modifier
title: Smooth-Step Gradient Modifier Component
description: Use the Smooth-Step Gradient Modifier component to generate a gradient with falloff in Open 3D Engine (O3DE).
---

The **Smooth-Step Gradient Modifier** component generates a gradient with falloff, smoothing the transition between high and low gradient values.

## Provider

[Gradient Signal Gem](/docs/user-guide/gems/reference/utility/gradient-signal)

## Smooth-Step Gradient Modifier properties

![Smooth-Step Gradient Modifier component properties](/images/user-guide/components/reference/gradient-modifiers/smooth-step-gradient-modifier-component.png)

| Property | Description | Values | Default |
|-|-|-|-|
| **Preview** | Displays the output gradient of this component after all properties are applied. | | |
| **Pin Preview to Shape** | Sets an entity with a compatible Shape component to use as the bounds of the preview if **Constrain to Shape** is `Enabled`. | EntityId | Current Entity |
| **Preview Position** | Sets the world location of the preview.<br> <br>*This field is available only if there is no entity selected in **Pin Preview to Shape**.* | Vector3: -Infinity to Infinity | X:`0.0`, Y:`0.0`, Z:`0.0` |
| **Preview Size** | Sets the dimensions of the preview. | Vector3: 0.0 to Infinity | X:`1.0`, Y:`1.0`, Z:`1.0` |
| **Constrain to Shape** | If `Enabled`, the gradient preview uses the bounds of the entity selected in **Pin Preview to Shape**.<br> <br>*This field is available only if an entity is selected in **Pin Preview to Shape**.* | Boolean | `Disabled` |
| **Falloff Midpoint** | Sets the midpoint of the falloff gradient. | Float: 0.0 - 1.0 | `0.5` |
| **Falloff Range** | Sets the maximum range of the falloff gradient. | Float: 0.0 - 1.0 | `0.5` |
| **Falloff Softness** | Sets the softness of the falloff gradient. | Float: 0.0 - 1.0 | `0.25` |
| **Gradient** | Refer to [Gradient properties](#gradient-properties) below. | | |

### Gradient properties

![Gradient properties](/images/user-guide/components/reference/vegetation-modifiers/gradient-properties.png)

| Property | Description | Values | Default |
|-|-|-|-|
| **Gradient Entity Id** | Sets an entity with an active **Gradient** component. | EntityId | None |
| **Opacity** | Sets the opacity of the input gradient. | Float: 0.0 - 1.0 | `1.0` |
| **Invert Input** | Inverts the values of the input gradient. | Boolean | `Disabled` |
| **Preview (Input)** | Displays the gradient provided by the entity set in **Gradient Entity Id**. |  |  |
| **Enable Transform** | If `Enabled`, the translation, scale, and rotation of the input gradient may be modified. | Boolean | `Disabled` |
| **Translate** | Sets the translation of the input gradient. | Vector3: -Infinity to Infinity | X:`0.0`, Y:`0.0`, Z:`0.0` |
| **Scale** | Sets the scale of the input gradient. | Vector3: 0.0001 to Infinity | X:`1.0`, Y:`1.0`, Z:`1.0` |
| **Rotate** | Sets the rotation of the input gradient. | Vector3: -Infinity to Infinity | X:`0.0`, Y:`0.0`, Z:`0.0` |
| **Enable Levels** | If `Enabled`, the input and output values of the input gradient may be modified. | Boolean | `Disabled` |
| **Input Mid** | Sets the median value of the input gradient. | Float: 0.0 - 1.0 | `1.0` |
| **Input Min** | Sets a minimum value for the input gradient. | Float: 0.0 - 1.0 | `0.0` |
| **Input Max** | Sets a maximum value for the input gradient. | Float: 0.0 - 1.0 | `1.0` |
| **Output Min** | Sets a minimum value for the output gradient. | Float: 0.0 - 1.0 | `0.0` |
| **Output Max** | Sets a maximum value for the output gradient. | Float: 0.0 - 1.0 | `1.0` |

## SmoothStepRequestBus

Use the following request functions with the `SmoothStepRequestBus` EBus interface to communicate with Smooth-Step Gradient Modifier components in your game.

| Method Name | Description | Parameter | Return | Scriptable |
|-|-|-|-|-|
| `GetFallOffMidpoint` | Returns the value of **Falloff Midpoint**. | None | Midpoint: Float | Yes |
| `GetFallOffRange` | Returns the value of **Falloff Range**. | None | Max Range: Float | Yes |
| `GetFallOffStrength` | Returns the value of **Falloff Softness**. | None | Strength: Float | Yes |
| `SetFallOffMidpoint` | Sets the value of **Falloff Midpoint**. | Midpoint: Float | None | Yes |
| `SetFallOffRange` | Sets the value of **Falloff Range**. | Max Range: Float | None | Yes |
| `SetFallOffStrength` | Sets the value of **Falloff Softness**. | Strength: Float | None | Yes |
