---
title: Application Parts in ASP.NET Core
author: rick-anderson
description: Share controllers, view, Razor Pages and more with Application Parts in ASP.NET Core
ms.author: riande
ms.date: 05/14/2019
uid: mvc/extensibility/app-parts
---

# Share controllers, views, Razor Pages and more with Application Parts in ASP.NET Core
=======

<!-- DO NOT MAKE CHANGES BEFORE https://github.com/aspnet/AspNetCore.Docs/pull/12376 Merges -->

By [Rick Anderson](https://twitter.com/RickAndMSFT)

[View or download sample code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts) ([how to download](xref:index#how-to-download-a-sample))

An *Application Part* is an abstraction over the resources of an app. Application Parts allow ASP.NET Core to discover controllers, view components, tag helpers, Razor Pages, razor compilation sources, and more. [AssemblyPart](/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.assemblypart#Microsoft_AspNetCore_Mvc_ApplicationParts_AssemblyPart) is an Application part. `AssemblyPart` encapsulates an assembly reference and exposes types and compilation references.

*Feature providers* work with application parts to populate the features of an ASP.NET Core app. The main use case for application parts is to configure an app to discover (or avoid loading) ASP.NET Core features from an assembly. For example, you might want to share common functionality between multiple apps. Using Application Parts, you can share an assembly (DLL) containing controllers, views, Razor Pages, razor compilation sources, Tag Helpers, and more with multiple apps. Sharing an assembly is preferred to duplicating code in multiple projects.

ASP.NET Core apps load features from <xref:System.Web.WebPages.ApplicationPart>. The <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> class represents an application part that's backed by an assembly.

## Load ASP.NET Core features

Use the `ApplicationPart` and `AssemblyPart` classes to discover and load ASP.NET Core features (controllers, view components, etc.). The <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> tracks the application parts and feature providers available. `ApplicationPartManager` is configured in `Startup.ConfigureServices`:

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup.cs?name=snippet)]

The following code provides an alternative approach to configuring `ApplicationPartManager` using `AssemblyPart`:

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup2.cs?name=snippet)]

The preceding two code samples load the `SharedController` from an assembly. The `SharedController` is not in the applications project. See the [WebAppParts solution](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample1/WebAppParts) sample download.

### Include views

To include views in the assembly:

* Add the following markup to the shared project file:

  ```csproj
    <ItemGroup>
      <EmbeddedResource Include = "Views\**\*.cshtml" />
    </ ItemGroup >
  ```

* Add the <xref:Microsoft.Extensions.FileProviders.EmbeddedFileProvider> to the <xref:Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine>:

[!code-csharp[](./app-parts/sample1/WebAppParts/StartupViews.cs?name=snippet&highlight=3-7)]

### Prevent loading resources

Application parts can be used to *avoid* loading resources in a particular assembly or location. Add or remove members of the  <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> collection to hide or make available resources. The order of the entries in the `ApplicationParts` collection isn't important. Configure the `ApplicationPartManager` before using it to configure services in the container. For example, configure the `ApplicationPartManager` before invoking `AddControllersAsServices`. Call `Remove` on the `ApplicationParts` collection to remove a resource.

The following code uses <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> to remove `MyDependentLibrary` from the app:
[!code-csharp[](./app-parts/sample1/WebAppParts/StartupRm.cs?name=snippet)]

The `ApplicationPartManager` includes parts for:

* The apps assembly and dependent assemblies.
* `Microsoft.AspNetCore.Mvc.TagHelpers`
* `Microsoft.AspNetCore.Mvc.Razor`.

## Application feature providers

Application feature providers examine application parts and provide features for those parts. There are built-in feature providers for the following ASP.NET Core features:

* [Controllers](/dotnet/api/microsoft.aspnetcore.mvc.controllers.controllerfeatureprovider)
* [Tag Helpers](/dotnet/api/microsoft.aspnetcore.mvc.razor.taghelpers.taghelperfeatureprovider)
* [View Components](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponents.viewcomponentfeatureprovider)

Feature providers inherit from <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1>, where `T` is the type of the feature. Feature providers can be implemented for any of the previously listed feature types. The order of feature providers in the `ApplicationPartManager.FeatureProviders` can impact run time behavior. Later added providers can react to actions taken by earlier added providers.

### Generic controller feature

ASP.NET Core ignores [generic controllers](/dotnet/csharp/programming-guide/generics/generic-classes). A generic controller has a type parameter (for example, `MyController<T>`). The following sample adds generic controller instances for a specified list of types.

[!code-csharp[](./app-parts/sample2/AppPartsSample/GenericControllerFeatureProvider.cs?name=snippet)]

The types are defined in `EntityTypes.Types`:

[!code-csharp[](./app-parts/sample2/AppPartsSample/Models/EntityTypes.cs?name=snippet)]

The feature provider is added in `Startup`:

[!code-csharp[](./app-parts/sample2/AppPartsSample/Startup.cs?name=snippet)]

Generic controller names used for routing are of the form *GenericController`1[Widget]* rather than *Widget*. The following attribute modifies the name to correspond to the generic type used by the controller:

[!code-csharp[](./app-parts/sample2/AppPartsSample/GenericControllerNameConvention.cs)]

The `GenericController` class:

[!code-csharp[](./app-parts/sample2/AppPartsSample/GenericController.cs)]

### Display available features

The features available to an app can be enumerated by by requesting an `ApplicationPartManager` through [dependency injection](../../fundamentals/dependency-injection.md):

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

The [download sample](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample2) uses the preceding code to display the app features.