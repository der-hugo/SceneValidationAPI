Package entry point: [README](../README.md)

---

<h1><img src="images/Icon.png" alt="Logo" width="80" align="middle" />&nbsp; Scene Validation - Scripting API</h1>

This page documents the public extension API of `SceneValidation` and is aligned with the XML documentation in code.

## Contents

- [Overview](#overview)
- [Code-Driven Validation](#code-driven-validation)
  - [Run Validation and Read Summary](#run-validation-and-read-summary)
  - [Walk Result Graph](#walk-result-graph)
  - [Mute or Fix Issues from Code](#mute-or-fix-issues-from-code)
  - [Automation Notes](#automation-notes)
- [Required Attribute](#required-attribute)
  - [Required Parameter Reference](#required-parameter-reference)
  - [Recovery Option Order](#recovery-option-order)
  - [Required Related Pages](#required-related-pages)
- [RequiredFieldRule](#requiredfieldrule)
  - [RequireField Callback Signature](#requirefield-callback-signature)
  - [RequiredFieldRule Behavior Notes](#requiredfieldrule-behavior-notes)
  - [RequiredFieldRule Related Pages](#requiredfieldrule-related-pages)
- [ValidationRule](#validationrule)
  - [ValidationRule Authoring Surface](#validationrule-authoring-surface)
  - [AddIssue Callback Parameters](#addissue-callback-parameters)
  - [ValidationRule Behavior Notes](#validationrule-behavior-notes)
  - [ValidationRule Related Pages](#validationrule-related-pages)
- [Choose the Right Extension Type](#choose-the-right-extension-type)
- [Related Documentation](#related-documentation)

---

## Overview

You typically extend validation in two ways:

- [`Required`](#required-attribute) and [`RequiredFieldRule<TTarget>`](#requiredfieldrule): required reference checks.
- [`ValidationRule<TTarget>`](#validationrule): semantic or relationship checks.

Use [`Required`](#required-attribute) when you own the field.  
Use [`RequiredFieldRule<TTarget>`](#requiredfieldrule) when you do not control the source type.

You can also run Scene Validation from editor code, automation tools, CI, editor tests, or agent workflows through [`ValidationEngine`](#code-driven-validation).

## Code-Driven Validation

Namespace: `derHugo.SceneValidation.Editor`

The public automation entry point is `ValidationEngine`. It runs the same validation engine used by the window, toolbar, build preprocessor, hierarchy badges, and project settings views.

This API is editor-only. Put calling code in an Editor assembly, an `Editor/` folder, or an asmdef that references `derHugo.SceneValidation.Editor`.

### Run Validation and Read Summary

Use `ValidationEngine.Validate` when an automation workflow needs deterministic, freshly rebuilt results.

```csharp
using System;
using derHugo.SceneValidation.Editor;

public static class SceneValidationAutomation
{
    public static void ValidateProject()
    {
        var report = ValidationEngine.Validate(ValidationEngine.AllScopes);
        var summary = report.Summary;

        if (summary.ErrorCount > 0)
        {
            throw new Exception(
                $"Scene Validation failed with {summary.ErrorCount} error(s) and {summary.WarningCount} warning(s).");
        }
    }
}
```

Typical CI usage is to call this method from Unity batch mode with `-executeMethod`.

Core entry points:

| API | Use |
| --- | --- |
| `ValidationEngine.Validate()` | Forces validation for `ValidationEngine.AllScopes` and returns a `ValidationReport`. |
| `ValidationEngine.Validate(ValidationResultScope scope, bool force = true)` | Forces validation for a selected scope. |
| `ValidationEngine.GetReport()` | Returns a report for all scopes using current caches, populating missing caches on demand. |
| `ValidationEngine.GetReport(ValidationResultScope scope)` | Returns a report for a selected scope using current caches, populating missing caches on demand. |
| `ValidationEngine.Refresh(ValidationResultScope scope, bool force = false)` | Lower-level cache refresh without directly returning a report. |
| `ValidationEngine.EnsureHasRun(ValidationResultScope scope, bool bypassAutomaticScanningSetting = false)` | Ensures caches exist before direct result queries. |
| `ValidationEngine.HasRunForScope(ValidationResultScope scope)` | Checks whether the requested scope currently has cached results. |

Scope helpers:

| API | Meaning |
| --- | --- |
| `ValidationEngine.AllScopes` | All supported scene and asset scopes for the current package configuration. |
| `ValidationResultScope.LoadedScenes` | Currently loaded scene objects and prefab stage contents. |
| `ValidationResultScope.BuildSettingsScenes` | Scenes included in Build Settings. |
| `ValidationResultScope.PrefabAssets` | Prefab asset validation results. |
| `ValidationResultScope.ScriptableObjectAssets` | ScriptableObject asset validation results. |

`ValidationEngine.Validate` and `ValidationEngine.GetReport` return `report.Scope` as the effective scope after Scene Validation settings are applied.

### Walk Result Graph

`ValidationReport` exposes the same grouped model used by the editor UI.

| Type | Important Members |
| --- | --- |
| `ValidationReport` | `Scope`, `Summary`, `SceneResults`, `PrefabResults`, `ScriptableObjectResults` |
| `ValidationEngine.ValidationIssueSummary` | `ErrorCount`, `WarningCount`, `VisibleIssueCount`, `IgnoredIssueCount`, `IgnoredErrorCount`, `IgnoredWarningCount` |
| `SceneValidationResult` | `Scene`, `SceneName`, `SceneAssetPath`, `GameObjects`, `IssueCount` |
| `PrefabValidationResult` | `PrefabRoot`, `PrefabName`, `PrefabAssetPath`, `GameObjects`, `IssueCount` |
| `GameObjectValidationResult` | `GameObject`, `GameObjectName`, `Components`, `MissingScriptsCount`, `MissingManagedReferenceTypesCount`, `HasBrokenPrefabReference`, issue keys |
| `ComponentValidationResult` | `Component`, `GameObject`, `ComponentType`, `ComponentName`, `Issues` |
| `ScriptableObjectValidationResult` | `Target`, `TargetType`, `TargetName`, `AssetPath`, `Issues`, `MissingScriptsCount`, `MissingManagedReferenceTypesCount` |
| `ValidationIssue` | `TargetObject`, `Component`, `PropertyPath`, `DisplayName`, `Message`, `Severity`, `PersistentKey`, source-location data |

Component and ScriptableObject issues are represented as `ValidationIssue` rows. Integrated special cases such as missing component scripts, missing managed-reference types, and broken prefab references are represented on `GameObjectValidationResult` or `ScriptableObjectValidationResult`.

```csharp
using derHugo.SceneValidation.Editor;
using UnityEngine;

public static class SceneValidationIssueLogger
{
    public static void LogLoadedSceneIssues()
    {
        var report = ValidationEngine.Validate(ValidationResultScope.LoadedScenes);

        foreach (var sceneResult in report.SceneResults)
        {
            foreach (var gameObjectResult in sceneResult.GameObjects)
            {
                foreach (var componentResult in gameObjectResult.Components)
                {
                    foreach (var issue in componentResult.Issues)
                    {
                        if (ValidationEngine.IsIssueIgnored(issue))
                        {
                            continue;
                        }

                        Debug.Log(
                            $"{sceneResult.SceneName}: {issue.HierarchyPath} -> " +
                            $"{issue.ComponentName}.{issue.DisplayName} {issue.Message}",
                            issue.TargetObject);
                    }
                }

                if (gameObjectResult.MissingScriptsCount > 0)
                {
                    Debug.LogWarning(
                        $"{sceneResult.SceneName}: {gameObjectResult.GameObjectName} has " +
                        $"{gameObjectResult.MissingScriptsCount} missing component script(s).",
                        gameObjectResult.GameObject);
                }
            }
        }
    }
}
```

For per-result counters, use the public summary helpers:

```csharp
var sceneSummary = ValidationEngine.GetIssueSummary(report.SceneResults[0]);
var visibleIssues = ValidationEngine.GetVisibleIssueCount(report.SceneResults[0]);
```

### Mute or Fix Issues from Code

The public API also exposes the same ignore and fix operations used by the result views.

Ignore helpers:

| API | Use |
| --- | --- |
| `ValidationEngine.IsIssueIgnored(ValidationIssue issue)` | Checks whether a concrete issue is muted. |
| `ValidationEngine.IsIssueIgnored(string issueKey)` | Checks whether a persistent issue key is muted. |
| `ValidationEngine.SetIssueIgnored(ValidationIssue issue, bool ignored)` | Mutes or unmutes one concrete issue. |
| `ValidationEngine.SetIssuesIgnored(IEnumerable<string> issueKeys, bool ignored)` | Mutes or unmutes multiple persistent issue keys. |

Fix helpers:

| API | Use |
| --- | --- |
| `ValidationEngine.HasTryFixConfiguredForIssue(ValidationIssue issue)` | Checks whether a required-reference issue has configured recovery options. |
| `ValidationEngine.CanTryFixIssue(ValidationIssue issue)` | Checks whether recovery can run right now. |
| `ValidationEngine.TryFixIssue(ValidationIssue issue)` | Attempts configured recovery for a required-reference issue. |
| `ValidationEngine.CanRemoveMissingComponents(GameObject gameObject)` | Checks whether missing component scripts can be removed. |
| `ValidationEngine.RemoveMissingComponents(GameObject gameObject)` | Removes missing component scripts from a GameObject. |
| `ValidationEngine.CanFixMissingManagedReferenceTypes(GameObject gameObject)` | Checks scene/prefab object managed-reference cleanup support. |
| `ValidationEngine.FixMissingManagedReferenceTypes(GameObject gameObject)` | Clears missing managed-reference type data from components on a GameObject. |
| `ValidationEngine.CanFixMissingManagedReferenceTypes(string assetPath)` | Checks ScriptableObject asset managed-reference cleanup support. |
| `ValidationEngine.FixMissingManagedReferenceTypes(string assetPath)` | Clears missing managed-reference type data from ScriptableObjects stored at an asset path. |

Example agent-style fix loop for currently loaded scenes:

```csharp
using derHugo.SceneValidation.Editor;

public static class SceneValidationAutoFix
{
    public static void FixRecoverableLoadedSceneIssues()
    {
        var report = ValidationEngine.Validate(ValidationResultScope.LoadedScenes);

        foreach (var sceneResult in report.SceneResults)
        {
            foreach (var gameObjectResult in sceneResult.GameObjects)
            {
                foreach (var componentResult in gameObjectResult.Components)
                {
                    foreach (var issue in componentResult.Issues)
                    {
                        if (!ValidationEngine.IsIssueIgnored(issue)
                            && ValidationEngine.CanTryFixIssue(issue))
                        {
                            ValidationEngine.TryFixIssue(issue);
                        }
                    }
                }

                if (gameObjectResult.MissingScriptsCount > 0
                    && ValidationEngine.CanRemoveMissingComponents(gameObjectResult.GameObject))
                {
                    ValidationEngine.RemoveMissingComponents(gameObjectResult.GameObject);
                }

                if (gameObjectResult.MissingManagedReferenceTypesCount > 0
                    && ValidationEngine.CanFixMissingManagedReferenceTypes(gameObjectResult.GameObject))
                {
                    ValidationEngine.FixMissingManagedReferenceTypes(gameObjectResult.GameObject);
                }
            }
        }

        ValidationEngine.Refresh(ValidationResultScope.LoadedScenes, true);
    }
}
```

### Automation Notes

- `Validate` uses the same project settings, enabled/disabled rule settings, ignored issue settings, and asset filters as the editor UI.
- `Validate` is the recommended entry point for CI and agents because it forces the requested caches to rebuild by default.
- `GetReport` is useful for tools that want the current editor state without an unconditional rebuild.
- The report lists are snapshots, but result objects still reference Unity objects. Re-run validation after scene or asset mutations.
- Build Settings scene scans can open scenes additively when required. Use focused scopes such as `LoadedScenes` or `PrefabAssets` when a tool only needs one domain.
- This API is editor-only and is not available in player/runtime builds.

## Required Attribute

Namespace: `derHugo.SceneValidation`

Use `[Required]` on serialized object-reference fields.

```csharp
using derHugo.SceneValidation;
using UnityEngine;

public sealed class WeaponView : MonoBehaviour
{
    [Required] [SerializeField] private Transform _muzzle;
}
```

### Required Parameter Reference

| Parameter | Meaning | Validation Effect |
| --- | --- | --- |
| `ValidationScope scope` | Where the requirement is enforced. | `Scene` applies in scene validation workflows. `Prefab` applies in prefab workflows and prefab-context checks. |
| `RecoveryOption recovery` | Which fix strategies are allowed when the reference is missing. | Controls what `Try Fix` can attempt (manual and auto mode). Multiple flags can be combined. |
| `ValidationIssueSeverity severity` | How strict the issue is reported. | `Error` is blocking. `Warning` is advisory. |

### Recovery Option Order

When recovery is attempted, options are considered in this order:

1. `GetComponent`
2. `GetComponentInChildren` (includes inactive)
3. `GetComponentInParent` (includes inactive)
4. `FindAnyInScene` (includes inactive)
5. `ALL` (convenience flag for combining all options above)

### Required Related Pages

- [Getting Started - Required Attribute](GettingStarted.md#required-attribute)
- [Project Settings - Required Fields](ProjectSettings.md#required-fields)

## RequiredFieldRule

Namespace: `derHugo.SceneValidation.Editor`

Use this to register required fields for types you cannot annotate with `[Required]`.

```csharp
using derHugo.SceneValidation.Editor;
using UnityEngine;

namespace MyCompany.Validation
{
    internal sealed class AudioSourceRequiredFieldsRule : RequiredFieldRule<AudioSource>
    {
        protected override void CollectFields(RequireFieldCallback requireField)
        {
            requireField("m_OutputAudioMixerGroup");
        }
    }
}
```

### RequireField Callback Signature

```csharp
requireField(
    string fieldName,
    ValidationScope scope = ValidationScope.Scene,
    RecoveryOption recovery = RecoveryOption.None,
    ValidationIssueSeverity severity = ValidationIssueSeverity.Error)
```

Parameter semantics:

- `fieldName`: serialized property path relative to `TTarget` (Unity serialized name/path, not C# display name). The resolved member is expected to be an object-reference member.
- `scope`, `recovery`, `severity`: same semantics as [`Required`](#required-attribute).

Path notes:

- You can register nested paths such as `m_Settings.m_Target`.
- Registering a collection root such as `m_Items` applies to collection elements during validation.

### RequiredFieldRule Behavior Notes

- Rules are discovered automatically from non-abstract `RequiredFieldRule` types.
- Registered rules appear in `Project Settings > derHugo > SceneValidation > Settings > Rules > Required Fields`.
- Optional rule toggles and issue muting depend on stable path and type registration identifiers.
- Override `CollectFields` as `protected override`. The base member is declared `protected internal`, so from your own assembly only the `protected` part applies.

### RequiredFieldRule Related Pages

- [Getting Started - RequiredFieldRule](GettingStarted.md#requiredfieldrule)
- [Project Settings - Required Fields](ProjectSettings.md#required-fields)

## ValidationRule

Namespace: `derHugo.SceneValidation.Editor`

Use this for semantic checks that are not simple missing-reference requirements.

```csharp
using derHugo.SceneValidation.Editor;
using UnityEngine;

namespace MyCompany.Validation
{
    internal sealed class CameraDepthRule : ValidationRule<Camera>
    {
        protected override string SettingsDisplayName => "Camera: Depth Range";
        protected override string SettingsDescription => "Checks that Camera.depth is in an expected range.";

        protected override void Validate(in ValidationRuleContext context, AddIssueCallback addIssue)
        {
            var targetObject = context.TargetObject;
            if (targetObject.depth < -100f || targetObject.depth > 100f)
            {
                addIssue("m_Depth", "Depth", "must be between -100 and 100", ValidationIssueSeverity.Warning);
            }
        }
    }
}
```

### ValidationRule Authoring Surface

- `SettingsDisplayName`: label shown in the Rules page.
- `SettingsDescription`: description/tooltip shown in the Rules page.
- `Validate(in ValidationRuleContext context, AddIssueCallback addIssue)`: called for each matching target object.

`ValidationRuleContext` members:

- `context.TargetObject`: typed object currently being validated.
- `context.Scope`: current `ValidationRuleScope` — `Scene`, `Prefab`, or `ScriptableObject`.
- `context.SerializedObject`: serialized access to inspect properties by path.
- `context.IsIsolatedSceneScope`: `true` when the current scene-scope execution comes from an isolated scene scan (for example a Build Settings or Addressable scene pass); otherwise validation runs against the currently loaded scene set.

### AddIssue Callback Parameters

`addIssue(string propertyPath, string displayName, string message, ValidationIssueSeverity severity = ValidationIssueSeverity.Error)`

- `propertyPath`: serialized path to existing or virtual property, or empty for a rule-level issue.
- `displayName`: label shown before the message text.
- `message`: concise violation description shown in results.
- `severity`: `Error` or `Warning`.

### ValidationRule Behavior Notes

- Rules are discovered automatically from non-abstract `ValidationRule` types.
- Rules appear in `Project Settings > derHugo > SceneValidation > Settings > Rules > Validation Rules`.
- `ValidationRule<TTarget>` derives issue identity from the concrete rule type.
- If `displayName` is empty, Scene Validation derives a fallback from property path or rule metadata.
- Override `Validate` as `protected override`. Also override `SettingsDisplayName` / `SettingsDescription` as `protected override`: their base declarations are `protected internal`, so from your own assembly you use `protected`.

### ValidationRule Related Pages

- [Getting Started - ValidationRule](GettingStarted.md#validationrule)
- [Project Settings - Validation Rules](ProjectSettings.md#validation-rules)
- [Built-in Rules](BuiltInRules.md)

## Choose the Right Extension Type

- Use [`Required`](#required-attribute) for direct serialized field ownership.
- Use [`RequiredFieldRule<TTarget>`](#requiredfieldrule) for external or third-party target types.
- Use [`ValidationRule<TTarget>`](#validationrule) for semantic validation logic.

## Related Documentation

- [Getting Started](GettingStarted.md)
- [Project Settings](ProjectSettings.md)
- [Results Views](ResultsViews.md)
- [Built-in Rules](BuiltInRules.md)
