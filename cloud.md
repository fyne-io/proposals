# Cloud features and providers

Part of a complete software ecosystem is the ability for apps to stay in sync. Through preference and/or document transfer users can be editing/reading/playing on one device, move to another and continue where they left off. With a truly cross-platform toolkit like Fyne this should be seamless across all devices.

In addition to the [vision](https://fyne.io/vision/) that *it should be free and simple to develop an application* we must strive to make **usage** of the applications developed under this technology equally democratised. As such any sync solution should, be available to everybody for free or as cheap as possible for the level of functionality delivered.

It is possible that this solution may involve various levels of functionality, for example someone just looking to sync their preferences could perhaps get this for free. However an individual or business working with classified documents that must remain encrypted may wish to pay more for a more sophisticated service.

## Background

The popular platforms now have cloud integration, this is essential in many cases and nice to have in a majority.
To remain competitive this is an integration we should offer.
In most cases the cloud is tied to the platform, but as a platform-agnostic toolkit we need to be able to offer a way for the cloud provision to work on any device.
As a platform that is democratising access to techology we also need to provide choice.

## Architecture / API

## `fyne.io/fyne`

* CloudProvider interface
    - ProviderName
    - ProviderIcon
    - ProviderDescription

* CloudProviderPreferences interface
    - CloudPreferences(fyne.App) fyne.Preferences

* CloudProviderStorage interface
    - CloudStorage(fyne.App) fyne.Storage

* App.SetCloudProvider(CloudProvider) // configure cloud for this app

* App.CloudProvider() CloudProvider // get the (if any) configured provider

## `fyne.io/cloud`

This package imports all of the known providers.

Also a place for lots of docs and information about implementing etc. :)

* Enable(fyne.App)
  - auto-wires up the user's choice, assuming they have previously chosen / configured a cloud provider.
* Register(fyne.CloudProvider)
* ShowSettings(fyne.App, fyne.Window)
  - will show the options that are available, leading to a configuration workflow.

## `fyne.io/cloud/provider/xxx`

NewProvider() fyne.CloudProvider

## Design

The cloud selection screen is the only design element planned at this stage.
It will be developed as part of the `fyne.io/cloud` repository.
The elements displayed will be inferred from the details in each `CloudProvider` instance.

## Implementation

To verify the design we should build more than one provider as part of the initial work.
These may be DropBox and AletheiaWare, but could change during development

## Prior art


