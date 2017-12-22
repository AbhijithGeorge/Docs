---
title: Working with multiple environments in ASP.NET Core
author: ardalis
description: Learn how ASP.NET Core provides support for controlling app behavior across multiple environments.
keywords: ASP.NET Core,Environment settings,ASPNETCORE_ENVIRONMENT
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/environments
---
# Working with multiple environments

en-us/

By [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core provides support for setting application behavior at runtime with environment variables. 

[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/environments/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))

## Environments

ASP.NET Core reads the environment variable `ASPNETCORE_ENVIRONMENT` at application startup and stores that value in [IHostingEnvironment.EnvironmentName](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.environmentname?view=aspnetcore-2.0#Microsoft_AspNetCore_Hosting_IHostingEnvironment_EnvironmentName). `ASPNETCORE_ENVIRONMENT` can be set to any value, but [three values are supported by the framework](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.environmentname?view=aspnetcore-2.0): `Development`, `Staging`, and `Production`. 

[!code-csharp[Main](environments/sample/WebApp1/Startup.cs?name=snippet)]

The preceding code:

* Calls [UseDeveloperExceptionPage](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.developerexceptionpageextensions.usedeveloperexceptionpage?view=aspnetcore-2.0#Microsoft_AspNetCore_Builder_DeveloperExceptionPageExtensions_UseDeveloperExceptionPage_Microsoft_AspNetCore_Builder_IApplicationBuilder_) and [UseBrowserLink](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.browserlinkextensions.usebrowserlink?view=aspnetcore-2.0#Microsoft_AspNetCore_Builder_BrowserLinkExtensions_UseBrowserLink_Microsoft_AspNetCore_Builder_IApplicationBuilder_) when `ASPNETCORE_ENVIRONMENT` is set to `Development`.
* Calls [UseExceptionHandler](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.exceptionhandlerextensions.useexceptionhandler?view=aspnetcore-2.0#Microsoft_AspNetCore_Builder_ExceptionHandlerExtensions_UseExceptionHandler_Microsoft_AspNetCore_Builder_IApplicationBuilder_) when the value of `ASPNETCORE_ENVIRONMENT` is set one of the following:

    * `Staging`
    * `Production`
    * `Staging_2`

The [Environment Tag Helper ](xref:mvc/views/tag-helpers/builtin-th/environment-tag-helper) uses the value of `IHostingEnvironment.EnvironmentName` to include or exclude markup in the element:

[!code-html[Main](environments/sample/WebApp1/Pages/About.cshtml)]

The `PageModel` for the preceding Razor page sets the `Message` property with the value of `ASPNETCORE_ENVIRONMENT`:

[!code-csharp[Main](environments/sample/WebApp1/Pages/About.cshtml.cs)]

Note: On Windows and macOS, environment variables and values are insensitive. Linux environment variables and values are **case sensitive** by default. 

### Development

The development environment can enable features should not be exposed in production. For example, the ASP.NET Core templates enable the [developer exception page](xref:fundamentals/error-handling#the-developer-exception-page) in the development environment.

The environment for local machine development is typically set in the *Properties\launchSettings.json* file of the project. Environment values set in *launchSettings.json* override values set in the system environment.

The following XML shows two profiles from a *launchSettings.json* file:

[!code-xml[Main](environments/sample/WebApp1/Properties/launchSettings.json)]

If you're using Visual Studio, the environment can be configured in your project's debug profiles. Debug profiles specify the [server](xref:fundamentals/servers/index) to use when launching the application and any environment variables to be set. Your project can have multiple debug profiles that set environment variables differently. You manage these profiles by using the **Debug** tab of your web application project's **Properties** menu. The values you set in project properties are persisted in the *launchSettings.json* file, and you can also configure profiles by editing that file directly.

The profile for IIS Express is shown here:

![Project Properties Setting Environment variables](environments/_static/project-properties-debug.png)

Here is a `launchSettings.json` file that includes profiles for `Development` and `Staging`:


Changes made to project profiles may not take effect until the web server used is restarted (in particular, Kestrel must be restarted before it will detect changes made to its environment).

>[!WARNING]
> Environment variables stored in *launchSettings.json* are not secured in any way and will be part of the source code repository for your project, if you use one. **Never store credentials or other secret data in this file.** If you need a place to store such data, use the *Secret Manager* tool described in [Safe storage of app secrets during development](xref:security/app-secrets).


### Production

The `Production` environment is the environment in which the application runs when it is live and being used by end users. This environment should be configured to maximize security, performance, and application robustness. Some common settings that a production environment might have that would differ from development include:

* Turn on caching

* Ensure all client-side resources are bundled, minified, and potentially served from a CDN

* Turn off diagnostic ErrorPages

* Turn on friendly error pages

* Enable production logging and monitoring (for example, [Application Insights](https://azure.microsoft.com/documentation/articles/app-insights-asp-net-five/))

This is by no means meant to be a complete list. It's best to avoid scattering environment checks in many parts of your application. Instead, the recommended approach is to perform such checks within the application's `Startup` class(es) wherever possible

## Setting the environment

The method for setting the environment depends on the operating system.

### Windows
To set the `ASPNETCORE_ENVIRONMENT` for the current session, if the app is started using `dotnet run`, the following commands are used

**Command line**
```
set ASPNETCORE_ENVIRONMENT=Development
```
**PowerShell**
```
$Env:ASPNETCORE_ENVIRONMENT = "Development"
```

These commands take effect only for the current window. When the window is closed, the ASPNETCORE_ENVIRONMENT setting reverts to the default setting or machine value. In order to set the value globally on Windows open the **Control Panel** > **System** > **Advanced system settings** and add or edit the `ASPNETCORE_ENVIRONMENT` value.

![System Advanced Properties](environments/_static/systemsetting_environment.png)

![ASPNET Core Environment Variable](environments/_static/windows_aspnetcore_environment.png) 

**web.config**

See the *Setting environment variables* section of the [ASP.NET Core Module configuration reference](xref:hosting/aspnet-core-module#setting-environment-variables) topic.

**Per IIS Application Pool**

If you need to set environment variables for individual apps running in isolated Application Pools (supported on IIS 10.0+), see the *AppCmd.exe command* section of the [Environment Variables \<environmentVariables>](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) topic in the IIS reference documentation.

### macOS
Setting the current environment for macOS can be done in-line when running the application;

```bash
ASPNETCORE_ENVIRONMENT=Development dotnet run
```
or using `export` to set it prior to running the app.

```bash
export ASPNETCORE_ENVIRONMENT=Development
``` 
Machine level environment variables are set in the *.bashrc*  or *.bash_profile* file. Edit the file using any text editor and add the following statment.

```
export ASPNETCORE_ENVIRONMENT=Development
```  

### Linux
For Linux distros, use the `export` command at the command line for session based variable settings and *bash_profile* file for machine level environment settings.

## Determining the environment at runtime

The `IHostingEnvironment` service provides the core abstraction for working with environments. This service is provided by the ASP.NET hosting layer, and can be injected into your startup logic via [Dependency Injection](dependency-injection.md). The ASP.NET Core web site template in Visual Studio uses this approach to load environment-specific configuration files (if present) and to customize the app's error handling settings. In both cases, this behavior is achieved by referring to the currently specified environment by calling `EnvironmentName` or
`IsEnvironment` on the instance of `IHostingEnvironment` passed into the appropriate method.

> [!NOTE]
> If you need to check whether the application is running in a particular environment, use `env.IsEnvironment("environmentname")` since it will correctly ignore case (instead of checking if `env.EnvironmentName == "Development"` for example).

For example, you can use the following code in your Configure method to setup environment specific error handling:


If the app is running in a `Development` environment, then it enables the runtime support necessary to use the "BrowserLink" feature in Visual Studio, development-specific error pages (which typically should not be run in production) and special database error pages (which provide a way to apply migrations and should therefore only be used in development). Otherwise, if the app is not running in a development environment, a standard error handling page is configured to be displayed in response to any unhandled exceptions.

You may need to determine which content to send to the client at runtime, depending on the current environment. For example, in a development environment you generally serve non-minimized scripts and style sheets, which makes debugging easier. Production and test environments should serve the minified versions and generally from a CDN. You can do this using the Environment [tag helper](../mvc/views/tag-helpers/intro.md). The Environment tag helper will only render its contents if the current environment matches one of the environments specified using the `names` attribute.


To get started with using tag helpers in your application see [Introduction to Tag Helpers](../mvc/views/tag-helpers/intro.md).

## Startup conventions

ASP.NET Core supports a convention-based approach to configuring an application's startup based on the current environment. You can also programmatically control how your application behaves according to which environment it is in, allowing you to create and manage your own conventions.

When an ASP.NET Core application starts, the `Startup` class is used to bootstrap the application, load its configuration settings, etc. ([learn more about ASP.NET startup](startup.md)). However, if a class exists named `Startup{EnvironmentName}` (for example `StartupDevelopment`), and the `ASPNETCORE_ENVIRONMENT` environment variable matches that name, then that `Startup` class is used instead. Thus, you could configure `Startup` for development, but have a separate `StartupProduction` that would be used when the app is run in production. Or vice versa.

> [!NOTE]
> Calling `WebHostBuilder.UseStartup<TStartup>()` overrides configuration sections.

In addition to using an entirely separate `Startup` class based on the current environment, you can also make adjustments to how the application is configured within a `Startup` class. The `Configure()` and `ConfigureServices()` methods support environment-specific versions similar to the `Startup` class itself, of the form `Configure{EnvironmentName}()` and `Configure{EnvironmentName}Services()`. If you define a method `ConfigureDevelopment()` it will be called instead of `Configure()` when the environment is set to development. Likewise, `ConfigureDevelopmentServices()` would be called instead of `ConfigureServices()` in the same environment.


## Additional Resources

* [Configuration](xref:fundamentals/configuration/index)
* [IHostingEnvironment.EnvironmentName](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.environmentname?view=aspnetcore-2.0#Microsoft_AspNetCore_Hosting_IHostingEnvironment_EnvironmentName)
* 
