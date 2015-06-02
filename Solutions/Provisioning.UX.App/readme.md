# PnP Provisioning - Self service site collection provisioning reference implementation#

### Summary ###

Even with good governance, your sites can proliferate and grow out of control. Sites are created as they are needed, but sites are rarely deleted. Many organization have search crawl burdened by unused site collections, difficulty with outdated and irrelevant results. This Solution shows a reference sample on how to build self-service site collection provisioning solution using the Office 365 Developer PnP provisioning engine, implements additional scenarios and samples to bring together a cohesive governance solution that can be used in your enterprise.

### Features ###
- User Interface to request site collections
- Capability to store Site Requests in either a SharePoint list or Azure Document DB 
- Request are processed asynchronously using the remote timer job pattern
- New site collection creation to Office 365 MT.
- New site collection creation in SharePoint on-premises builds including Office 365 Dedicated.
- Apply a configuration template to newly created sites using the PnP Provisioning Framework
- Enable External sharing for sites that are hosted in SharePoint Online MT
- Visual indicator if a Site is externally shared
- Site Classification.
- Site Policies and a visual indicator of the site policy that is applied
- Applying Composed Looks including, Alternate CSS, Logo, Background image, and fonts
- Provision site artifacts for example Site Columns, Content Types, List Definitions and Instances, Pages (either WebPart Pages or Wiki Pages)


**NOTICE THIS SOLUTION IS UNDER ACTIVE DEVELOPMENT**


### Applies to ###
-  Office 365 Multi-tenant (MT)
-  Office 365 Dedicated (D)
-  SharePoint 2013 on-premises


### Solution ###
Solution | Author(s)
---------|----------
Provisioning.UX.App | Frank Marasco, Brian Michely and Steven Follis

### Version history ###
Version  | Date | Comments
---------| -----| --------
.1  | June 1, 2015 | Initial version

### Disclaimer ###
**THIS CODE IS PROVIDED *AS IS* WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING ANY IMPLIED WARRANTIES OF FITNESS FOR A PARTICULAR PURPOSE, MERCHANTABILITY, OR NON-INFRINGEMENT.**


----------

# Conceptual design #
DOCUMENTATION IN PROGRESS

# Solution description #
Projects what are included in the solution . 

### Provisioning.UX.App###
SharePoint Add-In that is deployed to a Site Collection that will host the Application.

### Provisioning.Common ###
Reusable component that implements reusable logic for the Site Provisioning UX and Timer Job projects.

### Provisioning.Job ###
Remote Timer job project which maybe deployed to Azure or on-premises.  Will be responsible of the actual site collection creation and the logic on how to apply configuration/customization to newly created site.


### Provisioning.UX.AppWeb ###
This is the user interface (UX) for self service site collection creation application. This interface was built using primarily AngularJS and HTML. The intent was to create a modern interface that was easy to edit, and extend.

The interface is launched from default.aspx and the wizard itself is modal based and loads HTML views. These views make a wizard provisioning approach that collects data from the user and submits that data to the back-end provisioning engine. 

Landing Page:

