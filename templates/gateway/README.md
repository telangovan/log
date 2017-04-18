# Rancher Cattle Catalog for Mule


Containerized API Platform (CAP) - Mule Runtime 

Enables full realization of the potential of an API Management Platform in a Cloud Computing paradigm within an Enterprise.

Support for:
 * Salt Minion - Expects the Salt Master Stack to be running in the Environment
 * Mulesoft API Gateway

Dependency: 
 * Rancher Server 1.1.4
 * http://bitbucket.org/dirosbit/apiplatform.git - Load Balancer and Application Deployment Manager Stacks

Prerequisite:
 * Launch one or more instances of Containerized API Platform (CAP) - Load Balancer
 * Launch one or more instances of Containerized API Platform (CAP) - Application Deployment Manager

Configurations:
  Service Group Identifier and Environment drives the behavior of this Stack
  Example: 
      Input: Group Identifier = crmproject, Environment = dev
      Listen Path: api.crmproject.dev.cap.internal:8081/
      
  The Apps listen path will be configured in the Load Balancer. External LB configuration has been left outside the scope of this project. The following changes needs to be done in your infra:
    * DNS entry for *.cap.internal mapped to hosts that have load.balancer=true label
    * External Load Balancer rule to route calls with appropriate host name based on Group ID and Env 
  
  Note: If this container should not be spun on a particular host, set label runtime.exclude=true