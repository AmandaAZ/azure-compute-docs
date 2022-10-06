---
title: Enable SAP Principal Propagation for live OData feeds with Power Query 
description: Learn about configuring SAP Principal Propagation for live OData feeds with Power Query 
author: MartinPankraz

ms.service: virtual-machines-sap
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/10/2022
ms.author: mapankra
---
# Enable SAP Principal Propagation for live OData feeds with Power Query 

Working with SAP datasets in Microsoft Excel or Power BI is a common requirement for customers.

This article describes the required configurations and components to enable SAP dataset consumption via OData with [Power Query](https://docs.microsoft.com/power-query/power-query-what-is-power-query). The SAP data integration is considered "live" because it can be refreshed from clients like Excel or Power BI on-demand--in contrast to Excel exports (like ALV CSV exports) for instance. Those exports are static by nature and have no continuous relationship with the data origin.

The article puts emphasis on end-to-end user mapping between the known Azure AD identity in Power Query and the SAP backend user. This mechanism is often referred to as SAP Principal Propagation.

The focus of the described configuration is on the [Azure API Management](https://azure.microsoft.com/services/api-management/), [SAP Gateway](https://help.sap.com/viewer/product/SAP_GATEWAY), [SAP OAuth 2.0 Server with AS ABAP](https://help.sap.com/docs/SAP_NETWEAVER_750/e815bb97839a4d83be6c4fca48ee5777/0b899f00477b4034b83aa31764361852.html), and OData sources, but the concepts used apply to any web-based resource.

> [!IMPORTANT]
> Note: SAP Principal Propagation ensures user-mapping to the licensed named SAP user. For any SAP license related questions please contact your SAP representative.

## Overview of Microsoft products with SAP integration

Integrations between SAP products and the Microsoft 365 portfolio range from custom codes, and partner add-ons to fully customized Office products. Here are a couple of examples:

-   [SAP Analysis for Microsoft Office Excel and PowerPoint](https://help.sap.com/docs/SAP_BUSINESSOBJECTS_ANALYSIS_OFFICE/ca9c58444d64420d99d6c136a3207632/ebf198667aa54740b9049d9da804a901.html)

-   [SAP Analytics Cloud, add-in for Microsoft Office](https://help.sap.com/docs/SAP_ANALYTICS_CLOUD_OFFICE/c637c9ff5d61457eb415ce161e38e57b/c9217302603a4fd6baa0fe6a6e780f8d.html)

-   [Access SAP Data Warehouse Cloud with Microsoft Excel](https://blogs.sap.com/2022/05/17/access-sap-data-warehouse-cloud-with-saps-microsoft-excel-add-ins/)

-   [SAP HANA Connector for Power Query](https://docs.microsoft.com/power-query/connectors/sap-hana/overview)

-   [Custom Excel Macros to interact with SAP back ends](https://help.sap.com/docs/SAP_BUSINESSOBJECTS_ANALYSIS_OFFICE/ca9c58444d64420d99d6c136a3207632/f270fd456c9b1014bf2c9a7eb0e91070.html)

-   [Export from SAP List Viewer (ALV) to Microsoft Excel](https://help.sap.com/docs/ABAP_PLATFORM_NEW/b1c834a22d05483b8a75710743b5ff26/4ec38f8788d22b90e10000000a42189d.html)

The mechanism described in this article uses the standard built-in OData capabilities of Power Query and puts emphasis for SAP landscapes deployed on Azure. Address on-premises landscapes with the Azure API Management [self-hosted Gateway](https://docs.microsoft.com/azure/api-management/self-hosted-gateway-overview).

For more information on which Microsoft products support Power Query, see [the Power Query documentation](https://docs.microsoft.com/power-query/power-query-what-is-power-query#where-can-you-use-power-query).

## Setup considerations

End users have a choice between local desktop or web-based clients (for instance Excel or Power BI). The client execution environment needs to be considered for the network path between the client application and the target SAP workload. Network access solutions such as VPN aren't in scope for apps like Excel for the web.

[Azure API Management](https://azure.microsoft.com/services/api-management/) reflects local and web-based environment needs with different deployment modes that can be applied to Azure landscapes ([internal](https://learn.microsoft.com/azure/api-management/api-management-using-with-internal-vnet?tabs=stv2)
or [external](https://learn.microsoft.com/azure/api-management/api-management-using-with-vnet?tabs=stv2)). Internal refers to instances that are fully restricted to a private virtual network whereas external retains public access to Azure API Management. On-premises installations require a hybrid deployment to apply the approach as is using the Azure API Management [self-hosted Gateway](https://docs.microsoft.com/azure/api-management/self-hosted-gateway-overview).

Power Query requires matching API service URL and Azure AD application ID URL. Since standard Azure domains, like `apim-service-name.azure-api.net`, can't be registered as application ID, a [custom domain for Azure API Management](https://docs.microsoft.com/azure/api-management/configure-custom-domain) needs to be set up.

[SAP Gateway](https://help.sap.com/docs/SAP_GATEWAY) needs to be configured to expose the desired target OData services. Discover and activate available services via SAP transaction code `/IWFND/MAINT_SERVICE`. For more information, see SAP's [OData configuration](https://help.sap.com/docs/SAP_GATEWAY).

## Azure API Management custom domain configuration

See below the screenshot of an example configuration in API Management using a custom domain called `api.custom-apim.domain.com` with a managed certificate and [Azure App Service Domain](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain). For more domain certificate options, see the Azure API Management [documentation](https://learn.microsoft.com/azure/api-management/configure-custom-domain?tabs=managed).

:::image type="content" source="media/expose-sap-odata-to-power-query/apim-custom-domain-configuration.png" alt-text="Screenshot that shows the custom domain configuration in Azure API Management":::

Complete the setup of your custom domain as per the domain requirements. For more information, see the [custom domain documentation](https://learn.microsoft.com/azure/api-management/configure-custom-domain?tabs=managed#set-a-custom-domain-name---portal). To prove domain name ownership and grant access to the certificate, add those DNS records to your Azure App Service Domain `custom-apim.domain.com` as below:

:::image type="content" source="media/expose-sap-odata-to-power-query/apim-custom-domain-setup.png" alt-text="Screenshot that shows custom domain mapping to Azure API Management domain":::

The respective Azure AD application registration for the Azure API Management tenant would look like below.

:::image type="content" source="media/expose-sap-odata-to-power-query/aad-app-reg-for-apim-configuration.png" alt-text="Screenshot that shows the app registration for Azure API Management in Azure Active Directory":::

> [!NOTE]
> If custom domain for Azure API Management isn't an option for you, you need to use a [custom Power Query Connector](https://docs.microsoft.com/power-query/startingtodevelopcustomconnectors) instead.

## Azure API Management policy design for Power Query

Use [this](https://github.com/Azure/api-management-policy-snippets/blob/master/examples/Handle%20Power%20Query%20access%20request%20to%20custom%20API.policy.xml) Azure API Management policy for your target OData API to support Power Query's authentication flow. See below a snippet from that policy highlighting the authentication mechanism. Find the used client ID for Power Query [here](https://learn.microsoft.com/power-query/connectorauthentication#supported-workflow).

```xml
<!-- if empty Bearer token supplied assume Power Query sign-in request as described [here:](https://docs.microsoft.com/power-query/connectorauthentication#supported-workflow) -->
<when condition="@(context.Request.Headers.GetValueOrDefault("Authorization","").Trim().Equals("Bearer"))">
    <return-response>
        <set-status code="401" reason="Unauthorized" />
        <set-header name="WWW-Authenticate" exists-action="override">
            <!-- Check the client ID for Power Query [here:](https://docs.microsoft.com/power-query/connectorauthentication#supported-workflow) -->
            <value>Bearer authorization_uri=https://login.microsoftonline.com/{{AADTenantId}}/oauth2/authorize?response_type=code%26client_id=a672d62c-fc7b-4e81-a576-e60dc46e951d</value>
        </set-header>
    </return-response>
</when>
```

In addition to the support of the **Organizational Account login flow**, the policy supports **OData URL response rewriting** because the target server replies with original URLs. See below a snippet from the mentioned policy:

```xml
<!-- URL rewrite in body only required for GET operations -->
<when condition="@(context.Request.Method == "GET")">
    <!-- ensure downstream API metadata matches Azure API Management caller domain in Power Query -->
    <find-and-replace from="@(context.Api.ServiceUrl.Host +":"+ context.Api.ServiceUrl.Port + context.Api.ServiceUrl.Path)" to="@(context.Request.OriginalUrl.Host + ":" + context.Request.OriginalUrl.Port + context.Api.Path)" />
</when>
```

> [!NOTE]
> For more information about secure SAP access from the Internet and SAP perimeter network design, see this [guide](https://docs.microsoft.com/azure/architecture/guide/sap/sap-internet-inbound-outbound#network-design). Regarding securing SAP APIs with Azure, see this [article](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/expose-sap-process-orchestration-on-azure).

## SAP OData authentication via Power Query on Excel Desktop

With the given configuration, the built-in authentication mechanism of Power Query becomes available to the exposed OData APIs. Add a new OData source to the Excel sheet via the Data ribbon (Get Data -\> From Other Sources -\> From OData Feed). Maintain your target service URL. Below example uses the SAP Gateway demo service **GWSAMPLE_BASIC**. Discover or activate it using SAP transaction `/IWFND/MAINT_SERVICE`. Finally add it to Azure API Management using the [official OData import guide](https://learn.microsoft.com/azure/api-management/sap-api).

:::image type="content" source="media/expose-sap-odata-to-power-query/odata-url-retrieve-from-apim.png" alt-text="Screenshot that shows how to discover the OData URL within Azure API Management":::

Retrieve the Base URL and insert in your target application. Below example shows the integration experience with Excel Desktop.

:::image type="content" source="media/expose-sap-odata-to-power-query/excel-odata-feed.png" alt-text="Screenshot that shows the OData configuration wizard in Excel Desktop":::

Switch the login method to **Organizational account** and click Sign in. Supply the Azure AD account that is mapped to the named SAP user on the SAP Gateway using SAP Principal Propagation. For more information about the configuration, see [this Microsoft tutorial](https://learn.microsoft.com/azure/active-directory/saas-apps/sap-netweaver-tutorial#configure-sap-netweaver-for-oauth). Learn more about SAP Principal Propagation from [this](https://blogs.sap.com/2021/08/12/.net-speaks-odata-too-how-to-implement-azure-app-service-with-sap-odata-gateway/) SAP community post and [this video series](https://github.com/MartinPankraz/SAP-MSTeams-Hero/blob/main/Towel-Bearer/103a-sap-principal-propagation-basics.md).

Continue to choose at which level the authentication settings should be applied by Power Query on Excel. Below example shows a setting that would apply to all OData services hosted on the target SAP system (not only to the sample service GWSAMPLE_BASIC).

> [!NOTE]
> The authorization scope setting on URL level in below screen is independent of the actual authorizations on the SAP backend. SAP Gateway remains the final validator of each request and associated authorizations of a mapped named SAP user.

:::image type="content" source="media/expose-sap-odata-to-power-query/excel-odata-login.png" alt-text="Screenshot that shows the login flow within Excel for the Organizational Account option":::

> [!IMPORTANT]
> The above guidance focusses on the process of obtaining a valid authentication token from Azure AD via Power Query. This token needs to be further processed for SAP Principal Propagation.

## Configure SAP Principal Propagation with Azure API Management

Use [this](https://github.com/Azure/api-management-policy-snippets/blob/master/examples/Request%20OAuth2%20access%20token%20from%20SAP%20using%20AAD%20JWT%20token.xml) second Azure API Management policy for SAP to complete the configuration for SAP Principal Propagation on the middle layer. For more information about the configuration of the SAP Gateway backend, see [this Microsoft tutorial](https://learn.microsoft.com/azure/active-directory/saas-apps/sap-netweaver-tutorial#configure-sap-netweaver-for-oauth). Learn more about SAP Principal Propagation from [this](https://blogs.sap.com/2021/08/12/.net-speaks-odata-too-how-to-implement-azure-app-service-with-sap-odata-gateway/) SAP community post and [this video series](https://github.com/MartinPankraz/SAP-MSTeams-Hero/blob/main/Towel-Bearer/103a-sap-principal-propagation-basics.md)

:::image type="content" source="media/expose-sap-odata-to-power-query/app-registration-dependencies.png" alt-text="Diagram that shows the Azure Active Directory app registrations involved in this article":::

The policy relies on an established SSO setup between Azure AD and SAP Gateway (use [SAP NetWeaver from the Azure AD gallery](https://learn.microsoft.com/azure/active-directory/saas-apps/sap-netweaver-tutorial#adding-sap-netweaver-from-the-gallery)). See below an example with the demo user Adele Vance. User mapping between Azure AD and the SAP system happens based on the user principal name (UPN) as the unique user identifier.

:::image type="content" source="media/expose-sap-odata-to-power-query/aad-user-config-for-sso.png" alt-text="Screenshot that shows the UPN of the demo user in Azure Active Directory":::

:::image type="content" source="media/expose-sap-odata-to-power-query/aad-enterprise-sap-registration-sso.png" alt-text="Screenshot that shows the SAML2 configuration for SAP Gateway with UPN claim":::

The UPN mapping is maintained on the SAP back end using transaction **SAML2**.

:::image type="content" source="media/expose-sap-odata-to-power-query/saml2-config.png" alt-text="Screenshot that shows the email mapping mode in SAP SAML2 transaction":::

According to this configuration **named SAP users** will be mapped to the respective Azure AD user. See below an example configuration from the SAP back end using transaction code SU01.

:::image type="content" source="media/expose-sap-odata-to-power-query/sap-su01-config.png" alt-text="Screenshot of named SAP user in transaction SU01 with mapped email address":::

 For more information about the required [SAP OAuth 2.0 Server with AS ABAP](https://help.sap.com/docs/SAP_NETWEAVER_750/e815bb97839a4d83be6c4fca48ee5777/0b899f00477b4034b83aa31764361852.html) configuration, see this [Microsoft](https://docs.microsoft.com/azure/active-directory/saas-apps/sap-netweaver-tutorial#configure-sap-netweaver-for-oauth) tutorial about SSO with SAP NetWeaver using OAuth.

Using the described Azure API Management policies **any** Power Query enabled Microsoft product may call SAP hosted OData services, while
honoring the SAP named user mapping.

:::image type="content" source="media/expose-sap-odata-to-power-query/excel-odata-import.png" alt-text="Screenshot that shows the OData response in Excel Desktop":::

## SAP OData access via other Power Query enabled applications and services

Above example shows the flow for Excel Desktop, but the approach is applicable to **any** Power Query enabled Microsoft product. For more information which products support Power Query, see [the Power Query documentation](https://docs.microsoft.com/power-query/power-query-what-is-power-query#where-can-you-use-power-query). Popular consumers are [Power BI](https://learn.microsoft.com/power-bi/connect-data/desktop-connect-odata), Excel for the web, [Azure Data Factory](https://learn.microsoft.com/azure/data-factory/control-flow-power-query-activity), [Azure Synapse Analytics Pipelines](https://learn.microsoft.com/azure/data-factory/control-flow-power-query-activity), [Power Automate](https://powerquery.microsoft.com/flow/) and [Dynamics 365](https://learn.microsoft.com/power-query/power-query-what-is-power-query#where-can-you-use-power-query).

## Next steps

[Learn from where you can use Power Query](https://docs.microsoft.com/power-query/power-query-what-is-power-query#where-can-you-use-power-query)

[Work with SAP OData APIs in Azure API Management](https://docs.microsoft.com/azure/api-management/sap-api)

[Configure Azure API Management for SAP APIs](https://docs.microsoft.com/azure/api-management/sap-api)

[Protect APIs with Application Gateway and API Management](https://docs.microsoft.com/azure/architecture/reference-architectures/apis/protect-apis)

[Integrate API Management in an internal virtual network with Application Gateway](https://docs.microsoft.com/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway)

[Understand Azure Application Gateway and Web Application Firewall for SAP](https://blogs.sap.com/2020/12/03/sap-on-azure-application-gateway-web-application-firewall-waf-v2-setup-for-internet-facing-sap-fiori-apps/)

[Understand implication of combining Azure Firewall and Azure Application Gateway](https://docs.microsoft.com/azure/architecture/example-scenario/gateway/firewall-application-gateway#application-gateway-before-firewall)

[Automate API deployments with APIOps](https://docs.microsoft.com/azure/architecture/example-scenario/devops/automated-api-deployments-apiops)
