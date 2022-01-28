---
title: "Resource dictionaries"
description: ".NET MAUI XAML resources dictionaries enable resources to be shared and reused throughout an app."
ms.date: 01/28/2022
---

# Resource dictionaries

A .NET Multi-platform App UI (.NET MAUI) `ResourceDictionary` is a repository for resources that are used by a .NET MAUI app. Typical resources that are stored in a `ResourceDictionary` include styles, control templates, data templates, converters, and colors.

[!INCLUDE [docs under construction](~/includes/preview-note.md)]

XAML resources that are stored in a `ResourceDictionary` can be referenced and applied to elements by using the `StaticResource` or `DynamicResource` markup extension. In C#, resources can also be defined in a `ResourceDictionary` and then referenced and applied to elements by using a string-based indexer. However, there's little advantage to using a `ResourceDictionary` in C#, as shared objects can be stored as fields or properties, and accessed directly without having to first retrieve them from a dictionary.

## Create resources

Every `VisualElement` derived object has a `Resources` property, which is a `ResourceDictionary` that can contain resources. Similarly, an `Application` derived object has a `Resources` property, which is a `ResourceDictionary` that can contain resources.

A .NET MAUI app can contain only a single class that derives from `Application`, but often makes use of many classes that derive from `VisualElement`, including pages, layouts, and views. Any of these objects can have its `Resources` property set to a `ResourceDictionary` containing resources. Choosing where to put a particular `ResourceDictionary` impacts where the resources can be used:

- Resources in a `ResourceDictionary` that is attached to a view, such as `Button` or `Label`, can only be applied to that particular object.
- Resources in a `ResourceDictionary` attached to a layout, such as `StackLayout` or `Grid`, can be applied to the layout and all the children of that layout.
- Resources in a `ResourceDictionary` defined at the page level can be applied to the page and to all its children.
- Resources in a `ResourceDictionary` defined at the application level can be applied throughout the app.

With the exception of implicit styles, each resource in resource dictionary must have a unique string key that's defined with the `x:Key` attribute.

The following XAML shows resources defined in an application level `ResourceDictionary` in the **App.xaml** file:

```xaml
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="ResourceDictionaryDemo.App">
    <Application.Resources>

        <Thickness x:Key="PageMargin">20</Thickness>

        <!-- Colors -->
        <Color x:Key="AppBackgroundColor">AliceBlue</Color>
        <Color x:Key="NavigationBarColor">#1976D2</Color>
        <Color x:Key="NavigationBarTextColor">White</Color>
        <Color x:Key="NormalTextColor">Black</Color>

        <!-- Implicit styles -->
        <Style TargetType="NavigationPage">
            <Setter Property="BarBackgroundColor"
                    Value="{StaticResource NavigationBarColor}" />
            <Setter Property="BarTextColor"
                    Value="{StaticResource NavigationBarTextColor}" />
        </Style>

        <Style TargetType="ContentPage"
               ApplyToDerivedTypes="True">
            <Setter Property="BackgroundColor"
                    Value="{StaticResource AppBackgroundColor}" />
        </Style>

    </Application.Resources>
</Application>
```

In this example, the resource dictionary defines a `Thickness` resource, multiple `Color` resources, and two implicit `Style` resources.<!-- For more information about the `App` class, see [.NET MAUI App Class](~/fundamentals/application-class.md).-->

> [!IMPORTANT]
> Inserting resources directly between the `Resources` property-element tags automatically creates a `ResourceDictionary` object. However, it's also valid to place all resources between optional `ResourceDictionary` tags.

## Consume resources

Each resource has a key that is specified using the `x:Key` attribute, which becomes its dictionary key in the `ResourceDictionary`. The key is used to reference a resource from the `ResourceDictionary` with the `StaticResource` or `DynamicResource` XAML markup extension.

The `StaticResource` markup extension is similar to the `DynamicResource` markup extension in that both use a dictionary key to reference a value from a resource dictionary. However, while the `StaticResource` markup extension performs a single dictionary lookup, the `DynamicResource` markup extension maintains a link to the dictionary key. Therefore, if the dictionary entry associated with the key is replaced, the change is applied to the visual element. This enables runtime resource changes to be made in an app. For more information about markup extensions, see [XAML markup extensions](~/xaml/markup-extensions/consume.md).

