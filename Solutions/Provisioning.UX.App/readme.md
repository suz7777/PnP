# PnP Provisioning - Self service site collection provisioning reference implementation#

### Summary ###

Even with good governance, your sites can proliferate and grow out of control. Sites are created as they are needed, but sites are rarely deleted. Many organization have search crawl burdened by unused site collections, difficulty with outdated and irrelevant results. This Solution shows a reference sample on how to build self-service site collection provisioning solution using the Office 365 Developer PnP provisioning engine, implements additional scenarios and samples to bring together a cohesive governance solution that can be used in your enterprise.

### Features ###
- User Interface to request site collections
- Capability to store Site Requests in either a SharePoint list or Azure Document DB 
- Request are processed asynchronously using the remote timer job pattern
- New site collection creation to Office 365 MT.
- New site collection creation in SharePoint on-premises builds including Office 365 Dedicated.
- Multiple host header site collection provisioning for on-premises builds
- Hybrid support for provisioning sites in SharePoint Online and SharePoint On-premises in one common solution
- Apply a configuration template to newly created sites using the PnP Provisioning Framework
- Enable External sharing for sites that are hosted in SharePoint Online MT
- Visual indicator if a Site is externally shared
- Site Classification.
- Site Policies and a visual indicator of the site policy that is applied
- Applying Composed Looks including, Alternate CSS, Logo, Background image, and fonts
- Provision site artifacts for example Site Columns, Content Types, List Definitions and Instances, Pages (either WebPart Pages or Wiki Pages)




### Applies to ###
-  Office 365 Multi-tenant (MT)
-  Office 365 Dedicated (D)
-  SharePoint 2013 on-premises 


### Solution ###
Solution | Author(s)
---------|----------
Provisioning.UX.App | Frank Marasco, Brian Michely and Steven Follis

*PnP remote provisioning Core Engine work done by Erwin van Hunen (Knowit AB), Paolo Pialorsi (PiaSys.com), Bert Jansen (Microsoft), Frank Marasco (Microsoft), Vesa Juvonen (Microsoft)*

### Version history ###
Version  | Date | Comments
---------| -----| --------
.1  | June 1, 2015 | Initial version

### Disclaimer ###
**THIS CODE IS PROVIDED *AS IS* WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING ANY IMPLIED WARRANTIES OF FITNESS FOR A PARTICULAR PURPOSE, MERCHANTABILITY, OR NON-INFRINGEMENT.**


**NOTICE THIS SOLUTION IS UNDER ACTIVE DEVELOPMENT**

### Prerequisites ###
- Azure subscription and existing DocumentDB Account which can be configured for to store your site requests (Optional) 
- April 2014 Cumulative Update or higher for the SharePoint 2013 on-premises builds
- For SharePoint on-premises see Vesa's to [blog](http://blogs.msdn.com/b/vesku/archive/2014/06/09/provisioning-site-collections-using-sp-app-model-in-on-premises-with-just-csom.aspx) to configure Provisioning of site collection for on-premises.
- If your are provisioning sites in a hybrid approach (on-premises and Office 365) it is required that your SharePoint on-premises farm is configured for Apps and must have a trust established to your Azure Active Directory


----------

# Conceptual design #

