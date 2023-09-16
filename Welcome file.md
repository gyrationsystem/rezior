## Manufacturer Getting Started Guide

Authorized Partner APIs allow the ability to connect to the TCB from your internal systems so that you can directly deposit and manage an 8112 offer into a manufacturer’s TCB account and retrieve redemption data. APIs are NOT required for a Authorized Partner to perform your functions as they relate to The Coupon Bureau, these APIs are designed specifically for partners who are looking to integrate these functions into your own system.


## Step #1 : Get Access Token

    {
        curl -X POST '/access_token' \   
        -H 'Content-Type: application/json' \ 
        -H 'x-api-key: ACCESS_KEY' \ 
        --data '{ 
        "access_key": "ACCESS_KEY", 
       "secret_key": "SECRET_KEY" 
         }
        }

The Access Token will be valid for 24 hours. You should cache it and use the same access token for next 23 hours 59 mins to call any other APIs.

## Step #2 : Get Connected Manufacturers

Required in step #3 where you will create master offer file on behalf of one manufacturer.

    {
        curl -X GET '/manufacturer_agent/manufacturers' \ 
        -H 'Content-Type: application/json' \ 
        -H 'x-api-key: ACCESS_KEY' \ 
        -H 'x-access-token: ACCESS_TOKEN' 
                
        }

You can cache the response of this API call as there will be no change in response until and unless another manufacturer authorizes the same Authorized Partner. Once you cache it, you can skip step #2 while creating a master offer file.

## Step #3 : Create Master Offer File for a Manufacturer


    curl -X POST '/manufacturer/base_gs1?email_domain=<MANUFACTURER_EMAIL_DOMAIN>' \ 
    -H 'Content-Type: application/json' \ 
    -H 'x-api-key: ACCESS_KEY' \ 
    -H 'x-access-token: ACCESS_TOKEN' \ 
    --data '{
        "data":{
            "base_gs1":"8112010031493140188",
            "brand_id":"XYZ",
            "description":"50% off ",
            "campaign_start_time":"04/21/2020",
            "campaign_end_time":"04/30/2020",
            "redemption_start_time":"04/22/2020",
            "redemption_end_time":"04/30/2020",
            "total_circulation":"100",
            "primary_purchase_save_value":"1",
            "primary_purchase_requirements":"1",
            "primary_purchase_req_code":"1",
            "primary_purchase_gtins":[
                "294239749273","2390843209"
                ],
            "additional_purchase_rules_code":"",
            "second_purchase_requirements":"",
            "second_purchase_gs1_company_prefix":"",
            "second_purchase_req_code":"",
            "second_purchase_gtins":[
                ""
                ],
            "third_purchase_requirements":"",
            "third_purchase_gs1_company_prefix":"",
            "third_purchase_req_code":"",
            "third_purchase_gtins":[
                ""
                ],
            "gln":"",
            "save_value_code":"1",
            "applies_to_which_item":"",
            "store_coupon":"1",
            "donot_multiply_flag":""
        }
    }'
            
    

Once the master offer file is created, you can use the below API call to manage the master offer files.

Assign / Unassign provider to master offer file

Delete master offer file

To get master offer file(s) created by you

My master offer files

Master offer file detail

## Step #3.1 : Get all Providers connected to TCB

{
    curl -X GET '/providers' \ 
    -H 'Content-Type: application/json' \ 
    -H 'x-api-key: ACCESS_KEY' \ 
    -H 'x-access-token: ACCESS_TOKEN' 
            
    }

You can call this API multiple times to authorize multiple providers. Once the provider is authorized, you can get the list of all authorized providers of a master offer file

Get assigned providers of master offer file

## Step #4.1 : Authorize a Authorized Partner

{
    curl -X PUT '/manufacturer/toggle/manufacturer_agent' \ 
    -H 'Content-Type: application/json' \ 
    -H 'x-api-key: ACCESS_KEY' \ 
    -H 'x-access-token: ACCESS_TOKEN' \ 
    --data '{"email_domain":"example1.com"}'
            
    }

You can call this API multiple times to authorize multiple Authorized Partners.

To unauthorize a Authorized Partner use

Call the same toggle api (/manufacturer/toggle/manufacturer_agent)

You can also get the list of all authorized Authorized Partners

Get connected Authorized Partners

## Step #4.2 : Give Brand Access to a Authorized Partner to create / manage master offer files and receive redemption data

{
curl -X PUT '/manufacturer/manufacturer_agents/:manufacturer_agent_email_domain/toggle_brand/:brand_internal_id' \ 
-H 'Content-Type: application/json'  
-H 'x-api-key: ACCESS_KEY' \ 
-H 'x-access-token: ACCESS_TOKEN' \ 
--data '{"access_type":"campaign_set"}'
            
    }

You can call this API to authorize your brand to a authorized partner with proper access_type. access_type attribute value should be one of the below

-   **view_only:**  can view all the master offer files of the brand.
-   **campaign_set:**  can edit campaign detail of the master offer file of the brand.
-   **full_set:**  can edit complete master offer file of the brand.
-   **full_set_with_lock:**  can edit complete master offer and of the brand and can lock it to stop future update. Once the master offer file is locked, only campaign data (following attributes) can be edited.
    -   description
    -   campaign_start_time
    -   campaign_end_time
    -   redemption_start_time
    -   redemption_end_time
    -   total_circulation
    -   primary_purchase_gtins
    -   second_purchase_gtins
    -   third_purchase_gtins

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcxNzI2MDI3OCwxMTYwMTMxOTM2LDE1MT
Y2NDY0NzcsNDA5ODQwODg0LC05MjI0MzQ1OTZdfQ==
-->