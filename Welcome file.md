
## Local Master Offer File Validation During Redemption

Accelerators and Retailers can make use of TCB's master offer file sync API to create a local database to speed up the redemption process. This process will eliminate the need of redemption API call if the basket does not match with the purchase requirements.

####   
Sync API (/syncmof/:from_date/:to_date/:mode)

Use this API to pull all master offer files purchase requirements into your server for faster processing. Use created mode first to fetch all the master offer files created till date in the selected date range. Then use updated mode to fetch the master offer files that are updated. We recommend running sync mof api call with updated mode in every day to keep your local database up to date.

The webhook message will look like this

{
  reportUrl: 'CSV report url. This URL is valid for 5 days', 
  action: 'report',
  job_id: 'The job id will be returned when you run the query',
  primary_signature: '...',
  secondary_signature: '...',
  event_timestamp: ...
}
                  

  
CSV file will include the followings (CSV File Header).

"primary_purchase_gtins","second_purchase_gtins","third_purchase_gtins","settled","base_gs1","campaign_start_time","campaign_end_time","redemption_start_time","redemption_end_time","primary_purchase_save_value","primary_purchase_requirements","primary_purchase_req_code","additional_purchase_rules_code","second_purchase_requirements","second_purchase_gs1_company_prefix","second_purchase_req_code","third_purchase_requirements","third_purchase_gs1_company_prefix","third_purchase_req_code","save_value_code","applies_to_which_item","store_coupon","donot_multiply_flag","update_datetime"
                  

  
reportUrl file is public and will be valid for 5 days. You could download the file and parse it to get the master offer file purchase requirements. Sample parsing code in NodeJS (using fast-csv library)

const fs = require('fs');
const path = require('path');
const csv = require('fast-csv');

fs.createReadStream(path.resolve(__dirname, '', 'file.csv'))
    .pipe(csv.parse({ headers: true }))
    .on('error', error => console.error(error))
    .on('data', row => console.log(row))
    .on('end', rowCount => console.log(`Parsed ${rowCount} rows`));
                  

  
fast-csv will convert every row to a JSON object and every key in the JSON object will be converted to string. You need to do appropriate formatting of the keys as defined in [Master Offer File Creation](https://try.thecouponbureau.org/developer/api_docs?menu=manufacturer&tab=master_offer_file&api=api_create_master_offer_file_post) API before storing in your local database.

####   
Use Local Database during coupon redemption

There are multiple ways coupons can be presented by consumers at the checkout counter.

-   **Single 8112 data string**: This will start with either  **81120 or 81121**  and length of the string  **> 14**
    Local database  **can be used**  in this scenario.  [**Click here**](https://try.thecouponbureau.org/developer/)  to get the parsing logic (in NodeJS) to extract the base_gs1 which you can use to get the purchase requirement from your local database.
-   **Fetch Code**: This will start  **8112**  and have fixed lengh  **(14)**
    Local database  **can not be used**  in this scenario. Call redemption API directly with this string to retrieve the purchase requirements.
-   **Bundle**: This will start  **81125**. Bundling of coupons is one of the fundamental features recommended by Coupon Bureau and our consumer app ecosystems created by providers apps and wallets will have this in built. This will enable consumers to select up to 15 coupons and then bundle them together into a single barcode to reduce the no of scans at the checkout counter. The Coupon Bureau supports two bundling formats.
    -   **Simple Bundle**: Displayed as a Code128 barcode
        Local database  **can not be used**  in this scenario. Use the redemption API directly.
    -   **Expanded Bundle**  (with MOF data): Displayed as a QR code.
        Local database  **can be used**  in this scenario.  [**Click here**](https://try.thecouponbureau.org/developer/)  to get the parsing logic (in NodeJS) to extract the base_gs1s and bundle_id. With the base_gs1s in hand, you will be able to use local database to extract the purchase requirements and validate basket. When the basket validation is done and you have identified the base_gs1s that are not applicable for this transaction, you will exclude these base_gs1s from the bundle by passing appropriate value in exclude_from_bundle parameter duing redemption.  [**Click here**](https://try.thecouponbureau.org/developer/)  to understand how you need to format the data for exclude_from_bundle parameter.

####   
Without Local Database

For retailers and accelerators not using local database validation, any code presented (fetch_code, 8112 data string, bundle id, expanded_bundle_id) will be passed using TCB redemption API. We recommend two phase redemption process. First phase with pre_process flag, which will return the exact same response without doing actual redemption. This process will help Retailers and Accelerators get the purchase requirements. Then the basket validation will happen and once the appropriate data stings are identified for the transaction, redeem the data strings with pre_process flag "no".

####   
Recommended Local Database Workflow

Below is a possible workflow to process redemption with local database efficiently. This process will update the local database in real time if the the master offer files are missing from local database or if there is any changes in the purchase requirements. You can use this approach without the sync API call. This approach will eventually build the local database as the redemption happens.![](https://tcb-static.s3.amazonaws.com/imgs/local_mof_flow.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTQ4MTg0ODcxLDkxNjIyNjA5NCwtMTc2OT
UzNjE0NiwtMTYwMzE0ODA1MywtOTkxMTI2NDM5LDIwMzE5OTY5
ODMsLTUyODk2NTQ0OSwtNzE3MjYwMjc4LDExNjAxMzE5MzYsMT
UxNjY0NjQ3Nyw0MDk4NDA4ODQsLTkyMjQzNDU5Nl19
-->