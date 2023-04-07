# APICv10-gateway-extensions  
API Connect gateway extensions are ways to infuse datapower objects or services into the APIC framework to be referenced or used on APIC.  
For example, if there are MQ objects or custom logging targets that you would like to create within APIC, you cannot just create them in the APIC domain on the DataPower Gateway because they may be overriden by the APIC manager, or lost once pod is rebooted on OCP/k8s deployments.  
The best way to get desired DataPower objects or services into the APIC framework, is to create gateway extensions on APIC.  
More details about gateway extensions may be found in the IBM API Connect documentation [Extending the Gateway server behavior](https://www.ibm.com/docs/en/api-connect/10.0.5.x_lts?topic=environment-extending-gateway-server-behavior).  
This document will go over the example of creating an MQ object within the APIC domain via gateway extension
