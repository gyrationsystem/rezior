## Accelerator Getting Started Guide

Accelerators will facilitate retailer connectivity to The Coupon Bureau Positive Offer File. The Coupon Bureau will authorize an Accelerator to execute coupon redemption/ rollback on behalf of multiple retailers. Retailers will use this accelerator connectivity as their connection to The Coupon Bureau Positive Offer File (POF).  
  
![](https://d2bejab2nu51ww.cloudfront.net/imgs/accelerator_retailer_connectivity.png)

When an AI (8112) serialized data string is scanned at the POS, the POS makes an API request to POF with that serialized data string. The POF will then return the serialized data string with a status of “success” or “Failure” – if “success,” it will also send additional purchase requirement data. The POS then uses the additional purchase requirement data from the API response to validate offer against basket data and apply the discount or send a rollback response if the discount cannot be applied.  
  

There are three main components to consider:  
  
**Get Connected Retailers:**
Retailer will allow the accelerator to perform operation on their behalf. In this step, accelerator will pull the list of authorized retailers so that they can use the appropriate retailer_client_id while making the API calls  
  
**Redeem:**  
This is the primary API endpoint for TCB Positive Offer File functionality. The accelerator (on behalf of a retailer) sends scanned serialized data string(s) to POF, and POF responds with one or more serialized data strings. with "success" or "failure" for each.  
**"success"** means the serialized data string is available for redemption  
"failure" means the serialized data string was already redeemed or was in some other way invalid.  
  
Whenever the API returns any "success" response, it is also marking the relevant complete serialized data string as "redeemed", meaning the same complete serialized data string cannot be used more than once.  
On **"success"**, ", the serialized data string will be accompanied with offer details.  
On "failure", the failure reason will be returned per data string.  
  
Data strings will be use in the "rollback" scenario described below.  
  
**Rollback: ** 
Depending on the point of sale implementation, it may be necessary to "rollback" relevant data string redemptions. For example, if "redeem" is called before the point of sale confirms payment, and payment is rejected, any complete data strings previously "redeemed" during the checkout process must be rolled back.  
  

#### Step #1 : Get Access Token

    curl -X POST '/access_token' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    --data '{ 
        "access_key": "ACCESS_KEY", 
        "secret_key": "SECRET_KEY" 
    }'

The Access Token will be valid for 24 hours. You should cache it and use the same access token for next 23 hours 59 mins to call any other APIs.

#### Step #2 : Get connected retailers.

  
Accelerator can perform operation (redeem / rollback) on behalf of an authorized retailer. Retailer connected to The Coupon Bureau® will authorize an Accelerator to do the work on their behalf.  

    curl -X GET '/accelerator/retailers' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN'    

       

####   Step #3.1 : Redeem a Coupon  
  

    curl -X POST '/retailer/redeem' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN' \
    --data '{
        "gs1s":"<Comma separated GS1s>",
        "retailer_email_domain":"<Retailer email domain>"
      }' 

                 

**How to handle "redeem" API responses**  
  
"redeem" API responses will be returned in a JSON array, with each item corresponding to a GS1 from the "gs1s" request array.  
Response fields (per data string item):

  

> status -> "success" or "failure"   newly_redeemed -> List of data
> strings redeemed successfully
>           {
>             gs1: <data string redeemed>,
>             master_offer_file: <Master offer file (base gs1) data string> Refer to master_offer_files mapping to get the purchase
> requirements
>           }   master_offer_files -> Master offer file purchase details
> 
> 
> Example response for redeem
> 
> {   "status": "success",   "total_gs1s_processed": 3,  
> "already_redeemed": [
>       "811201777777754545412323432"   ],   "not_yet_live": [
>       "8112013333333676767223234543"   ],   "newly_redeemed": [
>       {
>           "gs1": "811201777777754545412323433",
>           "master_offer_file": "8112017777777545454"
>       }   ],   "master_offer_files": {
>       "8112017777777545454": {
>           "primary_purchase_gtins": [
>               "678987654323",
>               "876789876543"
>           ],
>           "second_purchase_gtins": [],
>           "third_purchase_gtins": [],
>           "primary_purchase_save_value": 100,
>           "primary_purchase_requirements": 1,
>           "primary_purchase_req_code": 0
>       }   },   "message": "Added 1 gs1(s) as available to redeem",   "execution_id": "9fc7f4db-c5ca-41c7-8a65-65be4bcd8e76",  
> "execution_time_in_ms": 2405,   "execution_start_time": 1587495466023
> }

        
        

####   Step #3.2 : Rollback a Coupon  
  

    curl -X DELETE '/accelerator/rollback/<GS1>' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN'

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMjY0NjI1OTMsLTEyODQxMzU1NzUsMT
MyOTkzMTI1MiwtODM4MDk0MTMzLC0xMzI3MDg2NTMwLC0xNTAw
MjMyMTM1LDkxNjIyNjA5NCwtMTc2OTUzNjE0NiwtMTYwMzE0OD
A1MywtOTkxMTI2NDM5LDIwMzE5OTY5ODMsLTUyODk2NTQ0OSwt
NzE3MjYwMjc4LDExNjAxMzE5MzYsMTUxNjY0NjQ3Nyw0MDk4ND
A4ODQsLTkyMjQzNDU5Nl19
-->