# APICv10-gateway-extensions  
API Connect gateway extensions are ways to infuse datapower objects or services into the APIC framework to be referenced or used on APIC.  
For example, if there are MQ objects or custom logging targets that you would like to create within APIC, you cannot just create them in the APIC domain on the DataPower Gateway because they may be overriden by the APIC manager, or lost once pod is rebooted on OCP/k8s deployments.  
The best way to get desired DataPower objects or services into the APIC framework, is to create gateway extensions on APIC.  
More details about gateway extensions may be found in the IBM API Connect documentation [Extending the Gateway server behavior](https://www.ibm.com/docs/en/api-connect/10.0.5.x_lts?topic=environment-extending-gateway-server-behavior).  
  
This document will go over the example of creating an MQ object and log target within the APIC domain via gateway extension.  
  
If on OCP/k8s and you haven't enabled the DataPower webgui yet, you may use the following instructions to enable and get to the gateway webgui: [APICv10: Enabling DataPower WebGUI](https://github.com/ibmArtifacts/APICv10-Enabling-DataPower-WebGUI)  
  
#### High-Level Steps  
- Part 1 Gather/create DataPower cfg code: Get DataPower cfg code that is the DataPower object.  
- Part 2: Create Gateway Extension & upload to APIC: Take the DataPower cfg code that is the DataPower object, place it in a notepad, zip it, upload to gateway. and disable/enable API Connect Gateway object on gateway to verify the MQ object is in APIC domain.  
- Part 3 Use/reference the DataPower object on APIC API.  
  
### Gather/Create DataPower CFG Code  
When a DataPower object or service is created, applied, and when the "save config" is clicked, it is written to the .cfg file in the File Management > config directory.  
The following is an example of an MQ object and its associated DataPower cfg code. This is what we need if we want to embed this service into the APIC framework.  
![image](https://user-images.githubusercontent.com/66093865/231477460-e90757e9-5378-4541-87c7-093dd901291d.png)  


### Create Gateway Extension & Upload to APIC   
To create the Gateway Extension take the desire cfg code and create a new file on your local system titled something like "apic-gw-extension.cfg".  
Input the desired DataPower cfg code into the "apic-gw-extension.cfg" like something similar to below.   
```
top; configure terminal;

password-alias "mqm_pw"
password INPUT_PASSWORD_HERE
exit

%if% available "mq-qm"

mq-qm "INPUT_QUEUE_MANAGER_OBJECT_NAME_HERE"
  hostname INPUT_MQ_CLUSTER_IP_HERE(WITH_PORT)
  queue-manager "INPUT_QUEUE_MANAGER_NAME_HERE"
  ccsid 819
  channel-name "SYSTEM.DEF.SVRCONN"
  mqcsp-userid "mqm"
  mqcsp-password-alias mqm_pw
  heartbeat 300
  maximum-message-size 1048576
  cache-timeout 60
  no automatic-backout 
  total-connection-limit 250
  initial-connections 1
  sharing-conversations 0
  no share-single-conversation 
  no permit-insecure-servers 
  no permit-ssl-v3 
  ssl-cipher none
  no auto-recovery 
  convert 
  auto-retry 
  retry-interval 5
  retry-attempts 3
  long-retry-interval 1800
  reporting-interval 1
  alternate-user 
  polling-tolerance 10
  xml-manager default
  ssl-client-type proxy
exit

%endif%
```  

The diagram below shows the cfg code of the DataPower MQ object on its own in the apic-gw-extension.cfg, being zipped up, and uploaded to the gateway in the Cloud Manager > Topology > Gateway.  
![image](https://user-images.githubusercontent.com/66093865/231494587-4f44cfc5-9af5-441a-bef8-78a3c44d3554.png)  
  
#### Force Management configuration push to gateway
Force the Management subsystem to push the configuration to the gateway by logging into the Gateway, selecting the apiconnect domain, and navigating into the API Connect Gateway Service. Disable and then reenable the Administrative State.  
![image](https://user-images.githubusercontent.com/66093865/231548666-5203808e-39c8-4466-944e-78d36e8ea136.png)  
  
Then verify that the MQ object is set.  
![image](https://user-images.githubusercontent.com/66093865/231548710-731834fa-d0bc-4828-b952-f662d681f74f.png)  
  

### Use/reference the DataPower object on APIC API  
You may now create APIs that uses the DataPower MQ object.  
This section will help you build an API to use that DataPower MQ object.  

1. Create an API with no security on it for testing purposes. Details about creating new REST APIs may be found in the API documentation: [Createing a new REST OpenAPI definition](https://www.ibm.com/docs/en/api-connect/10.0.5.x_lts?topic=definition-creating-new-rest-openapi), or you may use the sample API created [here](https://github.com/ibmArtifacts/APICv10-gateway-extensions/blob/main/sample-api-using-mq-object.yaml).  
  
2. In the sample API, ensure that the Parse policy and the GatewayScript are position like so:  
![image](https://user-images.githubusercontent.com/66093865/231633462-645f77d8-5701-4b44-afdd-79063596984b.png)  
  
3. The following code goes into the Gateway and that is what will use the DataPower MQ object to put messages on the queue.  
```  
var urlopen  = require('urlopen');

var inpMsg = context.get('request.body');
console.log('****inpMsg: ' + inpMsg);

var dpmqurl = { target: 'dpmq://INPUT_QUEUE_MANAGER_OBJECT_NAME_HERE/?',
    requestQueue: 'INPUT_REQUEST_QUEUE_HERE',
    transactional: false,
    sync: false,
    timeOut: 10000,
    data: inpMsg };
console.log('****dpmqurl: ' + JSON.stringify(dpmqurl));

urlopen.open (dpmqurl, function (error, response) {
    // handle the error when connecting to MQ
    if (error) {
        var msg = error + ' errorCode= ' + error.errorCode;
        console.error("MQ error is %s", msg);
        throw error;
    }
    
    var vResponse = response.headers.MQMD
	context.message.body.write(vResponse);
	context.message.header.set('Content-Type', "application/xml");
});
```  


