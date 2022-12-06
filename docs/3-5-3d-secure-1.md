# 3D Secure Flow (Test)

3D Secure is the new form of payment developed by Visa and Mastercard that makes possible to perform secure purchases on the Internet and authenticates the buyer as the legitimate holder of the card they are using.

When using our Gateway and Fiserv as the 3-D Secure provider, the authentication is performed in-line with the existing transaction flow. The process starts by performing a typical authorization or sale request with a desire to perform 3-D Secure authentication in the request.

The authorization is then placed into a ```WAITING``` status until the authentication process is completed. During authentication, the merchant may be required to update the original transaction request one or more times in order to move the process flow forward.

At the end of the authentication process, the original transaction is updated with the authentication results and the authorization is completed.

The sequence diagrams below map to the steps in the text that follows. The first diagram is for the frictionless flow. This means the issuer does not require the cardholder to authenticate.

![Sequence diagrama for frictionless flow!](/assets/images/3-5-1-3ds-frictionless.png "Frictionless flow")

The next diagram shows the flow when your customer has to authenticate, which means their issuer has requested they provide additional authentication details.

![Sequence diagrama for authenticate!](/assets/images/3-5-2-3ds-auth.png "Authenticate flow")

From now on you can only buy if you go through an authentication flow, which will authorize the purchase at that precise moment. If this flow is not completed, the customer will not be able to complete the purchase with the card.

In this way fraud in the network is impossible and guarantees total security in transactions.

**MOST of the transactions that we carry out by default need to go through the 3DSecure flow in our stores:** direct sales, msi sales, sales with token, scheduled recurrences, pre-auth, **with the sole exception of MIT recurring transactions** (Merchant Initiated Transactions), **token generation, paymentURL generation, and postauth.**

In other words, all transactions with the following requestTypes go through 3DS.

- PaymentCardPreAuthTransaction (preauthorization with card)
- PaymentCardSaleTransaction (sales with card)
- PaymentTokenPreAuthTransaction (preauthorization with token)
- PaymentTokenSaleTransaction (sales with token)

For this reason it is important that we understand the flow of transactions with 3DSecure, below is the diagram of a 3DS V2 authentication flow, for more information see the description of each of the processes according to the number below.
for this reason it is important that we understand the flow of transactions with 3DSecure, below is the diagram of a 3DS V2 authentication flow, for more information see the description of each of the processes according to the number below.

IMAGEN AQUI

## 1

Primary Transaction refers to the initial transaction where within the payload we can find the type of operation to be carried out (direct sale, msi sale, etc.) and the cardholder's information. This initial request must always include the **authenticationRequest** object and must contain the following parameters:

|**Parameters**|**Description**|
|----------|-----------|
|```authenticationType```|Protocol requested for authentication. It must be set with the value Secure3DAuthenticationRequest.|
|```termURL```|It is the callback url where the results of the authentication process are received from the ACS (Access Control Server) server, which is in charge of executing cardholder authentication.|
|```methodNotificationURL```|It is used to receive a notification when the 3DSMethod form is completed, by means of an http POST to a defined url. The URL must be unique and capable of identifying the transaction to which it belongs. Additionally it is possible to send the transaction reference inside the URL as a GET (queryString) parameter. Example: <https://www.mywebshop.com/process3dSecureMethod?reference=12345612352/>|
|```challengeIndicator```|In case you have a preference about the authentication flow to follow, you can append this optional parameter with any of the values listed below. In case of not sending it, the default value will be set to “01” – No preference.“01” = No preference (You have no preference on the flow)“02” = No challenge requested (You prefer not to request challenge)“03” = Challenge requested: 3DS Requestor Preference (You prefer that challenge be requested) “04” = Challenge requested: Mandate (There are some regions where the use of challenge is mandatory)|
|```challengeWindowSize```|You can append this parameter if you want to set the size of the authentication window shown to the cardholder during authentication. The possible values to send are. 01 = 250 x 400 02 = 390 x 400 03 = 500 x 600 04 = 600 x 400 05 = Full screen|

It is recommended provide billing and shipping information within the transaction to reduce the risk of authentication rejections. To do this, make sure to provide the data within your sales request.

The following JSON represents a sales transaction with the minimum requirements:

```json
{
  "requestType": "PaymentCardSaleTransaction",
  "transactionAmount": {
    "total": "122.04",
    "currency": "MXN"
  },
  "paymentMethod": {
    "paymentCard": {
      "number": "403587XXXXXX4977",
      "securityCode": "977",
      "expiryDate": {
        "month": "12",
        "year": "24"
      }
    }
  },
  "authenticationRequest": {
    "authenticationType": "Secure3DAuthenticationRequest",
    "termURL": "https://www.mywebshop.com/process3dSecure",
    "methodNotificationURL": "https://www.mywebshop.com/process3dSecureMethodNotification?transactionReferenceNumber=ffffffff-ba0b-539f-8000-016b2343ad7e",
    "challengeIndicator": "01",
    "challengeWindowSize": "01"
  }
}
```

