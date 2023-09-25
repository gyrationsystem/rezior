
## Near-realtime Webhook

If you need to receive realtime updates about the coupons (serialized data strings), you can register your webhook url and TCB will keep sending events in  **near realtime**. As TheCouponBureau® backend will be processing coupons at massive scale, our webhook processing layer could take upto 60 seconds to reach your endpoint. But most of the time you would get the webhook message within 10 seconds. TheCouponBureau® backend will generate events on the below actions. If you are authorized to receive the event, TCB will send push message to your configured webhook so that you can process it as per your need.

Before you subscribe your HTTPS endpoint, you must make sure that the HTTPS endpoint has the capability to handle the HTTPS POST requests that TCB uses to send the verification, subscription confirmation and notification messages. Usually, this means creating and deploying a web application (for example, a Java servlet if your endpoint host is running Linux with Apache and Tomcat) that processes the HTTPS requests from TCB.

####   Webhook Registration

TheCouponBureau® backend will verify the ownership of the webhook endpoint by sending a series of HTTP POST requests. You have to code your webhook endpoint to make sure you respond to the HTTP POST requests appropriately to complete the registration successfully. If your endpoint fails to do so, TCB will show appropriate error message in the portal.

####   Step #1 : Save Webhook

Go to your homepage, click on  **Enterprise Setting**, enter the webhook url and click  **save**  button. Your webhook url should be active. TCB backend will check the webhook by sending a  **HTTP POST**  request with body as shown below with  **Content-Type: text/plain; charset=UTF-8**

    POST / HTTP/1.1
    Content-Length: 1336
    Content-Type: text/plain; charset=UTF-8
    x-amz-sns-message-type: TCBVerifier
    Connection: Keep-Alive
    
    {
      "TCBVerifier": "..."
    }
                    

Your webook endpoint should return **HTTP Status Code: 200**. If it fails to do that, you will get an error message while saving the webhook.  
  
TCB will try to connect with your webhook by sending a **HTTP POST** request with body as shown below with **Content-Type: text/plain; charset=UTF-8**

    POST / HTTP/1.1
    x-amz-sns-message-type: SubscriptionConfirmation
    x-amz-sns-message-id: 165545c9-2a5c-472c-8df2-7ff2be2b3b1b
    x-amz-sns-topic-arn: arn:aws:sns:us-west-2:123456789012:MyTopic
    Content-Length: 1336
    Content-Type: text/plain; charset=UTF-8
    Host: example.com
    Connection: Keep-Alive
    User-Agent: Amazon Simple Notification Service Agent
    
    {
      "Type": "SubscriptionConfirmation",
      "MessageId": "...",
      "Token": "...",
      "TopicArn": "...",
      "Message": "...",
      "SubscribeURL": "https://.....",
      "Timestamp": "...",
      "SignatureVersion": "...",
      "Signature": "...",
      "SigningCertURL": "..."
    }
                    

