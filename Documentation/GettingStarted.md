Package entry point: [README](../README.md)

---

<h1><img src="images/Icon.png" alt="Logo" width="80" align="middle" />&nbsp; Scene Validation - Getting Started</h1>

Use this page as the practical onboarding guide.

## Contents

- [Define Requirements](#define-requirements)
- [Configuration](#configuration)
- [Resolve Issues](#resolve-issues)
- [UI Integration and Results](#ui-integration-and-results)

---

## Define Requirements

In addition to some [Built-in Rules](BuiltInRules.md), Scene Validation supports three requirement-definition paths:

- [`Required`](ScriptingAPI.md#required-attribute)
- [`RequiredFieldRule<TTarget>`](ScriptingAPI.md#requiredfieldrule)
- [`ValidationRule<TTarget>`](ScriptingAPI.md#validationrule)

### Required Attribute

Use this for serialized object-reference fields you own.

```csharp
using derHugo.SceneValidation;
using UnityEngine;

public sealed class ExampleComponent : MonoBehaviour
{
    [Required] [SerializeField] private Camera _mainCamera;
}
```

Where this appears in configuration:

- [Project Settings - Rules - Required Fields](ProjectSettings.md#required-fields)

API reference:

- [Scripting API - Required Attribute](ScriptingAPI.md#required-attribute)

#### Required Attribute Options

All options are documented in:

- [Scripting API - Required Parameter Reference](ScriptingAPI.md#required-parameter-reference)

Quick option overview:

- `ValidationScope`
  - `Scene` (default): enforce during scene validation workflows.
  - `Prefab`: enforce for prefab workflows and prefab-context checks.
- `RecoveryOption`
  - `None` (default), `GetComponent`, `GetComponentInChildren`, `GetComponentInParent`, `FindAnyInScene`, `ALL`
- `ValidationIssueSeverity`
  - `Error` (default), `Warning`

### RequiredFieldRule

Use this when you cannot annotate fields directly, for example on third-party components.

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

Where this appears in configuration:

- [Project Settings - Rules - Required Fields](ProjectSettings.md#required-fields)

API reference:

- [Scripting API - RequiredFieldRule](ScriptingAPI.md#requiredfieldrule)

#### RequireField Callback

The callback and parameters are documented here:

- [Scripting API - RequireField Callback Signature](ScriptingAPI.md#requirefield-callback-signature)

`scope`, `recovery`, and `severity` use the same semantics as the [`Required`](ScriptingAPI.md#required-attribute) attribute parameters.

### ValidationRule

Use this for semantic validation that is not a simple missing-reference requirement.

```csharp
using derHugo.SceneValidation.Editor;
using UnityEngine;

namespace MyCompany.Validation
{
    internal sealed class CameraDepthRule : ValidationRule<Camera>
    {
        protected override string SettingsDisplayName => "Camera: Depth Range";
        protected override string SettingsDescription => "Checks camera depth range.";

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

Where this appears in configuration:

- [Project Settings - Rules - Validation Rules](ProjectSettings.md#validation-rules)

API reference:

- [Scripting API - ValidationRule](ScriptingAPI.md#validationrule)

#### ValidationRule Authoring Options

Detailed parameter and callback behavior:

- [Scripting API - ValidationRule Authoring Surface](ScriptingAPI.md#validationrule-authoring-surface)
- [Scripting API - AddIssue Callback Parameters](ScriptingAPI.md#addissue-callback-parameters)

## Configuration

Primary configuration pages:

- [Project Settings - Settings](ProjectSettings.md#settings): behavior, UI integrations, Play Mode, pre-build checks.
- [Project Settings - Rules](ProjectSettings.md#rules): enable/disable required and validation rules.
- [Project Settings - Pre-Filters](ProjectSettings.md#pre-filters): assembly and asset-location exclusions.
- [Project Settings - Issues](ProjectSettings.md#issues): embedded results view.

Recommended first pass:

1. Configure strictness in [Settings](ProjectSettings.md#settings).
2. Review enabled checks in [Rules](ProjectSettings.md#rules).
3. Add only necessary exclusions in [Pre-Filters](ProjectSettings.md#pre-filters).
4. Resolve or intentionally mute remaining issues in [Results Views](ResultsViews.md).

## Resolve Issues

[![][9]][9]

In results views you can:

- select/ping related objects or assets from issue rows
- ignore/unignore issue occurrences
- run `Try Fix` where recovery options are configured

See [Results Views - Issue Interaction Workflow](ResultsViews.md#issue-interaction-workflow) for details.

## UI Integration and Results

Scene Validation can integrate directly into core editor workflows.

[![][1]][1]

Detailed manuals:

- [UI Integrations - Toolbar Badge](UIIntegrations.md#toolbar-badge-unity-63)
- [UI Integrations - Hierarchy Badges](UIIntegrations.md#hierarchy-badges)
- [UI Integrations - Custom Object Field Drawer](UIIntegrations.md#custom-object-field-drawer)
- [UI Integrations - Results Window](UIIntegrations.md#results-window)
- [Results Views](ResultsViews.md)

[1]: images/Integrations.png
[9]: images/ResultsWindow_FullUI.png