The following XAML example shows how to consume resources, and also define an additional resource in a `StackLayout`:

```xaml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="ResourceDictionaryDemo.MainPage"
             Title="Main page">
    <StackLayout Margin="{StaticResource PageMargin}"
                 Spacing="6">
        <StackLayout.Resources>
            <!-- Implicit style -->
            <Style TargetType="Button">
                <Setter Property="FontSize" Value="Medium" />
                <Setter Property="BackgroundColor" Value="#1976D2" />
                <Setter Property="TextColor" Value="White" />
                <Setter Property="CornerRadius" Value="5" />
            </Style>
        </StackLayout.Resources>

        <Label Text="This app demonstrates consuming resources that have been defined in resource dictionaries." />
        <Button Text="Navigate"
                Clicked="OnNavigateButtonClicked" />
    </StackLayout>
</ContentPage>
```

In this example, the `ContentPage` object consumes the implicit style defined in the application level resource dictionary. The `StackLayout` object consumes the `PageMargin` resource defined in the application level resource dictionary, while the `Button` object consumes the implicit style defined in the `StackLayout` resource dictionary. This results in the appearance shown in the following screenshot:

:::image type="content" source="media/resource-dictionaries/consuming.png" alt-text="Consuming resource dictionary resources.":::

> [!IMPORTANT]
> Resources that are specific to a single page shouldn't be included in an application level resource dictionary, as such resources will then be parsed at app startup instead of when required by a page. <!-- For more information, see [Reduce the Application Resource Dictionary Size](~/xamarin-forms/deploy-test/performance.md).-->

## Resource lookup behavior

The following lookup process occurs when a resource is referenced with the `StaticResource` or `DynamicResource` markup extension:

- The requested key is checked for in the resource dictionary, if it exists, for the element that sets the property. If the requested key is found, its value is returned and the lookup process terminates.
- If a match isn't found, the lookup process searches the visual tree upwards, checking the resource dictionary of each parent element. If the requested key is found, its value is returned and the lookup process terminates. Otherwise the process continues upwards until the root element is reached.
- If a match isn't found at the root element, the application level resource dictionary is examined.
- If a match still isn't found, a `XamlParseException` is thrown.

Therefore, when the XAML parser encounters a `StaticResource` or `DynamicResource` markup extension, it searches for a matching key by traveling up through the visual tree, using the first match it finds. If this search ends at the page and the key still hasn't been found, the XAML parser searches the `ResourceDictionary` attached to the `App` object. If the key still isn't found, an exception is thrown.

## Override resources

When resources share keys, resources defined lower in the visual tree will take precedence over those defined higher up. For example, setting an `AppBackgroundColor` resource to `AliceBlue` at the application level will be overridden by a page level `AppBackgroundColor` resource set to `Teal`. Similarly, a page level `AppBackgroundColor` resource will be overridden by a layout or view level `AppBackgroundColor` resource.

## Stand-alone resource dictionaries

A class derived from `ResourceDictionary` can also be in a stand-alone XAML file that doesn't have a corresponding code-behind file.

To create a stand-alone `ResourceDictionary`, add a new XAML file to the project and delete its code-behind file. Then, in the XAML file change the name of the base class to `ResourceDictionary`. In addition, remove the `x:Class` attribute from the root tag of the file.

> [!NOTE]
> A stand-alone `ResourceDictionary` must have a build action of **MauiXaml**.

The following XAML example shows a `ResourceDictionary` named **MyResourceDictionary.xaml**:

```xaml
<ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
    <DataTemplate x:Key="PersonDataTemplate">
        <ViewCell>
            <Grid RowSpacing="6"
                  ColumnSpacing="6">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="0.5*" />
                    <ColumnDefinition Width="0.2*" />
                    <ColumnDefinition Width="0.3*" />
                </Grid.ColumnDefinitions>
                <Label Text="{Binding Name}"
                       TextColor="{StaticResource NormalTextColor}"
                       FontAttributes="Bold" />
                <Label Grid.Column="1"
                       Text="{Binding Age}"
                       TextColor="{StaticResource NormalTextColor}" />
                <Label Grid.Column="2"
                       Text="{Binding Location}"
                       TextColor="{StaticResource NormalTextColor}"
                       HorizontalTextAlignment="End" />
            </Grid>
        </ViewCell>
    </DataTemplate>
</ResourceDictionary>
```

