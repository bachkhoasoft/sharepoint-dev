---
title: Customize OneDrive sites
description: The actual patterns that you can use with the add-in model to customize OneDrive sites.
ms.date: 03/08/2023
ms.localizationpriority: high
---

# Customize OneDrive sites

OneDrive sites can be customized in Microsoft 365 or with the add-in model in general, based on company requirements. Actual techniques to perform this customization are different than in the on-premises scenario because only add-in model techniques can be used.

> [!IMPORTANT]
> This applies to **only** the *classic* OneDrive experience in SharePoint Online. If you are using the new default experience, this is not supported. Modern or new experiences of OneDrive do not support custom branding. Tenant administrators can control the default experience from the SharePoint Online administrative settings.
>
> Patterns for Dedicated and on-premises are identical with the add-in model techniques, but there are differences in the possible technologies that can be used.

## Why customize OneDrive sites?

There are numerous aspects to applying customizations to OneDrive sites. You can customize these sites because they're SharePoint sites, but you should always consider the short-term and long-term impact of the customizations.

Following are the high-level guidelines for customizing OneDrive sites:

- Apply branding customizations by using Microsoft 365 themes or the SharePoint site theming engine.
- If theme engines aren't enough, you can adjust some CSS settings by using alternate CSS options.
- Avoid customizing OneDrive sites by using custom master pages because this causes you extra long-term costs and challenges with future updates.
  - In most cases, you can achieve all common branding scenarios with themes and alternate CSS.
  - If you choose to use custom master pages, be prepared to apply changes to the sites when major functional updates are applied to Microsoft 365.
- You can use JavaScript embedding to modify or hide functionalities from the site.
- You can use CSOM to control, for example, language or regional settings in the OneDrive sites (see new APIs).
- We don't recommend using content types and site columns in OneDrive sites. Use OneDrive sites for personal unstructured data and documents. Use team sites and collaboration sites for company data and documents, where you can use whatever information management policies and metadata you want.

Customizations are definitely supported in Microsoft 365, and you can keep on using them with OneDrive sites. We just want to ensure that you consider the impact of these customizations from an operational and maintenance perspective. This isn't specific for SharePoint, but rather a rule of thumb for any IT solution built with any platform.

Following is an example of an OneDrive site that has been customized using the previous guidelines. In this case, the end result has been achieved with a combination of Microsoft 365 themes, a site theme, and use of the JavaScript embedding pattern.

![A customized OneDrive site](media/Customization-Options-For-OD4B-Sites/Customization-Options-For-OD4B-Sites-01.png)

## Challenge with applying OneDrive site customizations

Each OneDrive site currently uses the same architecture as that of a personal site or My Sites, which was used in SharePoint 2007 and SharePoint 2010. This means that each OneDrive site is their own site collection, and we don't have any centralized location to apply branding or any other customizations.

![Each OneDrive site is its own site collection under the personal managed path, and the url is created based on the assigned user profile. In the image, three sites are listed as child sites. The URL of the first child site ends with /bill_contoso_com. The second ends with /vesa_contoso_com. The third ends with /john_contoso_com.](media/Customization-Options-For-OD4B-Sites/Customization-Options-For-OD4B-Sites-02.png)

The classic solution that was used to apply needed configurations to the OneDrive sites was based on feature stapling at the farm level. This meant that you deployed a farm solution to your SharePoint farm and used the Feature Framework to associate your custom feature to be activated each time a My Site was created, which was then responsible for applying needed customizations.

This approach doesn't work in Microsoft 365 because it requires a farm solution to be deployed, and that is impossible with Microsoft 365 sites. Therefore, we need to look at alternatives to apply the needed changes to the sites.

In Microsoft 365, there's no centralized event raised that we could attach our custom code to when an OneDrive site is created. Therefore, we need to think about alternative solutions, which is common with add-in model approaches. Don't get stuck on old models; think about how to achieve the same end result using new APIs and technologies. From a requirement perspective, it doesn't really matter how we apply the customizations to the sites, as long as they're applied, because the business requirement isn't to use feature stapling, it’s to apply needed customizations by using whatever technical mechanism is supported.

## Options for applying customizations

There are four different mechanisms to apply centralized customizations to OneDrive sites in Microsoft 365. You could also consider a manual option as the fifth one, but in the case of having hundreds or thousands of OneDrive sites, using the manual option isn't realistic. The options are:

- Microsoft 365 suite level settings (Microsoft 365 themes and other settings)
- Hidden app part with user context
- Pre-create and apply configuration
- Remote timer job based on user profile updates

Each of these options has advantages and disadvantages, and the right option depends on your detailed business requirements. You can also apply some settings from the Office 365 suite level, but you would be looking for some more specifics, so actual customizations are needed.

### Microsoft 365 suite level settings

Microsoft 365 is more than just SharePoint. You can find more services that aren't based on the SharePoint architecture, such as Delve and Yammer. This means that the enterprise branding and configuration isn't just about controlling what we have in the SharePoint sites; rather, we should be thinking about the overall end user experience and how we can provide consistent configurations across different services.

A classic example of these enterprise requirements is branding, and for that we already have Microsoft 365 theming introduced, which can be used to control some level of branding.

The following diagram shows the current settings for Microsoft 365 theming, which can be applied across all Microsoft 365 services.