![](http://i.imgur.com/sQQ6JUX.png)


# Solution description #
Projects what are included in the solution . 

### Provisioning.UX.App###
SharePoint Add-In that is deployed to a Site Collection that will host the Application.

### Provisioning.Common ###
Reusable component that implements reusable logic for the Site Provisioning UX and Timer Job projects.

### Provisioning.Job ###
Remote Timer job project which maybe deployed to Azure or on-premises.  Will be responsible of the actual site collection creation and the logic on how to apply configuration/customization to newly created site.


### Provisioning.UX.AppWeb ###
This is the user interface (UX) for self service site collection creation application. This interface was built using primarily AngularJS and HTML. The intent was to create a modern interface that was easy to edit, and extend. Also provides Web Api interfaces to the UX.

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

You should use AppRegNew.aspx to register the SharePoint Add-in. 

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

The Provisioning.UX.AppWeb and Provisioning.Job each has its own configuration settings and you have to ensure that the settings are applied in both projects.

Configuration File | Description
-------------------|----------
appSettings.config | An alternate file to store application settings
provisioningSettings.config | An alternate file which is configured to control the implementation classes for the Provisioning Engine
Templates.config   | Used to display the available site templates to the Provisioning.UX.AppWeb and provides a mapping to PnP Provisioning Template in the Provisioning.Job

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


##### Templates.config #####
The Templates.config file resides in both the Provisioning.UX.AppWeb and Provisioning.Job projects under the /Resources/SiteTemplates. In the Provisioning.UX.AppWeb project it is used to display the available site templates and in the Provisioning provides mapping to PnP Provisioning Template. You will have to update this files in both projects to match your environment. 

Setting | Description
-------------------|----------
Title | The Title to display in the User Interface 
Description | The Description to display in the user interface
ImageUrl | The Image Url to display. 
HostPath | The Host path and managed path where to provision the site. Currently only Sites and Team managed paths are supported in SharePoint Online.
TenantAdminUrl | The Tenant Admin URL of the path. A tenant Admin Site must reside in each Web Application or Host Name Site Collection in SharePoint on-premises builds.
SharePointOnPremises | If the site is being provisioning is hosted on SharePoint on-premises builds or in SharePoint Online Dedicated Service.
RootTemplate | The base template to create. 
RootWebOnly | Reserved for future use that will be used to display available site templates in the Sub-Site Provisioning Page.
StorageMaximumLevel | The storage quota of the new site.(only applicable for site collections and SharePoint MT). 
StorageWarningLevel | The amount of storage usage on the new site that triggers a warning. (only applicable for site collections and SharePoint MT). 
UserCodeMaximumLevel | The maximum amount of machine resources that can be used by user code on the new site (only applicable for site collections and SharePoint MT). 
UserCodeWarningLevel | The amount of machine resources used by user code that triggers a warning.(only applicable for site collections and SharePoint MT). 
ProvisioningTemplateContainer | Not Used
ProvisioningTemplate | The Name of the PnP Provisioning Template to apply to the newly created site.
Enabled | Controls if the Template is available in the user interface.


	TemplateConfiguration Version='1.0' xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance'>
	  <Templates>
	    <!--  StorageMaximumLevel Minimum 110 
	          For on-prem & SPO-D Storage and UserCode are not used and will default to the Quota Templates configured in your farm
	    -->
	    <Template Title="SPO Team Site"
	              Description="Team sites should be used for team collaboration and offer basic document and project management features."
	              ImageUrl ="../images/template-icon.png"
	              HostPath="https://contoso.sharepoint.com/sites/"
	              TenantAdminUrl="https://contoso-admin.sharepoint.com"
	              SharePointOnPremises ="false"
	              RootTemplate="STS#0"
	              RootWebOnly="false"
	              StorageMaximumLevel="110"
	              StorageWarningLevel="0"
	              UserCodeMaximumLevel="100"
	              UserCodeWarningLevel="0"
	              ProvisioningTemplateContainer="Resources/SiteTemplates/ProvisioningTemplate/"
	              ProvisioningTemplate="TeamSiteTemplate.xml"
	              Enabled="true"/>
	    <Template Title="SharePoint On-Premises Site"
	              Description="I still have a SharePoint On-premises."
	              ImageUrl ="../images/template-icon.png"
	              HostPath="https://spsites.contoso.com/sites/"
	              TenantAdminUrl="https://spsites.contoso.com/sites/msotenantcontext"
	              SharePointOnPremises ="true"
	              RootTemplate="STS#0"
	              RootWebOnly="true"
	              StorageMaximumLevel="110"
	              StorageWarningLevel="0"
	              UserCodeMaximumLevel="110"
	              UserCodeWarningLevel="0"
	              ProvisioningTemplateContainer="Resources/SiteTemplates/ProvisioningTemplate/"
	              ProvisioningTemplate="TeamSiteTemplate.xml"
	              Enabled="true"/>
		<Template Title="SharePoint DONT SHOW ME
	              Description="Team sites should be used for team collaboration and offer basic document and project management features."
	              ImageUrl ="../images/template-icon.png"
	              HostPath="https://spsites.contoso.com/sites/"
	              TenantAdminUrl="https://spsites.contoso.com/sites/msotenantcontext"
	              SharePointOnPremises ="true"
	              RootTemplate="STS#0"
	              RootWebOnly="true"
	              StorageMaximumLevel="110"
	              StorageWarningLevel="0"
	              UserCodeMaximumLevel="110"
	              UserCodeWarningLevel="0"
	              ProvisioningTemplateContainer="Resources/SiteTemplates/ProvisioningTemplate/"
	              ProvisioningTemplate="TeamSiteTemplate.xml"
	              Enabled="false"/>

	  </Templates>
	</TemplateConfiguration>

The below images display the template selection that is available to the user.

![](http://i.imgur.com/Tq63tq9.png)


##### TeamSiteTemplate.xml #####

Defined in the Provisioning.Job in the Resources/SiteTemplates/ProvisioningTemplate/ folder is our PNP Provision Template. Within the Provisioning.UX.AppWeb project the base configuration provides a site classification user interface, banners for external sharing, and subsite override. Ensure that the urls are updated to match your environment. If you do not require this functionality all if you have to do is remove the custom actions in the template file. 

	<?xml version="1.0" encoding="utf-8" ?>
	<pnp:SharePointProvisioningTemplate ID="TEMPLATE1" Version="1.0"
	  xmlns:pnp="http://schemas.dev.office.com/PnP/2015/03/ProvisioningSchema"
	  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
	  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	  <pnp:Files>
	    <pnp:File Src="siteLogo.png" Folder="SiteAssets" Overwrite="true"/>
	    <pnp:File Src="custombg.jpg" Folder="~themecatalog/15" Overwrite="true" />
	    <pnp:File Src="custom.spcolor" Folder="~themecatalog/15" Overwrite="true" />
	    <pnp:File Src="custom.spfont" Folder="~themecatalog/15" Overwrite="true" />
	  </pnp:Files>
	  <pnp:CustomActions>
	    <pnp:SiteCustomActions>
	      <pnp:CustomAction Name="CA_SITE_CLASSIFICATION"
	               Description="Site Classification Indicator"
	               Location="ScriptLink"
	               Title="CA_SITE_CLASSIFICATION"
	               ScriptSrc=""
	               ScriptBlock="
	                    var lbiImageSource ='https://pnpsiteprov.azurewebsites.net/images/LBI.png';
	                    var mbiImageSource ='https://pnpsiteprov.azurewebsites.net/images/MBI.png';
	                    var hbiImageSource ='https://pnpsiteprov.azurewebsites.net/images/HBI.png';
	                    var headID = document.getElementsByTagName('head')[0]; 
	                    var siteClassifciationTag = document.createElement('script');
	                    siteClassifciationTag.type = 'text/javascript';
	                    siteClassifciationTag.src = 'https://pnpsiteprov.azurewebsites.net/scripts/siteClassification.js';
	                    headID.appendChild(siteClassifciationTag);"/>
	      <pnp:CustomAction Name="CA_SITE_EXTERNALSHARING"
	                Description="External Sharing Banner"
	                Location="ScriptLink"
	                Title="CA_SITE_EXTERNALSHARING"
	                ScriptSrc=""
	                ScriptBlock="
	                    var headID = document.getElementsByTagName('head')[0]; 
	                    var externalSharingTag = document.createElement('script');
	                    externalSharingTag.type = 'text/javascript';
	                    externalSharingTag.src = 'https://pnpsiteprov.azurewebsites.net/scripts/externalSharing.js';
	                    headID.appendChild(externalSharingTag);"/>
	      <pnp:CustomAction Name="CA_SITE_SUBSITE_OVERRIDE"
	                Description="Override new sub-site link"
	                Location="ScriptLink"
	                Title="CA_SITE_SUBSITE_OVERRIDE"
	                ScriptSrc=""
	                ScriptBlock="
	                    var SubSiteSettings_Web_Url = 'https://pnpsiteprov.azurewebsites.net/pages/subsite/newsbweb.aspx?SPHostUrl=';
	                    var headID = document.getElementsByTagName('head')[0]; 
	                    var subsiteScriptTag = document.createElement('script');
	                    subsiteScriptTag.type = 'text/javascript';
	                    subsiteScriptTag.src = 'https://pnpsiteprov.azurewebsites.net/scripts/SubSiteOverride.js';
	                    headID.appendChild(subsiteScriptTag);"/> 
	    </pnp:SiteCustomActions>
	    <pnp:WebCustomActions>
	      <pnp:CustomAction Name="CA_SITE_SETTINGS_SITECLASSIFICATION"
	                Description="Site Classification Application"
	                Group="SiteTasks"
	                Location="Microsoft.SharePoint.SiteSettings"
	                Title="Site Classification"
	                Sequence="1000"
	                Url="https://pnpsiteprov.azurewebsites.net/pages/SiteClassification/SiteEdit.aspx?SPHostUrl={0}"
	                Rights="31"/>
	      <pnp:CustomAction Name="CA_SITE_STDMENU_SITECLASSIFICATION"
	                Description="Site Classification Module"
	                Group="SiteActions"
	                Location="Microsoft.SharePoint.StandardMenu"
	                Title="Site Classification"
	                Sequence="1000"
	                Url="https://pnpsiteprov.azurewebsites.net/pages/SiteClassification/SiteEdit.aspx?SPHostUrl={0}"
	                Rights="31"/>
	    </pnp:WebCustomActions>
	  </pnp:CustomActions>
	  <pnp:ComposedLook Name="Contoso" Version="1"
	                ColorFile="~themecatalog/15/custom.spcolor"
	                FontFile="~themecatalog/15/custom.spfont"
	                BackgroundFile="~themecatalog/15/custombg.jpg"
	                MasterPage="~masterpagecatalog/seattle.master"
	                AlternateCSS=""
	                SiteLogo="~site/SiteAssets/siteLogo.png" />
	</pnp:SharePointProvisioningTemplate>

##### External Sharing Notification Banner #####

![](http://i.imgur.com/Kd65FNs.png)

##### Site Classification Custom Action  #####

![](http://i.imgur.com/INzEnv5.png)

##### Site Classification Edit Screen  #####

![](http://i.imgur.com/OPwyx9H.png)


#### Coming Updates ####
- We are currently working on a update to the provisioning interface which will be a single page app instead of a dialog.
- Template.config file will have the option to be centrally located.
- Video walk-through and demo of the solution.