In this example, the `ResourceDictionary` contains a single resource, which is an object of type `DataTemplate`. **MyResourceDictionary.xaml** can be consumed by merging it into another resource dictionary.

<!--
By default, the linker will remove stand-alone XAML files from release builds when the linker behavior is set to link all assemblies. To ensure that stand-alone XAML files remain in a release build:

1. Add a custom `Preserve` attribute to the assembly containing the stand-alone XAML files. For more information, see [Preserving code](~/ios/deploy-test/linker.md).
1. Set the `Preserve` attribute at the assembly level:

    ```csharp
    [assembly:Preserve(AllMembers = true)]
    ```

For more information about linking, see [Linking Xamarin.iOS apps](~/ios/deploy-test/linker.md) and [Linking on Android](~/android/deploy-test/linker.md). -->

## Merge resource dictionaries

Resource dictionaries can be combined by merging one or more `ResourceDictionary` objects into another `ResourceDictionary`.

### Merge local resource dictionaries

A local `ResourceDictionary` file can be merged into another `ResourceDictionary` by creating a `ResourceDictionary` object whose `Source` property is set to the filename of the XAML file with the resources:

```xaml
<ContentPage ...>
    <ContentPage.Resources>
        <!-- Add more resources here -->
        <ResourceDictionary Source="MyResourceDictionary.xaml" />
        <!-- Add more resources here -->
    </ContentPage.Resources>
    ...
</ContentPage>
```

This syntax does not instantiate the `MyResourceDictionary` class. Instead, it references the XAML file. For that reason, when setting the `Source` property, a code-behind file isn't required, and the `x:Class` attribute can be removed from the root tag of the **MyResourceDictionary.xaml** file.

> [!IMPORTANT]
> The `ResourceDictionary.Source` property can only be set from XAML.

### Merge resource dictionaries from other assemblies

A `ResourceDictionary` can also be merged into another `ResourceDictionary` by adding it into the `MergedDictionaries` property of the `ResourceDictionary`. This technique allows resource dictionaries to be merged, regardless of the assembly in which they reside. Merging resource dictionaries from external assemblies requires the `ResourceDictionary` to have a build action set to **MauiXaml**, to have a code-behind file, and to define the `x:Class` attribute in the root tag of the file.

> [!WARNING]
> The `ResourceDictionary` class also defines a `MergedWith` property. However, this property has been deprecated and should no longer be used.

The following code example shows two resource dictionaries being added to the `MergedDictionaries` collection of a page level `ResourceDictionary`:

```xaml
<ContentPage ...
             xmlns:local="clr-namespace:ResourceDictionaryDemo"
             xmlns:theme="clr-namespace:MyThemes;assembly=MyThemes">
    <ContentPage.Resources>
        <ResourceDictionary>
            <!-- Add more resources here -->
            <ResourceDictionary.MergedDictionaries>
                <!-- Add more resource dictionaries here -->
                <local:MyResourceDictionary />
                <theme:DefaultTheme />
                <!-- Add more resource dictionaries here -->
            </ResourceDictionary.MergedDictionaries>
            <!-- Add more resources here -->
        </ResourceDictionary>
    </ContentPage.Resources>
    ...
</ContentPage>
```

In this example, a resource dictionary from the same assembly, and a resource dictionary from an external assembly, are merged into the page level resource dictionary. In addition, you can also add other `ResourceDictionary` objects within the `MergedDictionaries` property-element tags, and other resources outside of those tags.

> [!IMPORTANT]
> There can be only one `MergedDictionaries` property-element tag in a `ResourceDictionary`, but you can put as many `ResourceDictionary` objects in there as required.

When merged `ResourceDictionary` resources share identical `x:Key` attribute values, .NET MAUI uses the following resource precedence:

1. The resources local to the resource dictionary.
1. The resources contained in the resource dictionaries that were merged via the `MergedDictionaries` collection, in the reverse order they are listed in the `MergedDictionaries` property.

> [!TIP]
> Searching resource dictionaries can be a computationally intensive task if an app contains multiple, large resource dictionaries. Therefore, to avoid unnecessary searching, you should ensure that each page in an application only uses resource dictionaries that are appropriate to the page.