## 2

If the response from the Gateway contains the '3DSMethod' element, this element should be embed within your site as a hidden iframe; No graphical interface is presented and its only functionality is to collect user information, helping to identify potential fraudulent transactions.

## 3

Here is a JSON response in this format:

```json
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0c80a3403e2c2def0-d-ea-28805-6810951-2",
  "ipgTransactionId": "838916029301",
  "transactionType": "SALE",
  "transactionTime": 1518811817,
  "approvedAmount": {
  "total": 122.04,
  "currency": "USD"
  },
  "transactionStatus": "WAITING",
  "authenticationResponse": {
    "type": "3D_SECURE",
    "version": "2.1",
    "secure3dMethod": {
      "methodForm": "&lt;!DOCTYPE iframe SYSTEM "about:legacy-compat"&gt;
      &lt;iframe id="tdsMmethodTgtFrame" name="tdsMmethodTgtFrame"
      style="width: 1px; height: 1px; display: none;" src="javascript:false;"
      xmlns="http://www.w3.org/1999/xhtml"&gt;
      &lt;!--.--&gt; &lt;/iframe&gt;&lt;form id="tdsMmethodForm"
      name="tdsMmethodForm"
      action=https://localhost.modirum.com:8543/dstests/ACSEmu2
      method="post"
      target="tdsMmethodTgtFrame" xmlns="http://www.w3.org/1999/xhtml"&gt;
      &lt;input type="hidden" name="3DSMethodData"
      value="eyAidGhyZWVEU1NlcnZlclRyYW5zSUQiIDogIjAwMDAwMDAwLTU2NzYtNTY2My04MDAwLTAwMDAw    &amp;#10;MDAwNDFhOSIsICJ0aHJlZURTTWV0aG9kTm90aWZpY2F0aW9uVVJMIiA6ICJodHRwczovL2xvY2Fs&amp;#10;aG9zdC5tb2RpcnVtLmNvbTo4NTQzL21kcGF5bXBpL01lcmNoYW50U2VydmVyP21uPVkmdHhpZD0x&amp;#10;NjgwOSZkaWdlc3Q9aSUyQnhhUEF5NWFOcVJRbllqNmozbWFDZlFJbTdFdjJYTmkwNnh6YmZNJTJG&amp;#10;R3MlM0QiIH0"/&gt; &lt;input type="hidden"
      name="threeDSMethodData"            
  value="eyAidGhyZWVEU1NlcnZlclRyYW5zSUQiIDogIjAwMDAwMDAwLTU2NzYtNTY2My04MDAwLTAwMDAw&amp;#10;MDAwNDFhOSIsICJ0aHJlZURTTWV0aG9kTm90aWZpY2F0aW9uVVJMIiA6ICJodHRwczovL2xvY 2Fs&amp;#10;aG9zdC5tb2RpcnVtLmNvbTo4NTQzL21kcGF5bXBpL01lcmNoYW50U2VydmVyP21uPVkmdHhpZD0x&amp;#10;NjgwOSZkaWdlc3Q9aSUyQnhhUEF5NWFOcV JRbllqNmozbWFDZlFJbTdFdjJYTmkwNnh6YmZNJTJG&amp;#10;R3MlM0QiIH0"/&gt;
      &lt;/form&gt;&lt;script type="text/javascript"
      xmlns="http://www.w3.org/1999/xhtml"&gt;
      document.getElementById("tdsMmethodForm").submit(); &lt;/script&gt;",
      "secure3dTransId": "3ac7caa7-aa42-2663-791b-2ac05a542c4a"
    }
  }
}
```

## 4

Not all issuers support data collection using the 3DSMethod form. If no response iframe is returned within 10 seconds, in these scenarios no information will be returned to methodNotificationURL and the flow should continue to send status.
**EXPECTED_BUT_NOT_RECEIVED**

This is done by sending a PATCH request with the following information: **method HTTP PATCH** point to the URL <https://cert.api.firstdata.com/gateway/v2/payments/{ipgTransactionId}/>. Where the ipgTransactionId will be obtained from the previous response.

```json
{
  "authenticationType": "Secure3D21AuthenticationUpdateRequest",
  "storeId": "12345500000",
  "methodNotificationStatus": "EXPECTED_BUT_NOT_RECEIVED",
  "securityCode": "XXX"
}
```

NOTE: The **storeId** field is not mandatory.

## 5

