# Rancher Cattle Catalog for Mule


Containerized API Platform (CAP) - Simplified Log Manager

Enables full realization of the potential of an API Management Platform in a Cloud Computing paradigm within an Enterprise.

Support for:
 * Zookeeper
 * Kafka
 * Logspout Agent
 * GrayLog Log Server

Dependency: 
 * Rancher Server 1.1.4

Prerequisite:
 * Launch one or more instances of Containerized API Platform (CAP) - Load Balancer
 * Launch one or more instances of Containerized API Platform (CAP) - Application Deployment Manager
 * Launch one or more instances of Containerized API Platform (CAP) - Mule Runtime

Configurations:
- Goto System -> Input and create two Input Stream with below configuration:
  * fetch_min_bytes:    5
    fetch_wait_max:     100
    override_source:    CAP Runtime
    threads:            2
    throttling_allowed: false
    topic_filter:       ^capruntimelogs$
    zookeeper:          zookeeper.<<stackname>>:2181
    
    Within the Input, click Manage Extractor and Import below Extractor:
	{
	  "extractors": [
		{
		  "title": "CAP Runtime Log Extractor",
		  "extractor_type": "grok",
		  "converters": [],
		  "order": 0,
		  "cursor_strategy": "cut",
		  "source_field": "message",
		  "target_field": "",
		  "extractor_config": {
			"grok_pattern": "%{CAPLOGFORMAT}",
			"named_captures_only": false
		  },
		  "condition_type": "regex",
		  "condition_value": "service=(muleminion |mule )"
		}
	  ],
	  "version": "2.1.0-SNAPSHOT"
	}
	
  * fetch_min_bytes:    5
    fetch_wait_max:     100
    override_source:    CAP Config Manager
    threads:            2
    throttling_allowed: false
    topic_filter:       ^capcfgmgrlogs$
    zookeeper:          zookeeper.<<stackname>>:2181
    
    Similar to above, create an Extractor for this Input as well:
    
    - Goto     
	{
	  "extractors": [
		{
		  "title": "CAP Config Mgr Log Extractor",
		  "extractor_type": "grok",
		  "converters": [],
		  "order": 0,
		  "cursor_strategy": "cut",
		  "source_field": "message",
		  "target_field": "",
		  "extractor_config": {
			"grok_pattern": "%{CAPLOGFORMAT}",
			"named_captures_only": false
		  },
		  "condition_type": "regex",
		  "condition_value": "service=(grandmaster |gmstrminion )"
		}
	  ],
	  "version": "2.1.0-SNAPSHOT"
	}
- Goto System -> Grok Patterns and Create GROK Pattern:
    Name = CAPLOGFORMAT
    Pattern = time="%{GREEDYDATA:recdTimeStamp}" container_name="/%{GREEDYDATA:containerName}" source="%{GREEDYDATA:stdOutOrstdErr}" data="time=%{GREEDYDATA:logTimeStamp} level=%{GREEDYDATA:logLevel} service=%{GREEDYDATA:service} grpid=%{GREEDYDATA:buId} package=%{GREEDYDATA:packageName} function=%{GREEDYDATA:functionName}: message=%{GREEDYDATA:logMessage}"

