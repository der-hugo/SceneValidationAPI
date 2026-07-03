<h1><img src="Documentation~/images/Icon.png" alt="Logo" width="120" align="middle" />&nbsp; Scene Validation</h1>

`SceneValidation` is an editor-first quality gate that catches missing references and setup problems before they break Play Mode or builds.

## Contents

- [Requirements](#requirements)
- [Highlights](#highlights)
- [Where can I get it?](#where-can-i-get-it)
- [What You Can Validate](#what-you-can-validate)
- [Define Validation Requirements](#define-validation-requirements)
  - [Required Attribute](#required-attribute)
  - [RequiredFieldRule](#requiredfieldrule)
  - [ValidationRule](#validationrule)
- [UI and Results](#ui-and-results)
- [Documentation](#documentation)

## Requirements

- Unity 6000.3 LTS and newer

## Highlights

- Fast feedback directly in the editor through toolbar and hierarchy badges.
- Works across loaded scenes, Build Settings scenes, prefab assets, and ScriptableObject assets.
- Supports `[Required]`, `RequiredFieldRule<TTarget>`, and custom `ValidationRule<TTarget>` extensions.
- Includes optional `Try Fix` workflows (manual or automatic, depending on configuration).
- Adds Play Mode and pre-build guardrails to prevent shipping known setup issues.

## Where can I get it?

[Unity Asset Store](https://assetstore.unity.com/packages/slug/376598)

## What You Can Validate

- Loaded scenes
- Build Settings scenes
- Prefab assets
- ScriptableObject assets (including missing ScriptableObject scripts)
- Missing managed-reference types on components and ScriptableObject assets
- Addressable scenes (if installed)

See [Project Settings - Settings](Documentation~/ProjectSettings.md#settings) for scope toggles and build/play integration behavior.

## Define Validation Requirements

### Required Attribute

Use [`Required`](Documentation~/ScriptingAPI.md#required-attribute) when you own the serialized field.

- API details: [Scripting API - Required Attribute](Documentation~/ScriptingAPI.md#required-attribute)
- option details: [Scripting API - Required Parameter Reference](Documentation~/ScriptingAPI.md#required-parameter-reference)
- workflow guide: [Getting Started - Required Attribute](Documentation~/GettingStarted.md#required-attribute)

### RequiredFieldRule

Use [`RequiredFieldRule<TTarget>`](Documentation~/ScriptingAPI.md#requiredfieldrule) for third-party or external types that cannot be annotated directly.

- API details: [Scripting API - RequiredFieldRule](Documentation~/ScriptingAPI.md#requiredfieldrule)
- callback details: [Scripting API - RequireField Callback Signature](Documentation~/ScriptingAPI.md#requirefield-callback-signature)
- workflow guide: [Getting Started - RequiredFieldRule](Documentation~/GettingStarted.md#requiredfieldrule)

### ValidationRule

Use [`ValidationRule<TTarget>`](Documentation~/ScriptingAPI.md#validationrule) for semantic checks that are not just missing references.

- API details: [Scripting API - ValidationRule](Documentation~/ScriptingAPI.md#validationrule)
- callback details: [Scripting API - ValidationRule Authoring Surface](Documentation~/ScriptingAPI.md#validationrule-authoring-surface)
- workflow guide: [Getting Started - ValidationRule](Documentation~/GettingStarted.md#validationrule)

## UI and Results

[![][1]][1]
[![][2]][2]

### Main Toolbar Badge

<img src="Documentation~/images/ToolbarBadge.png" width="400"/>

### Hierarchy Badges

<img src="Documentation~/images/HierarchyBadges.png" width="400">

### Custom Object Field Drawer

<img src="Documentation~/images/CustomDrawer.png" width="400">

### Prevent Entering Play Mode

<img src="Documentation~/images/PlayPrevented.png" width="200">

<img src="Documentation~/images/PlayButton.png" width="400">

Cancels the Play Mode transition and disables the Play/Pause toolbar buttons while blocking validation errors exist.

### Build Pre-Processor Handling

<img src="Documentation~/images/BuildPrevent_Issues.png" width="200">
<img src="Documentation~/images/BuildPrevented_Ignored.png" width="200">

### Results Window

[![][2]][2]

Results behavior details:

- [UI Integrations](Documentation~/UIIntegrations.md)
- [Results Views](Documentation~/ResultsViews.md)
- [Project Settings - Issues](Documentation~/ProjectSettings.md#issues)

## Documentation

- [Getting Started](Documentation~/GettingStarted.md): quick-start onboarding and first setup pass.
- [UI Integrations](Documentation~/UIIntegrations.md): Visuals API for toolbar, hierarchy, inspector, play/build guardrails, and integration states.
- [Project Settings](Documentation~/ProjectSettings.md): all toggles, lists, and scope behavior.
- [Results Views](Documentation~/ResultsViews.md): validation window and project settings issue view behavior.
- [Built-in Rules](Documentation~/BuiltInRules.md): predefined requirement rules that ship with this package
- [Sample](Documentation~/Sample.md): walkthrough of the included example content.
- [Scripting API](Documentation~/ScriptingAPI.md): Scripting API for `Required`, `RequiredFieldRule<TTarget>`, and `ValidationRule<TTarget>`.

[1]: Documentation~/images/Integrations.png
[2]: Documentation~/images/ResultsWindow_FullUI.png
