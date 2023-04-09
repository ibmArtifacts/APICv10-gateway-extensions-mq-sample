# APICv10-gateway-extensions  
API Connect gateway extensions are ways to infuse datapower objects or services into the APIC framework to be referenced or used on APIC.  
For example, if there are MQ objects or custom logging targets that you would like to create within APIC, you cannot just create them in the APIC domain on the DataPower Gateway because they may be overriden by the APIC manager, or lost once pod is rebooted on OCP/k8s deployments.  
The best way to get desired DataPower objects or services into the APIC framework, is to create gateway extensions on APIC.  
More details about gateway extensions may be found in the IBM API Connect documentation [Extending the Gateway server behavior](https://www.ibm.com/docs/en/api-connect/10.0.5.x_lts?topic=environment-extending-gateway-server-behavior).  
  
This document will go over the example of creating an MQ object and log target within the APIC domain via gateway extension.  
  
If on OCP/k8s and you haven't enabled the DataPower webgui yet, you may use the following instructions to enable and get to the gateway webgui: [APICv10: Enabling DataPower WebGUI](https://github.com/ibmArtifacts/APICv10-Enabling-DataPower-WebGUI)  
  
#### High-Level Steps  
- Part 1: Get DataPower cfg code that is the DataPower object.  
- Part 2: Take the DataPower MQ object cfg code that is the DataPower object, zip it, upload to gateway. and disable/enable API Connect Gateway object on gateway to verify the MQ object is in APIC domain.  
- Part 3: Use/reference DataPower MQ object on APIC API.  
  
