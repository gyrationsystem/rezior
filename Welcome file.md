
## Clearinghouse Getting Started Guide

Retailer clearinghouse will be able to view the redemption report of authorized retailers. Retailers connected to The Coupon Bureau® will authorize a clearinghouse to do the final settlement with the manufacturer. Once the clearinghouse is authorized, they will be able to view the redemption data in realtime.

#### Step #1 : Get Access Token

    curl -X POST '/access_token' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    --data '{ 
        "access_key": "ACCESS_KEY", 
        "secret_key": "SECRET_KEY" 
    }'

The Access Token will be valid for 24 hours. You should cache it and use the same access token for next 23 hours 59 mins to call any other APIs.

#### Get Funder

This API will send back manufacturer settlement billing details for the Funder ID sent by the retailer clearinghouse.

    curl -X GET '/clearinghouse/funder/FUNDER_ID' 
    -H 'Content-Type: application/json' 
    -H 'x-api-key: ACCESS_KEY' 
    -H 'x-access-token: ACCESS_TOKEN'
          

#### Get Updated Funders List

This API can be used to get a list of all manufacturer settlement billing details that have changed within a specific date range.

    curl -X GET '/clearinghouse/funders/updated_list/<FROM_DATE (YYYY-MM-DD)>/<TO_DATE (YYYY-MM-DD)>/<PAGE_NO (E.g. 0,1...)>' 
    -H 'Content-Type: application/json' 
    -H 'x-api-key: ACCESS_KEY' 
    -H 'x-access-token: ACCESS_TOKEN'
        

#### Base GS1 Validation

The purpose of this API is to confirm that each base data string in a redemption file is valid and to return the offers details for any necessary audit.

    curl -X GET '/clearinghouse/basegs1/<BASE_GS1>' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN'

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwODcwODYwNDAsLTEyODQxMzU1NzUsMT
MyOTkzMTI1MiwtODM4MDk0MTMzLC0xMzI3MDg2NTMwLC0xNTAw
MjMyMTM1LDkxNjIyNjA5NCwtMTc2OTUzNjE0NiwtMTYwMzE0OD
A1MywtOTkxMTI2NDM5LDIwMzE5OTY5ODMsLTUyODk2NTQ0OSwt
NzE3MjYwMjc4LDExNjAxMzE5MzYsMTUxNjY0NjQ3Nyw0MDk4ND
A4ODQsLTkyMjQzNDU5Nl19
-->