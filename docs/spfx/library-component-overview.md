---
title: Using library component type in SharePoint Framework
description: Using library component type in SharePoint Framework
ms.date: 05/03/2023
ms.localizationpriority: high
---

# Using library component type in SharePoint Framework

The library component type in the SharePoint Framework (SPFx) enables you to have independently versioned and deployed code served automatically for the SharePoint Framework components with a deployment through an app catalog. Library components provide you an alternative option to create shared code, which can be then used and referenced cross all the components in the tenant.

You can create a library component solution by selecting **Library** as the **Component Type** when creating new projects with the SPFx Yeoman generator.

Library components have following characteristics:

- You can only host one library component version at the time in a tenant.
- You can deploy and host library components in the tenant app catalog or the site app catalog.

You can reference library component in the SharePoint solution by defining the dependency in the **package.json** file. The bundling process detects this dependency and adds it as to the consuming component's manifest. This dependency will then be detected by the SharePoint Framework at runtime and load the library before loading the component's bundle.
