# Rancher Cattle Catalog for Mule with Salt Master


Containerized API Platform (CAP) - Application Deployment Manager

Enables configuration management of an API Management Platform in a Cloud Computing paradigm within an Enterprise.

Support for:
 * Salt Master

Dependency:
 * Rancher Server 1.1.4
 
Prerequisite:
 * Hosts with label: config.manager=true

Configurations:
  Drivers for this Stack is the location from where Applications will be deployed. 
  
Key Point to Note:
 * Application Folder structure has to match the convention used in the Mule Runtime
 * First level folder has to be the Group Name - BU or Project ID
 * If Deployment Style is by Folder, second level Folder with only these names are supported - dev/stage/prod
 * If Deployment Style is by Tags, Apps can be placed under Group folder, but has to have dev/stage/prod in the Name
 * Modification of Applications is not supported. Delete and Create for Modifications of Applications