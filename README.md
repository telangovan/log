# Containerized Anypoint Runtime Platform for Mule with Load Balancer and Package Deployment

This repository contains the images used by Containerized Anypoint Platform (CAP) related images. The catalog for spinning these up within Rancher are provided in the adjunct repository dirosbit/captemplates. They support spinning multiple runtimes within a single host. 

The rationale behind this project is to create a light-weight but complete instance of Mule 3.8 CE Runtime that can host multiple Mule applications within each instance.

## Considerations

Middleware Infrastructure does not naturally lend itself to a containerized-microservices paradigm. Almost all the text book definitions gets broken - service is small, but the infra for running it is large; isolation is important, but cost of isolation is prohibitive both from license and operations perspective, nice to start for a small BU, but when an Enterprise puts its weight behind it things can get very unwieldy. 

In a large enterprise where multiple Business Units need to deploy their artifacts (Proxies, ESB Flows etc.) cannot get a container model without creating operations pressure on Central IT. Especially when the middleware platform has an image in the tune of 1 GB after deployment. Imagine a BU with 10 simple proxies requiring high availability configuration - even conservatively this translates to 20 GB of resources. The other option is, resort to building these with lightweight architectures a.k.a microservices from ground-up. This is realistically not an option considering the advantages of a API Management or Partner Management infrastructure that comes from the vendor, like Mulesoft Inc. 

Self-service with an element of central governance seemed like the way forward and some bold decisions to break away from immutable infrastructure. Note: This solution mutates the infra, by deploying apps into a running container. If this is anti-theses to your idea of containerization, this is not the project for you.

The approach adopted for this project was to build a platform that leverages Mulesoft (can be replaced with any other partner of choice) and augment it with Load Balancer, Configuration Management and Container Orchestration capabilities, thereby letting a Central IT team provide an isolated infrastructure, but at the same time optimising resource allocation and operations overheads. 

## Philosophy 

Based on the experience gathered at one of the largest Enterprise API Management initiatives, and the considerations we grappled with balancing Business Units demand for isolation and Operations team demand for manageable infrastructure complexity, this was the final approach we decided upon:

- Business Unit level isolation of Mule Runtime - Each BU should get complete control over their runtime instance, but the applications themselves should be isolated from other BU runtimes;
- Optimizing Mule Container utilization - Business Unit's should be able to determine how many applications they want to pack within a single container. Though this is an anti-pattern to load multiple applications within one container, due to large size of each container instance, this provides the optimal balance of isolation and resource utilization
- Simplified Deployment - Deployment should be easy for tenants without remembering the number of instances created and running


# Requirements

- Rancher Server >= v0.63.0
	- Vagrant 1.8.4
	- Virtual Box 5.0.20

# How to use

- Create a Rancher Cattle Environment. For creating this environment, follow the instructions in this Project. This application is distributed via a Private Catalog pulled from this repository. 
- Enable the Catalog under Admin => Settings => Add Catalog in the Rancher UI and add the following:
  - Name: Mule Catalog
  - URL: http://bitbucket.org/dirosbit/captemplates.git
- Add a few Hosts
- Label some Hosts with load.balancer=true, config.manager=true
- Mark some hosts with runtime.exclude=true - Typically you might want to do this for hosts that run Application Deployment Manager stack
- Launch Load Balancer. On all hosts that have load.balancer=true an instance of the container will be launched
- Launch Application Deployment Manager and provide a Host Location from where Applications will be deployed. On one host that has the label config.manager=true an instance will be launched. 
- Launch Mule Runtime Stack with the following values:

```
   - Name: <Unique Name for the Stack>
   - Description: Gateway Group assigned to Project One
   - Group ID - projectone
   - Listen At - /prjone/
   - Deployment Type - Folder
   - Environment - Dev   
```
- Launch another instance of Mule Runtime with the following values:

```
   - Name: <Another Unique Name for the Stack>
   - Description: Gateway Group assigned to BU One
   - Group ID - crmbu
   - Listen At - /sales/
   - Deployment Type - Tags
   - Environment - Stage
```
- In the Host Location mentioned in the Deployment Manager, Create two folders "projectone", "crmbu" - without quotes
- Copy the Sample App given in samples folder to these two locations. Note the sample application has been named to support Tag based deployment as well. 
- Deployment will happen every 60 seconds - currently not exposed as a Configuration
- From a command prompt, call the following URL:
```
curl -H Host:api.projectone.dev.cap.internal http://{host_with_lb_ip}:18080/api/get?a=123
curl -H Host:api.crmbu.stage.cap.internal http://{host_with_lb_ip}:18080/api/headers
```
{host_with_lb_ip} will be the IP address of any Host running Load Balancer stack. Note the proxies are pointing the public site http://httpbin.org. All supported Web Resources can be tested through this deployment.

# Explanation 

- Behind the scene when we launch a Mule instance the following is happening:
  - A Mule container is launched with the minion naming convention mule-<<groupid>>-<<env>>-<<Machine ID>>
  - Salt Master is configured to auto accept the Key and react to certain events - check etc/salt/master config
  - Salt Master pushes the configuration to the Mule container through the Minion - check srv/reactor/setupmule.sls
  - Salt Master pushes the Shared Domain project by executing a State file - check salt/mule/copyfiles-domain.sls
  - Salt Master pushes the Application designated for that BU by executing the appropriate State file - check salt/mule/copyfile-apps.sls

# Future Extensions

- This Release Candidate only focuses on running a healthy container environment and managing deployments
- Future Release will focus on JMX based monitoring of Mule

# Known Issues

The approach taken might not be acceptable to some usage scenarios because of the following reason:

- Salt Master Key Acceptance - Current approach auto-accepts the keys before controlling the minions. This might not be acceptance to most. This is done to ensure the Mule container is prepared with the Shared Domain and the Apps for the BU, without any manual involvement. If desired, turn Auto Accept to false on the Salt Master Config. Nothing else is impacted.
- Salt Minion Key Reissue - When Mule container restarts after a failed start (memory issue preventing the container to be healthy), new keys are issued. Frequent deletion of old keys is needed to keep the Master clean.  Executing 
```
sudo salt-run manage.down removeKeys
``` 
will do the trick, but this doesnt seem clean. We will be adding this to the Reactor soon.

- Distributing Mule Applications - Since the number and the actual server Mule instances are running for a particular BU cannot be determined without inquiring the Rancher Metadata, the approach adopted uses a naming convention of Mule Minion to push the Applications to the appropriate instances. The convention assumes the Listen Path captured will be used to determine the Mule stack to be routed to. There is no other mean of determining where the calls have to be routed
- Mule Application Repackaging - A downloaded proxy from Anypoint Platform uses a Default domain. Using this package as-is, prevents any more application from being deployed on the same port. To prevent this the Application has to be modified to use a different Domain - in this project "shared-domain". Making the necessary changes to get the package in order needs a better CI-CD process. The sample applications part of this repo already has this fixed. If you want to try with more Proxies, remember to make the following changes:
  - Remove HTTP Listener Config
  - Remove <api-platform-gw:api apiName="![p['api.name']]" version="![p['api.version']]" flowRef="proxy"> - This forces a binding with the Anypoint Platform
    </api-platform-gw:api>
  - Change HTTP Listener to Shared Listener
  - Change Default Domain to Shared Domain
