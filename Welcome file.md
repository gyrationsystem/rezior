
## Provider Guide
*Testing Title*

        Response :{
    }

**sdfsd**
Provider APIs allow a coupon provider to deposit/ delete a serialized data string (the serialized data string distributed to a consumer) into a manufacturer’s Master Offer File along with a series of other functions both required and optional to enhance the consumer experience. A provider will only be able to deposit serialized data strings into Master Offer Files of those manufacturers who have authorized them.  
  
**REQUIRED PROVIDER FUNCTIONS**

The following required functions are used to meet industry standards and maintain consistent basic consumer experience functions across providers. All required functions will require proof of application before moving to the production server.  [Click here](https://help.thecouponbureau.org/docs/provider-guide-to-universal-digital-coupons)  to view our Provider Guide to Universal Digital Coupons for more information about all provider functions.

-   DEPOSIT SERIALIZED DATA STRINGS USING PROVIDER PREFIX (instructions below)
-   DELETE SERIALIZED DATA STRINGS (instructions below)
-   APPLY FETCHCODE TO CONSUMER EXPERIENCE (instructions below)

#### Step #1 : Get Access Token

    curl -X POST '/access_token' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    --data '{ 
        "access_key": "ACCESS_KEY", 
        "secret_key": "SECRET_KEY" 
    }'

The Access Token will be valid for 24 hours. You should cache it and use the same access token for next 23 hours 59 mins to call any other APIs.

#### Step #2 : Get authorized master offer files.

  
Manufacturers / Authorized partners will authorize provider to distribute master offer files. To get the list of all authorized master offer files, use  **My master offer files API**

    curl -X GET '/provider/base_gs1s' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN' 
        

####   Step #3 : Generate your serialized data string

  
To accommodate multiple providers on a single master offer file, a “provider prefix” will be assigned and implemented as part of the serial number for each data string. This will resolve any potential duplication of serial numbers across providers. Follow the below process to generate your serialized data string

Master Offer File : 8112............VLI`**PROVIDER_PREFIX**`CONSUMER_IDENTIFIER

VLI = Length(PROVIDER_PREFIX) + LENGTH(CONSUMER_IDENTIFIER) - 6 
  
Use Serialization Prefix API to get your PROVIDER_PREFIX

    curl -X POST '/provider/serialization_prefix' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN' 

####   Step #4 : Deposit serialized data string

    curl -X POST '/provider/deposit' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN' \
    --data '{"gs1s":"<Comma separated GS1s>"}'

####   Recommendations for Consumer Presentment

#####   For Single Data String

Display the Single Data String as QR code to stay consistent with all display recommendations. If the QR code is not scannable, get the fetch_code and render as one dimensional Code-128 format. Also display the fetch_code below the one dimensional barcode so that the cashier can manually enter in case the screen is broken and one dimensional barcode is not scannable.

#####   For Provider Bundle (bundle of only provider authorized offers)

Provider should implement bundling of their serialized data strings and present the expanded_bundle_id as a QR code. If the QR code is not scannable, render the bundle_id as one dimensional Code-128 barocde. Also display the fetch_code below the one dimensional barcode so that the cashier can manually enter in case the screen is broken and one dimensional barcode is not scannable.

#####  For Universal Bundle (bundling across providers utilizing verified credential)

Provider could implement universal bundling and present the expanded_bundle_id as a QR code. If the QR code is not scannable, render the bundle_id as one dimensional barcode in Code128 format. Also display the fetch_code below the one dimensional barcode so that the cashier can manually enter in case the screen is broken and one dimensional barcode is not scannable.

![](https://tcb-static.s3.amazonaws.com/imgs/provider_presentment.png)

NOTE: bundle_id, expired_coupon_id and fetch_code are the codes that are retrieved in real-time from a consumer's phone. Bundle codes are valid for 60 minutes and fetch_code is valid for 10 minutes and will expire. Its recommended to refresh the code automatically without user action.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM5ODIwOTQ5NiwxNzM2MTI2MTE1LC0xOD
E0MzA2NjI2LDE3MTMwNTUxODIsLTE5NzI2ODkzNjQsMzY3NDMy
MTI5LC0xMTI0ODYyNzU2LC0xMjg0MTM1NTc1LDEzMjk5MzEyNT
IsLTgzODA5NDEzMywtMTMyNzA4NjUzMCwtMTUwMDIzMjEzNSw5
MTYyMjYwOTQsLTE3Njk1MzYxNDYsLTE2MDMxNDgwNTMsLTk5MT
EyNjQzOSwyMDMxOTk2OTgzLC01Mjg5NjU0NDksLTcxNzI2MDI3
OCwxMTYwMTMxOTM2XX0=
-->