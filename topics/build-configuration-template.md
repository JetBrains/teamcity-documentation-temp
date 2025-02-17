[//]: # (title: Build Configuration Template)
[//]: # (auxiliary-id: Build Configuration Template)

_Build configuration templates_ allow you to eliminate duplication of build configuration settings. If you want to have several similar (not necessarily identical) build configurations and be able to modify their common settings in one place without having to edit each configuration, create a build configuration template with those settings. Modifying template settings affects __all__ build configurations associated with this template.

It is possible to define a default template in a project for all build configurations in this project and its subprojects. The [section below](#Defining+default+template+for+project) contains details.

Build configuration templates support project hierarchy: once created, available templates include the ones from the current project and its parents. On copying a project or a build configuration, the templates that do not belong to the target project or one of its parents are automatically copied. You can associate a build configuration with a template only if the template belongs to the current project or one of its parents.

## Creating build configuration template

There are several ways to create a build configuration template:
* __Manually__, like a [regular build configuration](creating-and-editing-build-configurations.md#Creating+Build+Configuration+Template).
* __Extract__ from an existing build configuration: there is the __Extract template__ option available from the __Actions__ menu in the upper right corner of the screen. Note that if you extract a template from a build configuration, the original configuration automatically becomes [associated](#Associating+build+configurations+with+templates) with the newly created template.

<anchor name="BuildConfigurationTemplate-Definingdefaulttemplateforproject"/>

## Defining default template for project

Default templates allow affecting all build configurations in this project and its subprojects.

You can associate all the build configurations of the project with a default template using the __General Settings__ page of the project administration, by selecting a template from the __Default template__ drop-down menu. The option is available if at least one template is defined in the project or its parent. All new build configurations will inherit the default template settings.

The settings of the existing configurations will be preserved.

When defined, the default template affects all build configurations and subprojects of this project unless other default templates are defined in subprojects. With default templates, you can easily modify all project's build configurations, for example:
* add a specific build feature to all build configurations of a project
* switch all build configurations to some specific checkout mode
* provide a default failure condition

### Related settings changes

The project configuration schema accommodates for default templates.
* XML: The `project-config.xml` file contains the `<default-template ref="...." />` element.
* DSL: Project configuration contains the `defaultTemplate = "..."` method. See a sample project configuration with a default template configured:

```Shell
object Project : Project({
    uuid = "2b241ffb-9019-4e60-9a3a-d5475ab1f312"
    extId = "ExampleProject"
    parentId = "_Root"
    name = "Example Project"
    defaultTemplate = "ExampleProject_MyDefaultTemplate"
    ...
    features {
        ...
    }
    ...
})
```

## Associating build configurations with templates
* You can [create new build configurations based on a template](creating-and-editing-build-configurations.md#Creating+Build+Configuration+from+Template).
* You can associate/attach any number of existing build configurations with/to a template: there is the __Attach to template__ option available from the __Actions__ button in the upper right corner of the screen.
 
When you associate an existing build configuration with a template, the build configuration inherits __all the settings__ defined in the template, and if there is a conflict, the template settings supersede the settings of the build configuration (except dependencies, parameters, and requirements). The settings inherited from a template [can be overridden](#Redefining+settings+inherited+from+template).

You can associate a build configuration to a template only if the template belongs to the current project or one of its parents. A template which has at least one associated build configuration cannot be deleted, the associated build configurations need to be detached first.

It is also possible to associate a build configuration with multiple templates.

### Associating build configuration with multiple templates

A build configuration can be attached to multiple templates using the _Attach to template_ action menu. In the __Actions__ menu, the __Manage templates__ action allows users to:
* change the order of templates, which affects the overlapping settings priority and the build step order: the priority is given to the settings from the template higher in the list. It affects such entities as parameter names, setting IDs (for build steps, triggers, features, artifact dependencies, and requirements), VCS roots or snapshot dependency source build configurations IDs if they overlap between templates attached to a build configuration.
* detach the build configuration from some templates (the user marks those to be detached and then has to apply their changes).
* detach the build configuration from all templates using the respectively named button in the same dialog window.

You can view all the templates attached to a build configuration on the __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page.

The settings from all templates the build configuration is attached to are inherited and you can view where they are inherited from in the view/edit __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes)__ pages.

When a build configuration is detached from some of its templates, all the effective (that is not overridden in the config and not overlapped by some higher priority template) settings inherited from them are copied to the configuration. The copying build configuration logic is the same as extracting a template. On moving a build configuration/project, the logic checks all templates to which the build configuration is attached.

#### Related settings changes
{id="related-settings-changes-1"}

* XML: If a build configuration is attached to a single template, the resulting config XML format stays the same as it was before (`ref` attribute of the `settings` element). If it is attached to a number of templates, then references to them are stored in a separate element under the `settings` node, as follows:

```Shell

<inherits>
<ref id="Template1_ExternalId" />
<ref id="Template2_ExternalId />
.....
</inherits>
```

* DSL: Kotlin DSL is extended, so within a build configuration definition users can use the `templates(vararg)` method accepting either external ids or DSL template instances (but not a mix of them, so if both templates defined inside and outside DSL are used in the same configuration, the external ids of both must be used).   
The older `template(...)` method and property __cannot__ be used multiple times within the same build type definition to indicate that it is inherited from multiple templates: following earlier implementation, each time this method is used, it overrides the previous template external ID. It is preserved for backward compatibility.

## Detaching build configurations from template

When you _detach a build configuration from a template_ using the __Detach from template__ option available in the __Actions__ menu of the __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes)s__ page, all settings from the template will be copied to the build configuration and enabled for editing.   
Note that if a build configuration is attached to multiple templates, the __Detach from template__ option becomes unavailable — use __[Manage templates](#Associating+build+configuration+with+multiple+templates)__ instead.

## Redefining settings inherited from template

A build configuration associated with a template inherits all its settings (marked as _inherited_ in the UI). Inherited settings cannot be deleted from an associated build configuration, but the inherited build steps, triggers, build features, failure conditions, artifact dependencies, and agent requirements can be disabled in it.

Modifying settings in the template will influence __all configurations__ associated with this template; however, it is possible to redefine most settings in an associated build configuration.

You can redefine almost all the build configuration settings (for example, build steps, [parameters](configuring-build-parameters.md), [build options](configuring-general-settings.md#Build+Options)). The only exceptions are snapshot dependencies and checkout rules which __cannot be redefined__.

Modified settings are highlighted with a yellow border, and the __Reset__ button appears on the right of the modified settings enabling you to revert the changes to the original settings of the template.

Note that if you redefine an inherited parameter in a build configuration, it will be saved with the inherited name and the new value. If you then rename this parameter in the template, you will end up with two parameters in the build configuration: the one with the original name and the redefined value, and the renamed one inherited from the template.

### Using parameter reference

Beside redefining settings as described above, you can redefine individual fields of the inherited settings via parameter references.

To introduce a configuration parameter reference, use the `%\ParameterName%` syntax in the template text field. Once introduced, this parameter appears on the __Parameters__ page of the build configuration template marked as requiring a value.

You can either specify the parameter's default value or leave it without any value. You can then define the actual value for the parameter in a build configuration associated with the template.

See also [Configuring Build Parameters](configuring-build-parameters.md).

#### Example of configuration parameters usage

Assume that you have two similar build configurations that differ only by checkout rules. For instance, checkout rules for the first configuration should contain `+:release_1_0 => .` and for the second one `+:trunk => .`. All other settings are equal. It would be useful to have one template to associate with both build configurations, but this means changing the checkout rules in each build configuration separately.

To do so, perform the following steps:
1. Extract a template from one of those configurations.
2. In the template settings, navigate to __Version Control Settings__, open the __Checkout rules__ dialog for the VCS root, and enter there: `%\checkout.rules%`
3. For the inherited build configuration, open the configuration settings page and on the __Parameters__ page specify the actual value for the `checkout.rules` configuration parameter.
4. For the second build configuration, use the __Associate with template__ option from __Actions__ and choose the template. Specify an appropriate value for the `checkout.rules` parameter right in the "Associate with Template" dialog. Click __Associate__.

As a result, you'll have two build configurations with different checkout rules, but associated with one template.

This way you can create a configuration parameter and then reference it from any build configuration, which has a text field.

### Ability to have pre- and post-steps in a template
[//]: # ([//]: # (AltHead: BBuildStepPlaceholders)

There is sometimes a need to define a common build step in a template, so that this step will be executed either before all build configuration steps or after them.   
For a given template, it is possible to define such steps and then define their placement in respect to the build configuration steps. All build configuration steps are represented as a placeholder in the _Reorder Build Steps_ dialog. The template steps can be placed before and/or after this placeholder.

<img src="reorder-build-steps.png" alt="Reorder build steps" width="544"/>

You can still customize the order of build steps in a template-based build configuration if necessary:
* Using the TeamCity web UI, it is possible to change the placement of the build configuration steps in respect to the template steps.
* Using [versioned settings](storing-project-settings-in-version-control.md), it is possible to change not only the placement of the build configuration steps in respect to the template steps, but also to reorder the steps of the template itself.

<anchor name="BuildConfigurationTemplate-Enforcingsettingsinheritedfromtemplate"/>

## Enforcing settings inherited from template
[//]: # ([//]: # (AltHead:Enforced settings)

If you want to enforce some settings on all the build configurations in the project so that other users could not redefine them, TeamCity provides this ability for all the build configurations in a project hierarchy. For instance, using enforced settings it is possible to set [agent side checkout](vcs-checkout-mode.md) everywhere, or ensure that all build configurations have some strict [execution timeout](build-failure-conditions.md#Common+Build+Failure+Conditions). Currently, it is possible to enforce build features, options, and parameters. Build steps and build requirements can also be enforced.

To enforce some settings in the project hierarchy, create a template with these settings. After that, a system administrator can set this template as the enforced settings' template in the project:

<img src="enforced-settings-template.png" alt="Enforced settings template" width="1164"/>

The enforced settings' template is similar to the default template as all of its settings are inherited in build configurations of the project hierarchy. The difference is that these inherited settings cannot be disabled or overridden.

The system administrator role is required to associate a project with a specific enforced settings' template. The template itself can be edited by a project administrator who can administer the project where the template is defined.

If the enforced settings' template is specified in a project and a different template is assigned as the enforced settings in a subproject, the template of the subproject will have a higher priority.

<seealso>
        <category ref="admin-guide">
            <a href="creating-and-editing-build-configurations.md">Creating and Editing Build Configurations</a>
            <a href="configuring-build-parameters.md">Configuring Build Parameters</a>
        </category>
</seealso>