![Displays the Microsoft 365 site, showing the custom theming tab page, entitled Manage custom themes for your organization, Customize Office 365 to reflect your oganization's brand. Settings are available for Custom logo, URL for a clickable logo, Background image, Accent color, Navigation bar background color, Text and icons color, and App menu icon color.](media/Customization-Options-For-OD4B-Sites/Customization-Options-For-OD4B-Sites-03.png)

Because by default Office 365 theme settings control the OneDrive site suite bar, you'll most likely use these options together with other options to ensure that you can provide at least the right branding elements across your OneDrive sites. Notice that when you change, for example, the Office 365 theme settings in the Office 365 admin tool, it does take quite a long time to get the settings applied for OneDrive sites, so be patient.

### Hidden app part with user context

This approach uses a centralized landing page as the location for starting the needed customization process. This means that you need one centralized location, such as a company intranet home page, where users always land when they open their browser. This is a typical process for mid-sized and larger enterprises where the corporate landing page is controlled by using Group Policy settings in Active Directory. This ensures that end users can't override the default welcome page of the company domain-joined browsers.

When a user arrives on the intranet site, a hidden app part on the page starts the customization process. It can actually be responsible for the entire OneDrive site creation as well because normally a user would have to visit the OneDrive site once before the site creation process starts. The hidden app part hosts a page from the provider-hosted add-in hosted in Azure. This page is then responsible for starting the customization process.

Let’s have a closer look at the logical design of this approach.

![Diagram to show relationships. The App part on the SharePoint site uses instantiate to go to Provider Hosted Apps. Provider Hosted Apps uses Add Message to go to Storage Queue. Storage Queue uses instantiate to go to WebJob. WebJob uses Apply modifications to go to the OneDrive site.](media/Customization-Options-For-OD4B-Sites/Customization-Options-For-OD4B-Sites-04.png)

1. Place hidden app part on centralized site where end users land. Typically this is the corporate intranet front page.
1. App part is hosting a page from the provider-hosted add-in, where in the server-side code, we initiate the customization process by adding needed metadata to the Azure Storage Queue. This means that this page only receives the customization request, but doesn't actually apply any changes to keep the processing time normal.
1. This is the actual Azure Storage Queue, which receives the messages to queue for processing. This way we can handle the customization controlling process asynchronously so that it doesn't really matter how long the end user stays on the front page of the intranet. If the customization process was synchronous, we would be dependent on the end user keeping the browser open in the intranet front page until page execution was finalized. This would definitely not be an optimal end user experience.
1. WebJob is hooked to follow the Azure Storage Queue, which is called when a new item is placed in the Storage Queue. This WebJob receives the needed parameters and metadata from the queued message to access the right site collection. WebJob uses an app-only token and has been granted the needed permissions to manipulate site collections at the tenant level.
1. Actual customizations are applied one-by-one to the sites of the users who visit the intranet front page to start the process.

This is the most reliable process for ensuring that there are accurate configurations in the OneDrive sites. You can easily add customization versioning logic to the process, which applies any needed updates to the OneDrive sites when there's an update needed and the user visits the intranet front page next time. This option does, however, require that you have that centralized location where your end users are landing.

If you're familiar with classic SharePoint development models with farm solutions, this process is similar to one-time executing timer jobs.

### Pre-create and apply configuration

This option relies on the pre-creation of the OneDrive sites before users access them. This can be achieved by using relatively new API that provides us with a way to create OneDrive sites for specific users in a batch process by using either CSOM or REST. The needed code can be initiated by using a PowerShell script or by writing actual code that calls the remote APIs.

![An administrator uses, pre-create and customize, to create an OneDrive site.](media/Customization-Options-For-OD4B-Sites/Customization-Options-For-OD4B-Sites-05.png)

1. Administrator uses the remote creation APIs to create OneDrive sites for users and applies the needed customizations to the OneDrive sites as part of the script process.
1. Actual OneDrive sites are created to Microsoft 365 for specific users and associated to their user profiles.

While this is a reliable process, you have to manage new persons and updates manually, which could mean more work than using the hidden app part approach. Nevertheless, this is a valid approach and is especially useful if you're migrating from some other file sharing solution to the OneDrive and want to avoid users needing to access the OneDrive site one time before actual site creation is started.

### Remote timer job based on user profile updates

This approach means scanning through user profiles and checking to whom the OneDrive site has been created and then applying the changes to the sites as needed. This would mean scheduled jobs running outside of SharePoint, which periodically check status and perform needed customizations. Scheduled jobs could be running as WebJobs in Azure or as PowerShell scripts in your own Windows scheduler. Obviously, the scale of the deployment has a huge impact on the chosen scheduling option.

![A Remote timer job uses, Loop through site collections, to customize each site.](media/Customization-Options-For-OD4B-Sites/Customization-Options-For-OD4B-Sites-06.png)

1. Scheduled task is initiated, which accesses user profiles to check who has the OneDrive site provisioned.
1. Actual sites are customized one-by-one based on the business requirements.

One of the key downsides of this option is that there can clearly be a situation where a user can access the OneDrive sites before the customizations have been applied. At the same time, this option is an interesting add-on for other options to ensure that end users haven't changed any of the required settings on the sites or to check that the OneDrive site content aligns with the company policies.

## See also

- [Customizing OneDrive for Business sites with app model (blog post)](https://blogs.msdn.microsoft.com/vesku/2015/01/05/customizing-onedrive-for-business-sites-with-app-model/)
- [Classic app part and sync process for OneDrive site customization (GitHub)](https://github.com/SharePoint/PnP/tree/master/Solutions/Provisioning.OneDrive)
- [Pre-create OneDrive sites for users](https://github.com/SharePoint/PnP/tree/master/Samples/Provisioning.OneDriveProvisioning)
- [SharePoint site branding and page customization solutions](sharepoint-site-branding-and-page-customization-solutions.md)
- [Branding and site provisioning solutions for SharePoint](branding-and-site-provisioning-solutions-for-sharepoint.md)