![](http://i.imgur.com/TYiBokL.png)

Clicking the "Get Started" button above launches the Wizard:

![](http://i.imgur.com/Jcy7tEF.png)

#### Navigation ####
The wizard can be navigated either via left side navigation or arrow based navigation on the bottom right. The navigation and views are defined in the wizard.modal.html file. Note - next release will most likely load this from a configuration source, but for now, it's a simple modification to the html file to edit your navigation.

![](http://i.imgur.com/uYwJ0ac.png)

#### Services ####
There are some services exposed that can be used to get template and other data from the back-end, and a service for submitting that data. For PnP sample purposes, the the reference data for the sample meta-data fields gets loaded from .json files. There is a **BusinessMetadata factory** that loads the data from the json files and is invoked from the **wizard.modal.controller** script and the HTML fields bind to the model and the data is loaded via a repeater in most cases. This is only for sample purposes and for a real implementation this data may be list driven or from some other source and can be retrieved via other appropriate methods

![](http://i.imgur.com/9hkCeFf.png)

These services use the CSOM controller **provisioning.controller.cs** which uses **OfficeDevPnP.Core.WebAPI**.

#### People Picker ####
This solution also leverages the PnP JSOM version of the PeoplePicker. 

![](http://i.imgur.com/lmbNL2K.png)

#### Site Availability Checking ####
The site details view contains a field where the user specifies the url of their new site. The solution implements an angular directive that fires off and calls the sitequeryservice.js script which does the site availability check. If the site is available, the solution will set the field to validated, and if the site is not available, there will be a message displayed stating this.

#### Confirmation ####
Once user is done with the views in the wizard, they will be presented with a confirmation view and the chance to change their inputs. Once they click the checkmark icon, the site request object data will be submitted to the engine. 

----------

# Getting Started #

#### Site Policies ####
We need to define the site policies that will be available in all your sites collections. We are going to define the Site Policies in the content type hub and publish. In this example we are using SharePoint Online MT, but this same approach is available in SharePoint Online Dedicated as well as SharePoint on-premises. If your environment is hosted in SharePoint Online MT, your content type hub would be located at the following URL. https://[tenanatname]/sites/contentTypeHub. Navigate to Settings, then Site Policies under Site Collection Administration, and then finally create. 

See **Overview of site policies in SharePoint 2013** at http://technet.microsoft.com/en-US/library/jj219569(v=office.15).aspx for more information.

Create three site policies, HBI, MBI and then LBI.  Create an HBI Policy based on your requirements.

![](http://i.imgur.com/sKI5csC.png)

Repeat the above setup two more times for MBI and LBI. You should end up with the below:

![](http://i.imgur.com/lrw7nQD.png)

Once we have the policies created we are going to publish the Site Policies from the content type hub so they will be available to all the sites.

#### App Registration and Permissions ####

You should use AppRegNew.aspx to register your add-in for SharePoint. 

![](http://i.imgur.com/e6kIBzD.png)
	
This solution uses app only permissions so you will have to navigate to http://[Tenant]/_layouts/15/appinv.aspx and grant the application the following permissions.Use the Appinv.aspx page to lookup the add-in created in the previous step and then specify the permission XML. 


	<AppPermissionRequests AllowAppOnlyPolicy="true">
		    <AppPermissionRequest Scope="http://sharepoint/content/tenant" Right="FullControl" />
		    <AppPermissionRequest Scope="http://sharepoint/content/sitecollection/web" Right="FullControl" />
		    <AppPermissionRequest Scope="http://sharepoint/taxonomy" Right="Read" />
		    <AppPermissionRequest Scope="http://sharepoint/search" Right="QueryAsUserIgnoreAppPrincipal" />
		    <AppPermissionRequest Scope="http://sharepoint/content/sitecollection" Right="FullControl" />
	</AppPermissionRequests>
	
----------
#### Configuration Files ####

The Provisioning.UX.AppWeb and Provisioning.Job each has its own configuration settings.

Configuration File | Description
-------------------|----------
appSettings.config | An alternate file to store application settings
provisioningSettings.config | An alternate file which is configured to control the implementation classes for the Provisioning Engine

##### appSettings.config #####

	<appSettings>
		<!--USED TO SET THE SITE REQUEST TO Approve or New, IF A CUSTOM WORKFLOW IS USED SET TO false WILL SET THE SITE REQUEST STATUS as New-->
		<add key="AutoApproveSites" value="true" />
		<add key="ClientId" value="Insert Your Client ID" />
		<add key="ClientSecret" value="Insert Your Client Secret" />
		<!--THE SITE THAT HOSTS THE SITE PROVISIONING APPLICATION-->
		<add key="SPHost" value="The SharePoint Site that hosts the SharePoint Add-in />
		<add key="SupportTeamNotificationEmail" value="Your Support Email" />
		<!--THE TENANT ADMIN SITE FOR YOUR ENVIRONMENT-->
		<add key="TenantAdminUrl" value="Your Tenant Admin Url where the App-in is hosted" />
		<!--OVERRIDE FOR HOST NAME-->
		<add key="HostedAppHostNameOverride" value="Your Hosting FQDN of the Web" />
	</appSettings>


Setting | Description
-------------------|----------
AutoApproveSites | Used to set the site request to a Approved or New Status to support custom workflows to approve site requests. Set either to true or false
ClientId | Your Client ID 
ClientSecret | Your Client Secret
SPHost | The Site Url that hosts your SharePoint Add-in
SupportTeamNotificationEmail | Used to send notifications if there is an exception. This is reserved for future use in the Web Project
TenantAdminUrl | The Tenant Admin Site Url where the add-in is hosted
HostedAppHostNameOverride | The DNS name where the Web is hosted

##### provisioningSettings.config #####


Setting | Description
-------------------|----------
name | The name of the module to invoke. 
type | The class and assembly of the implementation
connectionString | The connection information that is used to connect to the source. 
container | The container where the artifacts are stored


Module Name | Description
RepositoryManager | Used to change the implementation class of the site request repository
MasterTemplateProvider | Used to display the available site templates and provides a mapping to PnP Provisioning Template. PnP provisioning XML uses community standardize schema available from own [repository](https://github.com/OfficeDev/PnP-Provisioning-Schema) under Office Dev in GitHub
ProvisioningProviders | PnP Core Provisioning Providers that contain the implementation on how to work with various source files.
ProvisioningConnectors | PnP Core Provisioning Connectors that contain the implementation on how to connect to custom PnP Providers. 

	<modulesSection>
	  <Modules>
	    <Module name="RepositoryManager" type="Provisioning.Common.Data.SiteRequests.Impl.SPSiteRequestManager, Provisioning.Common"
	            connectionString=""
	            container="" />
	    <!--IF RUNNING IN AZURE ADD [WEBROOT_PATH]/Resources/SiteTemplates/" TO CONNECTIONSTRING-->
	    <Module name="MasterTemplateProvider"
	            type="Provisioning.Common.Data.Templates.Impl.XMLSiteTemplateManager, Provisioning.Common"
	            connectionString="Resources/SiteTemplates/"
	            container="" />
	    <!--USED TO RETURN THE XML PROVIDERS-->
	    <!--PROVISIONING & PROVIDERS-->
	    <Module name="ProvisioningProviders"
	            type="OfficeDevPnP.Core.Framework.Provisioning.Providers.Xml.XMLFileSystemTemplateProvider, OfficeDevPnP.Core"
	            connectionString="Resources/SiteTemplates/ProvisioningTemplates"
	            container="" />
	    <Module name="ProvisioningConnectors"
	            type="OfficeDevPnP.Core.Framework.Provisioning.Connectors.FileSystemConnector, OfficeDevPnP.Core"
	            connectionString="Resources/SiteTemplates/ProvisioningTemplates"
	            container="" />
	    <!--AZURE CONNECTOR USED FOR STORING ASSESTS IN A BLOB-->
	    <!--<Module name="ProvisioningConnectors"
	              type="OfficeDevPnP.Core.Framework.Provisioning.Connectors.AzureStorageConnector, OfficeDevPnP.Core"
	              connectionString=""
	              container="assests\Resources\SiteTemplates\ProvisioningTemplates"/>
	        <Module name="XMLTemplateProviders"
	            type="OfficeDevPnP.Core.Framework.Provisioning.Providers.Xml.XMLAzureStorageTemplateProvider, OfficeDevPnP.Core"
	            connectionString=""
	            container="assests\Resources\SiteTemplates\ProvisioningTemplates"/>-->
	  </Modules>
	</modulesSection>

Note. The out of box configuration is configured to use a SharePoint List as the site request repository.The Site Request List is created at run time the first time a user tries to save a site request in the UX.

![](http://i.imgur.com/KQ4JvAb.png)


The following example configuration file shows how you can use the Azure Document DB to store the Site Requests. This gives us the capability to customer our Site Request Domain Model in a schema-free with native JSON support. 

	<modulesSection>
	    <Modules>
	      <Module name="RepositoryManager" type="Provisioning.Common.Data.SiteRequests.Impl.AzureDocDbRequestManager, Provisioning.Common"
	               connectionString="AccountEndpoint=https://yourazure.documents.azure.com:443/;AccountKey=frankwashere==;"
	               container="SiteRequests" />
	      <!--IF RUNNING IN AZURE ADD [WEBROOT_PATH]/Resources/SiteTemplates/" TO CONNECTIONSTRING-->
	      <Module name="MasterTemplateProvider" 
	              type="Provisioning.Common.Data.Templates.Impl.XMLSiteTemplateManager, Provisioning.Common" 
	              connectionString="Resources/SiteTemplates/" 
	              container="" />
	      <!--USED TO RETURN THE XML PROVIDERS-->
	      <!--PROVISIONING & PROVIDER FOR RUNNING IN ONPREM-->
	      <Module name="ProvisioningProviders" 
	              type="OfficeDevPnP.Core.Framework.Provisioning.Providers.Xml.XMLFileSystemTemplateProvider, OfficeDevPnP.Core" 
	              connectionString="Resources/SiteTemplates/ProvisioningTemplates" 
	              container="" />
	      <Module name="ProvisioningConnectors" 
	              type="OfficeDevPnP.Core.Framework.Provisioning.Connectors.FileSystemConnector, OfficeDevPnP.Core" 
	              connectionString="Resources/SiteTemplates/ProvisioningTemplates" 
	              container="" />
	      <!--AZURE CONNECTOR USED FOR STORING ASSESTS IN A BLOB-->
	      <!--<Module name="ProvisioningConnectors"
	              type="OfficeDevPnP.Core.Framework.Provisioning.Connectors.AzureStorageConnector, OfficeDevPnP.Core"
	              connectionString=""
	              container="assests\Resources\SiteTemplates\ProvisioningTemplates"/>
	        <Module name="XMLTemplateProviders"
	            type="OfficeDevPnP.Core.Framework.Provisioning.Providers.Xml.XMLAzureStorageTemplateProvider, OfficeDevPnP.Core"
	            connectionString=""
	            container="assests\Resources\SiteTemplates\ProvisioningTemplates"/>-->
	    </Modules>
	</modulesSection>


Notice  container for the RepositoryManager. This is the Azure Document Database. The implementation creates the database and collection at run time. 

![](http://i.imgur.com/U402PK5.png)

In order to use Azure Document DB you must first create a new DocumentDB Account in the [Microsoft Azure Preview Portal](https://portal.azure.com/).

![](http://i.imgur.com/SLb3KAm.png)

Copy the Primary or Secondary Connection string and update the connectionString in your RepositoryManager connectionString

![](http://i.imgur.com/uhStvV6.png)


#### Coming Updates ####
We are currently working an update to this interface which uses an angular schema form approach and will allow you to define a schema in JSON and the fields you wish to use. You can then use one line of html to load your form/view which will then be schema driven and defined there and not in your views.