**RECEIVED** = If you received the notification within the first 10 seconds to the url defined in your **methodNotificationURL**.
This is done by sending a PATCH request with the following information: **method HTTP PATCH** point to the URL <https://cert.api.firstdata.com/gateway/v2/payments/{ipgTransactionId}/>. Where the ipgTransactionId will be obtained from the previous response.

```json
{
  "authenticationType": "Secure3D21AuthenticationUpdateRequest",
  "storeId": "12345500000",
  "methodNotificationStatus": "RECEIVED",
  "securityCode": "XXX"
}
```

NOTE: The **storeId** field is not mandatory.

## 6

Once this flow has been completed, the system will determine by which type of 3DS flow the transaction can be taken. When a transaction is considered low risk, the **Frictionless** flow is applied. In this case, the Gateway will proceed to authorize the transaction without any additional input from the cardholder.

## 7

When the API call is completed and the system detects that it is a flow with Challenge, the transaction is not authorized immediately. Instead, you will get the **WAITING** status and the parameters to redirect the cardholder to the issuer's Directory Server in order to authenticate the operation:

```json
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0c80a3403e2c2def0-d-ea-28805-6810951-2",
  "ipgTransactionId": "838916029301",
  "transactionType": "SALE",
  "transactionTime": 1518811817,
  "approvedAmount": {
    "total": 122.04,
    "currency": "USD"
  },
  "transactionStatus": "WAITING",
  "authenticationResponse": {
    "type": "3D_SECURE",
    "version": "2.1",
    "params": {
      "acsURL": "https://3ds-acs.test.modirum.com/mdpayacs/pareq",
      "termURL": "https://www.mywebshop.com/process3dSecure/",
      "cReq": "ewogICAiYWNzVHJhbCIgOiA...wMDAtMDAwMDAwMDA0MWE5Igp9",
      "sessiondata": "50F2156E03083CA665BCB4.."
    }
  }
}
```

The **authenticationResponse** field will contain the following information:

|**Parameter**|**Description**|
|----------|-----------|
|```type```|3D_SECURE|
|```version``` |2.1.0 (3DS VERSION)|
|```acsURL|URL to issuer site to redirect cardholder and post cReq and sessionData|
|```termURL```|URL of the merchant where the authentication results will be received|
|```cReq```|Encrypted message returned from the ACS|
|```sessionData```|Session parameters used for authentication. This field is not always returned|

You should implement a form that is automatically submitted within your website.

```xml
<form name="frm" method="POST" action="{value obtained on acsURL}">
  <input type="hidden" name="creq" value="ewogICAiYWNzVHJhbCIgOi...wMDAtMDAwMDAwMDA0MWE5Igp9">
  <input type="hidden" name="threeDSSessionData" value="50F2156E03083CA665BCB4..">
</form>
```

## 8

Once the POST is sent, the buyer will be redirected to the 3DS authentication screen where they will be asked to enter their token/key (this key depends on the issuing bank and the mechanism used by the bank).

![Token request!](/assets/images/3-5-esp-3ds-screen.jpg "Token request")

## 9

When the correct key is entered, the buyer will be redirected to the URL we received in **"termURL"** along with the following key-values as **POST** parameters.

|**Key**|**Value**|
|-------|---------|
|```cRes```|	String containing encrypted information, result of authentication|

## 10

Complete the transaction

Within your termURL you will need to build a PATCH request to confirm the authentication results to the Gateway. The components to send within the request are the following: **method HTTP PATCH** point to the URL <https://cert.api.firstdata.com/gateway/v2/payments/{ipgTransactionId}/>. Where the ipgTransactionId will be obtained from the previous response.

```json
{
  "authenticationType": "Secure3D21AuthenticationUpdateRequest",
  "storeId": "12345500000",
  "billingAddress": {
    "company": "Test Company",
    "address1": "5565 Glenridge Conn",
    "address2": "Suite 123",
    "city": "Atlanta",
    "region": "Georgia",
    "postalCode": "30342",
    "country": "USA"
  },
  "securityCode": "123",
  "acsResponse": {
    "cRes": "ewogICAiYWNzUmVmZX…Fuc1N0YXR…IKfQ=="
  }
}
```

The response to this request must contain the final result of the authentication:

```json
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0c80a3403e2c2def0-d-ea-28805-6810951-2",
  "ipgTransactionId": "838916029301",
  "transactionType": "SALE",
  "transactionTime": 1518811817,
  "approvedAmount": {
    "total": 122.04,
    "currency": "USD"
  },
  "transactionStatus": "APPROVED",
  "schemeTransactionId": "019078743804756",
  "processor": {
    "responseCode": "00",
    "responseMessage": "APPROVED",
    "authorizationCode": "OK7118"
  },
  "secure3dResponse": {
    "responseCode3dSecure": "1"
  }
}
```

## 11

We correctly mapped our response according to the Response Management section of this manual.