Your code should parse the JSON document in the body of the HTTP POST request to read the name-value pairs that make up the TCB message. Use a JSON parser that handles converting the escaped representation of control characters back to their ASCII character values (for example, converting \n to a newline character). You can use an existing JSON parser such as the  [Jackson JSON Processor](https://github.com/FasterXML/jackson)  or write your own.

Once you convert the message to JSON, get the value of Type key. If **Type** is equal to **SubscriptionConfirmation**, your webhook endpoint should send a **HTTP GET** request to **SubscribeURL** to confirm the subscription. If **Type** is equal to **Notification**, your webhook endpoint should process the request as described below.

####   Step #2 : Process Incomming Request

A TCB push request looks like this example below. Note that the  **Message.data**  field is  [base64-encoded](https://tools.ietf.org/html/rfc4648#section-4).

    POST / HTTP/1.1
    x-amz-sns-message-type: Notification
    x-amz-sns-message-id: 165545c9-2a5c-472c-8df2-7ff2be2b3b1b
    x-amz-sns-topic-arn: arn:aws:sns:us-west-2:123456789012:MyTopic
    Content-Length: 1336
    Content-Type: text/plain; charset=UTF-8
    Host: example.com
    Connection: Keep-Alive
    User-Agent: Amazon Simple Notification Service Agent
    
    {
      "Type" : "Notification",
      "MessageId": "...",
      "Message": {
        "data": "SGVsbG8gQ2xvdWQgUHViL1N1YiEgSGV..........."
      },
      "SignatureVersion": "1",
      "Signature": "...",
      "SigningCertURL": "...",
      "UnsubscribeURL": "..."
    }
                    

Your webhook needs to handle incoming messages and return an HTTP status code to indicate success or failure. A success response is equivalent to acknowledging a message. The status codes interpreted as message acknowledgements by the TCB backend system are: 200, 201, 202, 204. A success response might look like this:

204 No Content
                    

  
If your webhook does not return a success code, TCB attempts up to three retries with a delay between failed attempts set at 20 seconds. Once you decode the **Message.data**, you will get the below JSON. event_timestamp will be in **Unix epoch format**. It will be the the number of milliseconds that have elapsed since January 1, 1970 (midnight EST), not counting leap seconds (in ISO 8601: 1970-01-01T00:00:00Z).  

#####  Events generating webhook push message

Below are the type of actions generates webhook push message

-   **Action on Serialized GS1s**  (All stakeholders connected to this gs1 will get webhook push)
    -   Deposit by provider (action:  **deposited**)
    -   Delete by provider (action:  **deleted**)
    -   Redemption by accelerator or retailer (action:  **redeemed**)
    -   Rollback by accelerator or retailer (action:  **rollback**)
    -   Receive serialized gs1 shared by other providers (action:  **shared**)
-   **Action on phone number change in a sharing app**
    -   **phone_number_change**
-   **Action on Master Offer File (by manufacturer or authorized partner)**  (Manufacturer, authorized partner and providers associated with this mof will get webhook push). Accelerators will also get webhook push for any purchase requirement change. Accelerator can use this to update locally synced master offer file purchase requirements for faster processing.
    -   MOF creation (action:  **mof_create**)
    -   MOF updation (action:  **mof_update**)
    -   MOF deletion (action:  **mof_delete**)
-   **Webhook push for reports**. (action:  **report**). When you issue a command to generate report, internally TCB creates and returns a job id. The job will be executed in the background and when the report is ready, it will be sent as a webhook push.

#####   Message Format: Action on Serialized GS1s

  
In case of deposit event, below message will be pushed to webhook  

    {
      gs1: 'GS1 data string (coupon)', 
      action: 'deposited',
      primary_signature: '...',
      secondary_signature: '...',
      provider: { email_domain: 'qples.com' },
      event_timestamp: ...,
      caller: 'Base64 encoded caller object'
    }
                    

  
In case of delete event (serialized data string is deleted by provider), below message will be pushed to webhook  

    {
      gs1: 'GS1 data string (coupon)', 
      action: 'deleted',
      primary_signature: '...',
      secondary_signature: '...',
      provider: { email_domain: 'qples.com' },
      event_timestamp: ...,
      caller: 'Base64 encoded caller object'
    }
                    

  
In case of redemption event (by retailer / accelerator), below message will be pushed to webhook  

    {
      gs1: 'GS1 data string (coupon)', 
      action: 'redeemed',
      primary_signature: '...',
      secondary_signature: '...',
      event_timestamp: ...,
      caller: 'Base64 encoded caller object'
    }
                    

  
In case of rollback event (redeemed serialized data string is roll backed by retailer / accelerator), below message will be pushed to webhook  

    {
      gs1: 'GS1 data string (coupon)', 
      action: 'rollback',
      primary_signature: '...',
      secondary_signature: '...',
      event_timestamp: ...,
      caller: 'Base64 encoded caller object'
    }
                    

  
In case of coupon sharing event, following message will be delivered to webhook.  

    {
        gs1: '811202120719781515159121831481597097',
        action: 'shared',
        primary_signature: '...',
        secondary_signature: '...',
        valid_till_timestamp: 1704016799000,
        phone_hash: '...',
        shared_by: 'example.com',
        event_timestamp: 1679423752364,
        campaign_metadata: {
          title: '',
          description: '',
          terms: '',
          mobile_image_url: '',
          desktop_image_url: '',
          dollar_amount: 2
        }
    }
                        

  
In case of phone number change event in coupon sharing app ecosystem, following message will be delivered to webhook.  

    {
        action: 'phone_number_change',
        primary_signature: '...',
        secondary_signature: '...',
        old_phone_hash: '...',
        new_phone_hash: '...',
        shared_by: 'example.com',
        event_timestamp: 1679423752364,
        change_location: '...'
    }
                    

#####   
Message Format: Action on Master Offer File

  
In case of master offer file creation / updation event, following message will be pushed to webhook  

    {
      base_gs1: '',
      action: 'mof_update', 
      primary_signature: '...',
      secondary_signature: '...',
      event_timestamp: ...,
      email_domain: 'manufacturer email domain',
      updated_mof_encoded: 'Base64 encoded master offer file object',
      caller: 'Base64 encoded caller object'
    }
                    

  
In case of master offer file deletion event, following message will be pushed to webhook  

    {
      base_gs1: '',
      internal_id: '', // This will come only in mof_delete action
      action: 'mof_delete', 
      primary_signature: '...',
      secondary_signature: '...',
      event_timestamp: ...,
      email_domain: 'manufacturer email domain',
      updated_mof_encoded: 'Base64 encoded master offer file object',
      caller: 'Base64 encoded caller object'
    }
                    

Use the  [atob](https://www.npmjs.com/package/atob)  library in NodeJS to convert the updated_mof_encoded back to master offer file object.  `var actual = JSON.parse(atob(updated_mof_encoded));`

updated_mof_encoded object will have the following information. You can keep the information in your own database to track changes in the master offer file.

    {
      internal_id: ,
      brand_id: , 
      base_gs1: , 
      description: , 
      campaign_start_time: , 
      campaign_end_time: , 
      redemption_start_time: , 
      redemption_end_time: , 
      rolling_expiration: , 
      rolling_expiration_days: , 
      total_circulation: , 
      primary_purchase_save_value: , 
      primary_purchase_requirements: , 
      primary_purchase_req_code: , 
      primary_purchase_gtins: , 
      additional_purchase_rules_code: , 
      second_purchase_requirements: , 
      second_purchase_gs1_company_prefix: , 
      second_purchase_req_code: , 
      second_purchase_gtins: , 
      third_purchase_requirements: , 
      third_purchase_gs1_company_prefix: , 
      third_purchase_req_code: , 
      third_purchase_gtins: , 
      save_value_code: , 
      applies_to_which_item: , 
      store_coupon: , 
      donot_multiply_flag: , 
      providers: , 
      active: , 
      campaign_metadata: 
    }
                                          

#####   Message Format: Webhook push for report

  
In case of any query initiation inside TCB platform, report will be sent to the webhook endpoint  

    {
      reportUrl: 'CSV report url. This URL is valid for 5 days', 
      action: 'report',
      job_id: 'The job id will be returned when you run the query',
      primary_signature: '...',
      secondary_signature: '...',
      event_timestamp: ...,
      metadata: {
        more_items: true | false,
        partNo: ...
      }
    }
                    

  
**metadata**  
If the requested query has more than 100k entries, TCB will process the reports in chunks and push every chunk as a seperate message / email. The attributes returned in the metadata will carry this information.  
**more_items** will return true if there are more items available which will arrive as a webhook message. TCB backend processes the query in asynchronous mode and the chunks might arrive in random order. You can use the **partNo** to assemble the chunks in proper order.  
  
  
Caller object will have the following information identifying the caller (portal user or API key identified using access_key) who did the action. This will be a base64 encoded JSON object. To get the actual JSON object, `var caller = JSON.parse(atob(caller));` (NodeJS Code). **NOTE:** Caller object will only be returned to the enterpise webhook that executed the action.  

    {
      identity: 'email address (portal user) or access_key (api user)', 
      access_type: 'portal or api',
      email_domain: ''
    }
                    

Primary / Secondary signature will help you validate that the message is originated at TheCouponBureau® server and it is intended for you. As soon as your callback is approved (Enterprise Setting page), you will see two secret keys (primary and secondary) associated with your callback.

![](https://d2bejab2nu51ww.cloudfront.net/imgs/callback_secret.png)

TheCouponBureau® backend will sign the message before pushing to your webhook using both primary and secondary keys and send those to your webhook as primary_signature / secondary_signature. Webhook implementation should recalculate the signatures and proceed execution if any one of the signatures matches.

-   primary_signature will be calculated using Primary Callback Secret
-   secondary_signature will be calculated using Secondary Callback Secret

Follow the below process to calculate the signature value (if value of the action key is "deposited" | "deleted" | "redeemed" | "rollback" | "shared")

primary_signature = md5 ( concat(gs1 + action + primary callback secret) )

secondary_signature = md5 ( concat(gs1 + action + secondary callback secret) )
                    

Follow the below process to calculate the signature value (if value of the action key is "phone_number_change")

primary_signature = md5 ( concat(new_phone_hash + action + primary callback secret) )

secondary_signature = md5 ( concat(new_phone_hash + action + secondary callback secret) )
                                          

Follow the below process to calculate the signature value (if value of the action key is "report")

primary_signature = md5 ( concat(reportUrl + action + job_id + primary callback secret) )

secondary_signature = md5 ( concat(reportUrl + action + job_id + secondary callback secret) )
                                          

Follow the below process to calculate the signature value (if value of the action key is "mof_create" | "mof_update" | "mof_delete")

primary_signature = md5 ( concat(base_gs1 + action + updated_mof_encoded + primary callback secret) )

secondary_signature = md5 ( concat(base_gs1 + action + updated_mof_encoded + secondary callback secret) )
                                          

Two secret keys concept will enable you rotate the key in case you want to change your keys. Here if the process you can use to safely rotate two keys.

-   Regenerate primary secret
-   Key change will take some time to reflect across TCB backend cluster
-   In the meantime, your backend will keep matching secondary_signature value
-   Once the key change is reflected across TCB cluster, which will take atleast 300 seconds, your primary_signature will start matching
-   You can now safely regenerate secondary secret
-   Your backend will keep matching primary_signature for sometime till secondary key will get stabilised in TCB cluster

Your webhook should calculate both signatures and continue if any one of the signatures matches. This will enable your to make sure that the messages are generated from TCB backend and not from other sources. As your endpoint is publicly available, its highly recommended to implement and validate signatures before processing the messages.

**NOTE: Duplicate Message**

TCB backend follows  **AT LEAST ONCE**  delivery mechanism. This means your webhook might receive duplicate messages. Its recommended to make your webhook  **idempotent**  using the  **MessageId**  key which will be passed along with every message.

**Retry window**

If the initial delivery of the message fails, TCB attempts up to three retries with a delay between failed attempts set at 20 seconds.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMzY0NzA1NzU3LC0xMjg0MTM1NTc1LDEzMj
k5MzEyNTIsLTgzODA5NDEzMywtMTMyNzA4NjUzMCwtMTUwMDIz
MjEzNSw5MTYyMjYwOTQsLTE3Njk1MzYxNDYsLTE2MDMxNDgwNT
MsLTk5MTEyNjQzOSwyMDMxOTk2OTgzLC01Mjg5NjU0NDksLTcx
NzI2MDI3OCwxMTYwMTMxOTM2LDE1MTY2NDY0NzcsNDA5ODQwOD
g0LC05MjI0MzQ1OTZdfQ==
-->