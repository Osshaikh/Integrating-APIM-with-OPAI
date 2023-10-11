# Integrating-APIM-with-OPAI
Integrating Azure OpenAI Service with APIM for multiple use cases

There are few explcitly reason where customer would rather use APIM for exposng OpenAI API instead of traditonal layer 7 LB
* Abstratcing Authentication/Authorization layer using APIM managed identity , remove need of using OpenAI Keys
* Advanced Throtlling or rate limiting based on multiple factors such as IP,Product,User.Identifier, JWT-Tokenclaims etc.
* load balancing across multiple backend openai endpoints
* Monitor Az OPAI JWT token consumption to keep track on pricing

* All APIM policies used in this demo will be shared in last section of page

To Get started these solution you would need to have APIM instance and atleast two Az OPAI service deployed

* Az OPAI provide two way of authentication i.e. API Keys & Azure AD Authentication, in this demo we are going to use latter in integration with APIM
* Enable managed identity on APIM and assign RBAC 'Cognitive Service User' on the respective OpenAI services
![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/8e68fdcb-7870-4183-9524-cd1460a83b42)

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/252ae779-b96a-4900-baff-d5401c787bf5)


* Az OPAI porvides Sample REST API refernces for all of its models, which can be modified based on your param and imported into APIM. link https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#chat-completions
![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/e8086cb3-3a2b-42b4-a609-56ccd448f9bd)

* In this demo we are using Chatgpt3.5 model with swagger spec, in this spec files we need to replace default values under server secton with own openAI endpoint fqdn, refer below screenshot
![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/27a2b6db-7a22-4786-9360-71c3285a69fc)

* open Azure API Management resource Under APIs, + Add API, search for OpenAPI. Upload your OpenAPI spec json that you modified in last step
![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/d8bcf570-2ed1-4a1a-a662-f2574ddc2470)

  * Az OPAI support Azure AD RABC Authentication, Hence we are using Authentication via APIM manage identity, in workflow when API request received by APIM it will used its own identity principal onbehalf of client & send request to azure AD to acquire Bearer Token , Then append the bearer token in existng client request and forward to OpenAI models
 
  * We are using APIM Inbound policies to use manage identity for authentication and get response from OpenAI models

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/f96a9e03-fb3f-4e16-bcab-b6da418951e6)

* Once Inbound policies for managed identity is confgured , Simulate test within APIM with traces enabled for verbose logging & you would notice managed identity being used to get bearer token

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/8e897003-080a-4dde-b5aa-7e60831c6ab0)

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/0f467a71-d680-4cb8-8d68-12b0ea248731)

* Another common use case by customer is to use APIM to load balance across multiple OpenAI models to avoid TPM(Token per Minute) limit in an regon, followig is policy to randomly load balance across multiple OpenAI models, Add named values for backend-url-1 & backend-url-2

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/6bcb248c-9825-434e-b42e-c1179b32f99f)

* Once Inbound policies is configured for load balancing along with managed identity , you can verify header details from response that request is answered by different OpenAI model for each request. Also note that both API request is not using any authentcation methord, All Auth handled by APIM itself

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/eedf589b-8549-4072-9d53-f974896fe682)

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/12706cbb-4994-4c98-b1e8-6dee4a4c0f15)

* Since APIM handle authentication on behalf of client request ,to restrict that only legitimate request should get response then we will have enable APIM Subscripiton keys as mandatory pparameters in client request. Once its enable only request that contains APIM subscripiton keys will received response, all other request will get access denied error

![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/7fe03867-f30a-4228-8d6e-134f365d9ae7)

* Sent post call to APIM frontend without subscripiton keys 
![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/dd4ace61-b677-49c0-a460-53f842edc45e)

* once subscripiton keys are included in client request, APIM will verify that request is legitimate & forward to OpenAI models
![image](https://github.com/Osshaikh/Integrating-APIM-with-OPAI/assets/44756471/9ca11da2-3b2c-43e7-a2ea-288127c5d927)

