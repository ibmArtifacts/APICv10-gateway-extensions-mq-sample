# APICv10 Gateway Extensions: MQ Sample  

API Connect gateway extensions are ways to infuse datapower objects or services into the APIC framework to be referenced or used on APIC.  
For example, if there are MQ objects or custom logging targets that you would like to create within APIC, you cannot just create them in the APIC domain on the DataPower Gateway because they may be overriden by the APIC manager, or lost once pod is rebooted on OCP/k8s deployments.  
The best way to get desired DataPower objects or services into the APIC framework, is to create gateway extensions on APIC.  
More details about gateway extensions may be found in the IBM API Connect documentation [Extending the Gateway server behavior](https://www.ibm.com/docs/en/api-connect/10.0.5.x_lts?topic=environment-extending-gateway-server-behavior).  
  
This document will go over the example of creating an MQ object within the APIC domain via gateway extension.  
  
If on OCP/k8s and you haven't enabled the DataPower webgui yet, you may use the following instructions to enable and get to the gateway webgui: [APICv10: Enabling DataPower WebGUI](https://github.com/ibmArtifacts/APICv10-Enabling-DataPower-WebGUI)  
  
#### High-Level Steps  
- Part 1 Create the DataPower object and export.  
- Part 2: Create the manifest.json file to package together as the Gateway Extension zip & upload to APIC.  
- Part 3 Use/reference the DataPower object on APIC API.  
  
### Create the DataPower object and export  
When a DataPower object or service is created, applied, and when the "save config" is clicked, it is written to the .cfg file in the File Management > config directory.  
The following is an example of an MQ object and how to export the DataPower object.  
![image](https://user-images.githubusercontent.com/66093865/236657111-8f701c37-82cd-4723-9cb4-09394e0c0fc4.png)  
  
NOTE: If you have certificates that are required with the service, you may:
1. unzip the export, 
2. update the export.xml file with the following <file> stanza into the <files> section, 
```
	<files>
		<file name="cert:///name_of_cert_here.pem" src="cert/name_of_cert_here.pem" location="cert" />
	</files>
```  
3. and add a cert directory with the certificate in the directory to be uploaded during the import:  
![image](https://user-images.githubusercontent.com/66093865/236658062-8709a8e2-c96a-4f15-8f6e-8f15ed9603eb.png)  
    
NOTE: The datapower export file have been renamed to dp-export-mq-with-tls.zip.  

### Create the manifest.json file to package together as the Gateway Extension zip & upload to APIC   
1. Create the manifest.json file as sampled below:  
```  
{
    "extension": {
      "properties": {
        "deploy-policy-emulator": false
      },
      "files": [
        {
          "filename":"dp-export-mq-with-tls.zip",
          "deploy": "immediate",
          "type": "dp-import"
        }
    ]
  }   
}
```
2. Zip up the manifest.json and datapower export.  
![image](https://user-images.githubusercontent.com/66093865/236659207-1294dd1f-1348-4c4c-bce5-b1c9ba30e6a6.png)  
  
3. Navigate to the Cloud Manager > Topology section, locate the Gateway line item, select the elipsis (![image](https://user-images.githubusercontent.com/66093865/236659714-3b0dd9df-fd01-41b4-8266-90857a2405e7.png)), and select Configure gateway extension.  
Add the gateway extension.   
![image](https://user-images.githubusercontent.com/66093865/236659775-b3c0a601-c6a5-4192-bfc5-2b349848dc03.png)  
  
Once uploaded, you will see the gateway-extension set:  
![image](https://user-images.githubusercontent.com/66093865/236660097-71073a9a-ea17-4b12-8781-5e910a58ddcb.png)  
  
Verify that the MQ object is set by logging into the gateways apiconnect domain:  
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

With correct MQ inputs, you may test the API and check the message PUT into the queue.  
![image](https://user-images.githubusercontent.com/66093865/231655989-c2809cdf-f725-4a8d-8def-3381082e8ced.png)  


