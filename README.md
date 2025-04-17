# X12-XML
XML to EDI converter
Certainly! Here’s a detailed, step-by-step guide to implement your EDI workflow in Azure using only Azure services (no custom code), specifically with Azure API Management (APIM), Logic Apps, and connectors.

---

## 1. **Set Up Azure API Management (APIM) to Receive EDI Requests**

### a. Create an API in APIM
- Go to the Azure Portal and open your APIM instance.
- Click on "APIs" > "+ Add API".
- Choose "Blank API" or "OpenAPI" (if you have a definition).
- Set the API name, and configure the URL suffix (e.g., `/navinet-edi`).
- Set the Web service URL to your Logic App HTTP trigger URL (to be created in the next step).

### b. Configure Security (Optional)
- Set up IP restrictions, subscription keys, or OAuth as needed to secure your API.

---

## 2. **Create a Logic App to Orchestrate the Workflow**

### a. Create a New Logic App
- In Azure Portal, click "Create a resource" > "Logic App".
- Fill in the required details and create the Logic App.

### b. Add HTTP Trigger
- In the Logic App Designer, search for "When a HTTP request is received".
- This will generate a URL after you save the Logic App.
- Copy this URL and set it as the backend for your APIM API.

---

## 3. **Validate and Parse EDI with X12 Connector**

### a. Add X12 Decode Action
- Click "+ New step" and search for "X12".
- Select "X12 Decode" (part of the Enterprise Integration Pack).
- Configure the X12 agreement:
  - Set up trading partners, agreements, and schemas in the Integration Account (required for X12).
  - Link your Integration Account to the Logic App.
  - Select the correct agreement for 270/271 transactions.

### b. Input the EDI Data
- Pass the body from the HTTP trigger as the input to the X12 Decode action.

---

## 4. **Build the SOAP Request Payload**

### a. Compose the SOAP Envelope
- Add a "Compose" action.
- In the Compose action, build your SOAP XML envelope.
- Use dynamic content to insert the decoded EDI (from X12 Decode) into the `<Payload><![CDATA[...]]></Payload>` section.
- Fill in other fields (PayloadType, ProcessingMode, etc.) as required, using static values or dynamic content.

#### Example Compose Body:
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:heal="http://healthedge.com">
  <soapenv:Header/>
  <soapenv:Body>
    <heal:realTimeTransaction>
      <body>
        <PayloadType>X12_270_Request</PayloadType>
        <ProcessingMode>RealTime</ProcessingMode>
        <PayloadID>@{guid()}</PayloadID>
        <TimeStamp>@{utcNow()}</TimeStamp>
        <SenderID>PayerC</SenderID>
        <ReceiverID>HospitalD</ReceiverID>
        <CORERuleVersion>2.2.0</CORERuleVersion>
        <Payload><![CDATA[@{outputs('X12_Decode')?['payload'] }]]></Payload>
      </body>
    </heal:realTimeTransaction>
  </soapenv:Body>
</soapenv:Envelope>
```
*(Replace dynamic expressions as per Logic App designer syntax.)*

---

## 5. **Send the SOAP Request to the External Connector**

### a. Add HTTP Action
- Add a new "HTTP" action.
- Set Method to POST.
- Set URI to `http://100.112.45.153:8181/connector/services/classic/BenefitEligibilityServiceStronglyTyped`.
- Set Headers:
  - Content-Type: text/xml; charset=utf-8
  - Authorization: (if needed, use Basic or other as required)
- Set Body to the output of your Compose action (the SOAP envelope).

---

## 6. **Extract the Payload from the SOAP Response**

### a. Parse the XML Response
- Add a "Parse XML" action.
- Use the output from the HTTP action as the content.
- Provide a schema for the SOAP response (you can generate this from a sample response).

### b. Extract the Payload Value
- Use Logic App expressions or the "XPath" action to extract the value inside the `<Payload><![CDATA[...]]></Payload>` element.

---

## 7. **Return the EDI Payload to Navinet**

### a. Add Response Action
- Add the "Response" action at the end of your Logic App.
- Set the Body to the extracted EDI payload.
- Set Content-Type to text/plain or as required by Navinet.

---

## 8. **(Optional) Add Monitoring and Logging**

- Add actions to log requests and responses to Azure Monitor, Log Analytics, or send email alerts for failures.

---

## **Summary Diagram**

1. **Navinet** → **APIM** → **Logic App (HTTP Trigger)**
2. **Logic App**: X12 Decode → Compose SOAP → HTTP (SOAP) → Parse XML → Extract Payload → Response

---

**All steps are performed using Azure Portal UI and Logic Apps Designer, with no custom code.**  
If you need a sample Logic App template or screenshots, let me know!
