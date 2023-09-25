## Retailer Getting Started Guide

When an AI (8112) serialized data string is scanned at the POS, the POS makes an API request to POF with that serialized data string. The POF will then return the serialized data string with a status of “success” or “Failure” – if “success,” it will also send offer details. The POS then uses offer details from the API response to validate offer against basket data and apply the discount or send a rollback response if the discount cannot be applied.  
  
There are two main components to consider:  
  
**Redeem**:  
This is the primary API endpoint for TCB Positive Offer File functionality. The retailer sends scanned data string(s) to POF, and POF responds with one or more data strings with "success" or "failure" for each.  
  
**"success"** means the serialized data string is available for redemption.  
**"failure"** means the data string was already redeemed or was in some other way invalid.  
  
Whenever the API returns any "success" response, it is also marking the relevant data string as "redeemed", meaning the same data string cannot be used more than once.  
  
On **"success"**, the data string will be accompanied with offer details  
On **"failure"**, the failure reason will be returned per data string  
  
Data strings will be use in the "rollback" scenario described below.  
  
**Rollback:**  
It may be necessary to "rollback" a data string in the event that purchase requirements are not met or a failed payment.  
  
**Example Flows**  
Retailer calls the TCB "redeem" API as they scan GS1s during the checkout process.

-   In this flow, if a customer has 5 GS1s to scan, the retailer makes 5 calls to the TCB API during the checkout process. TCB will respond with applicable complete GS1s and "success" (valid) or "failure" (invalid) for each serialized GS1.
-   Before payment is processed, the retailer must determine which valid complete GS1s are applicable to the customer's basket, and apply discounts appropriately.
-   If any returned redeemed serialized GS1s are *not* applicable to the customer's basket, the TCB "rollback" API should be called for those un-applied serialized GS1s.
-   If the payment fails for any reason and the transaction itself is rolled back, the TCB "rollback" API should be called for all serialized GS1s previously returned by the TCB "redeem" API, whether or not they were applied to the basket.

Retailer scans GS1s during the checkout process, but waits to call the TCB **"redeem"** API until all items have been scanned.  

-   In this flow, if a customer has 5 GS1s to scan, the retailer would scan GS1s during or at the end of the checkout process, and then make one single TCB "redeem" API call just before payment is processed. TCB will respond with a list of complete GS1s with "success" (valid) or "failure" (invalid) for each complete GS1.
-   After the "redeem" API call is made, the retailer should determine which redeemed serialized GS1s are applicable to the customer's basket and apply the discounts.
-   If any returned redeemed serialized GS1s are *not* applicable to the customer's basket, the TCB "rollback" API should be called for those un-applied complete GS1s.
-   If the payment fails for any reason and the transaction itself is rolled back, the TCB "rollback" API should be called for all serialized GS1s previously returned by the TCB "redeem" API, whether or not they were applied to the basket.

#### Step #1 : Get Access Token

    curl -X POST '/access_token' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    --data '{ 
        "access_key": "ACCESS_KEY", 
        "secret_key": "SECRET_KEY" 
    }'

The Access Token will be valid for 24 hours. You should cache it and use the same access token for next 23 hours 59 mins to call any other APIs.

####   Step #2.1 : Redeem a Coupon

    curl -X POST '/retailer/redeem' \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN' \
    --data '{
        "gs1s":"<Comma separated GS1s>",
        "selected_store":"<Store Internal ID>",
        "store_address":"<Store Address (E.g. Target 2417, N Haskel, Dallas)>",
      }'
        

**How to handle "redeem" API responses**  
  
**"redeem"** API responses will be returned in a JSON array, with each item corresponding to a GS1 from the "gs1s" request array.  
Response fields (per GS1 item):

  

> status -> "success" or "failure"   newly_redeemed -> List of GS1s
> redeemed successfully
>           {
>             gs1: <GS1 redeemed>,
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
>               " 876789876543"
>           ],
>           "second_purchase_gtins": [],
>           "third_purchase_gtins": [],
>           "primary_purchase_save_value": 100,
>           "primary_purchase_requirements": 1,
>           "primary_purchase_req_code": 0
>       }   },   "message": "Added 1 gs1(s) as available to redeem",   "execution_id": "9fc7f4db-c5ca-41c7-8a65-65be4bcd8e76",  
> "execution_time_in_ms": 2405,   "execution_start_time": 1587495466023
> }

        

####   Step #2.2 : Rollback a Coupon

    curl -X DELETE '/retailer/rollback/<GS1 to Rollback> \
    -H 'Content-Type: application/json' \
    -H 'x-api-key: ACCESS_KEY' \
    -H 'x-access-token: ACCESS_TOKEN'

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyODQxMzU1NzUsMTMyOTkzMTI1MiwtOD
M4MDk0MTMzLC0xMzI3MDg2NTMwLC0xNTAwMjMyMTM1LDkxNjIy
NjA5NCwtMTc2OTUzNjE0NiwtMTYwMzE0ODA1MywtOTkxMTI2ND
M5LDIwMzE5OTY5ODMsLTUyODk2NTQ0OSwtNzE3MjYwMjc4LDEx
NjAxMzE5MzYsMTUxNjY0NjQ3Nyw0MDk4NDA4ODQsLTkyMjQzND
U5Nl19
-->