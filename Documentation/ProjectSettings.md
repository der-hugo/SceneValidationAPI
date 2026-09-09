Package entry point: [README](../README.md)

---

<h1><img src="images/Icon.png" alt="Logo" width="80" align="middle" />&nbsp; Scene Validation - Project Settings</h1>

## Contents
- [Settings](#settings)
- [Pre-Filters](#pre-filters)
- [Rules](#rules)
- [Issues](#issues)

---

## Settings

`Project Settings > derHugo > SceneValidation > Settings`

[![][1]][1]

### General Settings

#### Automatic Scanning

Keeps validation results up to date automatically while editing.

- Enable for continuous, always-current feedback (recommended).
- Disable to scan only when you trigger a refresh manually (the `Refresh` button in a results view, or the toolbar badge's `Refresh` context action).

#### Scan Scope

Controls how far validation reaches. The three modes are strictly additive - each one validates everything the previous mode does, plus more. In every mode, prefabs and ScriptableObjects are included by following references (the assets a build would actually pull in), not by scanning the whole project - except for the widest mode.

- **Loaded Scenes & Prefab Stage Only** - the currently loaded scenes and the open prefab edit stage, plus the prefabs and ScriptableObjects they reference (recursively). Note: because this ignores everything else in a build, blocking issues elsewhere can go unnoticed - a warning is shown while it is selected.
- **Build-Referenced Assets** *(default, recommended)* - the above, plus Build Settings scenes (and, when the Addressables package is installed, Addressable scenes), the Player Settings preloaded assets, all Addressable assets, and everything in `Resources`/`StreamingAssets` folders - together with every prefab and ScriptableObject those roots reference (recursively). This matches what ends up in a build, so assets nothing references can no longer produce blocking issues. Addressable assets are always included (not only when referenced) because they are frequently loaded by address/path string rather than by a direct reference.
- **All Assets** - every prefab and ScriptableObject in the project, whether anything references it or not. Choose this if you want to validate the entire project (the behavior prior to this setting); it is slower on large projects and may report issues in assets no build includes.

The scope is recomputed automatically when the loaded scenes, prefab stage, Build Settings scene list, or Addressables configuration change. Use a manual `Refresh` for an exact, up-to-the-moment result.

- See [What You Can Validate](../README.md#what-you-can-validate) for the full list of validated scopes.


### UI Integrations

#### Show Toolbar Badge

Shows aggregate validation status in Unity's main toolbar.

- Useful for always-visible project health feedback.
- Detailed behavior: [UI Integrations - Toolbar Badge](UIIntegrations.md#toolbar-badge-unity-63).

#### Show Hierarchy Badges

Shows issue badges for loaded scenes and affected GameObjects in Hierarchy.

- Useful for direct navigation from Hierarchy into problematic objects.
- Detailed behavior: [UI Integrations - Hierarchy Badges](UIIntegrations.md#hierarchy-badges).

#### Apply Custom Drawer

Uses the custom inspector drawer for fields marked with [`Required`](ScriptingAPI.md#required-attribute).

- Highlights unresolved object references while editing components.
- Can expose manual `Try Fix` when recovery is configured.
- Detailed behavior: [UI Integrations - Custom Object Field Drawer](UIIntegrations.md#custom-object-field-drawer).

### Play Mode

#### Prevent Entering Play Mode

Cancels Play Mode entry when validation errors are present, and disables the Play/Pause toolbar buttons (with an explanatory tooltip) while blocked.

- Use in workflows where scene or asset validity must stay clean before runtime testing.
- Detailed behavior: [UI Integrations - Prevent Entering Play Mode](UIIntegrations.md#prevent-entering-play-mode).

#### Warn For Ignored Issues Before Play Mode

Shows a confirmation dialog before entering Play Mode if ignored issues exist.

- Useful when your team permits temporary muting but wants explicit acknowledgement.
- Related behavior: [UI Integrations - Prevent Entering Play Mode](UIIntegrations.md#prevent-entering-play-mode).

### Pre-Build

#### Check Before Building

Runs validation before build starts.

- Recommended for predictable CI and release builds.
- Detailed behavior: [UI Integrations - Build Pre-Processor](UIIntegrations.md#build-pre-processor).

#### Warn For Ignored Issues Before Build

Shows a confirmation dialog before build continues if ignored blocking issues exist.

- Keeps muted issues visible as an explicit decision point.
- In batch mode, this confirmation cannot be shown, so build is aborted when ignored blocking issues are present.
- Related behavior: [UI Integrations - Build Pre-Processor](UIIntegrations.md#build-pre-processor).

#### Build Issue Handling

Available when **Check Before Building** is enabled.

- `Ask`: prompt whether to continue when non-ignored blocking errors are found.
- `Fail`: abort the build immediately when non-ignored blocking errors are found.

### Advanced Settings

- `Go to Filters`: opens [Pre-Filters](#pre-filters).
- `Go to Rules`: opens [Rules](#rules).

## Pre-Filters

`Project Settings > derHugo > SceneValidation > Settings > Pre-Filters`

Pre-filters run before individual rule toggles and remove full targets or rule sources from evaluation.

[![][3]][3]

### Assembly Pre-Filters

#### Ignored Target Type Assemblies

Skips validation for targets whose concrete type comes from listed assemblies.

#### Ignored Validation/Required Rule Assemblies

Skips discovery or execution of [`ValidationRule<TTarget>`](ScriptingAPI.md#validationrule) and [`RequiredFieldRule<TTarget>`](ScriptingAPI.md#requiredfieldrule) defined in listed assemblies.

### Asset Pre-Filters

#### Ignored Prefab/ScriptableObject Packages

Skips prefab and ScriptableObject asset validation for assets in listed packages.

#### Ignored Prefab/ScriptableObject Asset Folders

Skips prefab and ScriptableObject asset validation for assets under listed `Assets/...` folders.

Filter usage guidance:

- Start with [Rules](#rules) toggles first.
- Use pre-filters when a complete dependency assembly/package/folder should stay out of validation.
- Keep filter lists minimal and review regularly.

## Rules

`Project Settings > derHugo > SceneValidation > Settings > Rules`

Use this page to enable or disable optional rule groups and individual rules.

[![][2]][2]

### Required Fields

Contains required-rule groups discovered from:

- [`Required`](ScriptingAPI.md#required-attribute) attributes
- [`RequiredFieldRule<TTarget>`](ScriptingAPI.md#requiredfieldrule) registrations

How to use:

- Disable a group to mute all related required checks for a target type.
- Keep group enabled and disable child entries for fine-grained control.

Related guides:

- [Getting Started - Define Requirements](GettingStarted.md#define-requirements)
- [Scripting API - Required](ScriptingAPI.md#required-attribute)
- [Scripting API - RequiredFieldRule](ScriptingAPI.md#requiredfieldrule)

### Validation Rules

Contains optional semantic rules, including:

- [Built-in Rules](BuiltInRules.md) and special checks outside of the normal requirements-definition pipeline
  - UnityEvent persistent listeners
  - missing components on GameObjects
  - missing managed-reference types on Components and ScriptableObject assets
  - missing scripts in ScriptableObject assets
  - [experimental] broken prefab references on GameObjects
- discovered custom [`ValidationRule<TTarget>`](ScriptingAPI.md#validationrule) rules

How to use:

- Opt-out of rules you don't want to apply to your project at all
- Disable noisy rules temporarily while migrating older content
- Re-enable once scenes/assets are cleaned up

Related guides:

- [Scripting API - ValidationRule](ScriptingAPI.md#validationrule)
- [Getting Started - ValidationRule](GettingStarted.md#validationrule)

## Issues

`Project Settings > derHugo > SceneValidation > Issues`

This page embeds the same grouped issue renderer used by the dedicated validation window.

- Behavior details: [Results Views - Project Settings Issues View](ResultsViews.md#project-settings-issues-view)
- Interaction details: [Results Views - Issue Interaction Workflow](ResultsViews.md#issue-interaction-workflow)

[1]: images/Settings_General_FullUI.png
[2]: images/Settings_Rules_FullUI.png
[3]: images/Settings_Filter_FullUI.png
