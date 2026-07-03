Package entry point: [README](../README.md)

---

<h1><img src="images/Icon.png" alt="Logo" width="80" align="middle" />&nbsp; Scene Validation - Built-in Rules</h1>

This page lists optional built-in `ValidationRule` checks shipped with this package. 

In case you do not want to apply these you can opt-out via the [Project Settings - Rules](ProjectSettings.md#rules) or [Project Settings - Pre-Filters](ProjectSettings.md#pre-filters).

## Contents

- [Special Rules](#special-rules)
- [Core Components](#core-components)
- [Input System (if installed)](#input-system-if-installed)
- [TextMeshPro (if installed)](#textmeshpro-if-installed)
- [UGUI (if installed)](#ugui-if-installed)

---

## Special Rules

These are a bit outliers in the general rule system and have to be configured individually via [Project Settings - Rules](ProjectSettings.md#rules)

- `UnityEvent` (and its variants) persistent listeners are checked for
  - target objects may not be unassigned
  - target method needs to exist
- `GameObject` is checked for **Missing Components**
- `Component` and `ScriptableObject` are checked for missing **Managed-Reference Types** (`[SerializeReference]`)
- `ScriptableObject` assets are checked for unloadable entries caused by **Missing Scripts**
- [experimental - off by default] `GameObject` is checked for **Broken Prefab References**

## Core Components

Assembly: `derHugo.SceneValidation.Rules.Core`

| Type            | Rules                                                                       |
|-----------------|-----------------------------------------------------------------------------|
| `AudioListener` | at most one per scene (error when several are enabled, otherwise a warning) |
| `MeshCollider`  | `m_Mesh` required                                                           |
| `MeshFilter`    | `m_Mesh` required; `MeshRenderer` component required                        |
| `MeshRenderer`  | `MeshFilter` component required                                             |
| `RectTransform` | `rect` dimensions may not be negative                                       |
| `Renderer`      | `sharedMaterials` may not be empty; `sharedMaterials` items may not be null |
| `Rigidbody`     | requires at least one `Collider`                                            |
| `Rigidbody2D`   | requires at least one `Collider2D`                                          |

## Input System (if installed)

Assembly: `derHugo.SceneValidation.Rules.InputSystem`

| Type                       | Rules                    |
|----------------------------|--------------------------|
| `InputSystemUIInputModule` | `m_ActionsAsset` required |

## TextMeshPro (if installed)

Assembly: `derHugo.SceneValidation.Rules.TextMeshPro`

| Type       | Rules                                               |
|------------|-----------------------------------------------------|
| `TMP_Text` | `m_fontAsset` required; `m_sharedMaterial` required |

## UGUI (if installed)

Assembly: `derHugo.SceneValidation.Rules.UGUI`

| Type         | Rules                                                                   |
|--------------|-------------------------------------------------------------------------|
| `Selectable` | `targetGraphic` required if `transition` is `ColorTint` or `SpriteSwap` |
