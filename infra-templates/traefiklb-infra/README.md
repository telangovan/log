# Rancher Cattle Catalog for Mule with Traefik Load Balancer


Containerized API Platform (CAP) - Load Balancer

Support for:
 * Traefik Load Balancer

Dependency: 
 * Rancher Server 1.1.4
 * http://bitbucket.org/dirosbit/apiplatform.git - Mule Runtime

Prerequisite:
 * Hosts with label: load.balancer=true

Configurations:
  Port on which Load Balancer has to listen.

  External LB configuration has been left outside the scope of this project. The following changes needs to be done in your infra:
    * DNS entry for *.cap.internal mapped to hosts that have load.balancer=true label
    * External Load Balancer rule to route calls with appropriate host name based on Group ID and Env 