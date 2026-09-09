Package entry point: [README](../README.md)

---

<h1><img src="images/Icon.png" alt="Logo" width="80" align="middle" />&nbsp; Scene Validation - Sample</h1>

The `Scene Validation Example` sample is a guided tour of what Scene Validation detects and how to extend it. It bundles scenes, prefabs, ScriptableObjects, and custom-rule examples that intentionally trigger the [built-in checks](BuiltInRules.md) and the [`Required`](ScriptingAPI.md#required-attribute) / [`RequiredFieldRule<TTarget>`](ScriptingAPI.md#requiredfieldrule) / [`ValidationRule<TTarget>`](ScriptingAPI.md#validationrule) extension points.

## Contents

- [Additional Dependencies](#additional-dependencies)
- [Import the Sample](#import-the-sample)
- [View Reported Issues](#view-reported-issues)
- [Scenes](#scenes)
- [Prefabs](#prefabs)
- [ScriptableObjects](#scriptableobjects)
- [Scripts](#scripts)
- [Editor / Custom Rules](#editor--custom-rules)

---

## Additional Dependencies

The sample content exercises the optional, integration-specific built-in rules. To see every demonstrated issue, make sure the matching modules and packages are installed:

- Physics (missing colliders on rigidbodies)
- UGUI (`Selectable` target graphic)
- TextMeshPro (a `TMP_Text` field in the coverage prefab).
  <br>*TextMeshPro ships inside `com.unity.ugui` 2.0.0+ (Unity 6+); with older uGUI (below 2.0.0, e.g. Unity 2022 LTS) the sample installs the separate `com.unity.textmeshpro` package automatically on import.*
- Input System (action asset)

Checks for packages that are not installed are simply skipped. See [Built-in Rules](BuiltInRules.md) for the full list of what each integration validates.

## Import the Sample

Find

    Assets/derHugo/SceneValidation/Samples/SceneValidationExample.unitypackage

and double click it in order to import the sample

The imported content lands under:

    Assets/derHugo/SceneValidation/Samples/

## View Reported Issues

Open one of the [scenes](#scenes) below, then open the results window to inspect the reported issues:

- `Window > derHugo > SceneValidation > Issues`, or
- click the main toolbar badge.

[![][1]][1]

From here you can select or ping the related objects and assets, ignore/unignore occurrences, and run `Try Fix` where recovery options are configured. See [Results Views](ResultsViews.md) for the full interaction workflow.

## Scenes

- `Scenes/SceneValidation_BuiltInRules.unity` - showcases the built-in core/component checks: missing mesh/material references, missing colliders on rigidbodies, invalid (negative) RectTransform dimensions, missing `Selectable` target graphic, missing TextMeshPro font/material, multiple `AudioListener`s, a missing Input System action asset, and a broken prefab reference.
- `Scenes/SceneValidation_CustomRules.unity` - showcases the user-defined extensions: [`Required`](ScriptingAPI.md#required-attribute), [`RequiredFieldRule<TTarget>`](ScriptingAPI.md#requiredfieldrule), and [`ValidationRule<TTarget>`](ScriptingAPI.md#validationrule).

## Prefabs

- `Prefabs/BuiltInRulesCoverage.prefab` - grouped objects that intentionally trigger the built-in rules.
- `Prefabs/CustomRulesCoverage.prefab` - grouped objects that intentionally trigger the custom rules plus the special `UnityEvent` and managed-reference checks.
- `Prefabs/GameObject_With_Missing_Prefab.prefab` - contains a nested prefab reference whose source asset is intentionally missing, to demonstrate the (experimental, off-by-default) broken-prefab-reference check. This is a pure-YAML broken reference and needs no scripts.

## ScriptableObjects

- `ScriptableObjects/ShowcaseConfig_Valid.asset` - a valid baseline configuration.
- `ScriptableObjects/ShowcaseConfig_Invalid.asset` - an invalid configuration that also contains a missing managed-reference type.
- `ScriptableObjects/ShowcaseConfig_MissingScript.asset` - a ScriptableObject asset whose script can no longer be resolved.

## Scripts

`Scripts/` contains the small showcase types referenced by the scenes, prefabs, and assets above (`SceneValidationShowcaseComponent`, `SceneValidationShowcaseConfig`, `SceneValidationShowcaseUnityEventTarget`, `SceneValidationCustomRuleTarget`, and the managed reference types). They double as a reference for annotating your own serialized fields with [`Required`](ScriptingAPI.md#required-attribute).

## Editor / Custom Rules

`Editor/CustomRuleExamples.cs` shows how to author rules for types you do not own:

- `ExampleAudioSourceRequiredRule` - a [`RequiredFieldRule<AudioSource>`](ScriptingAPI.md#requiredfieldrule) that requires the output mixer group.
- `ExampleCustomRangeRule` - a [`ValidationRule<SceneValidationCustomRuleTarget>`](ScriptingAPI.md#validationrule) that performs a semantic min/max check.

These rules appear under `Project Settings > derHugo > SceneValidation > Settings > Rules` and drive the issues shown in `SceneValidation_CustomRules.unity`.

Related pages:

- [Getting Started](GettingStarted.md)
- [Built-in Rules](BuiltInRules.md)
- [Project Settings](ProjectSettings.md)
- [Scripting API](ScriptingAPI.md)

[1]: images/ResultsWindow_FullUI.png
