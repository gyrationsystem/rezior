
## AI (8112) data string formats and processing

There are multiple ways coupons can be presented by consumers at the checkout counter. Each is necessary for accomplishing the unique needs of the shopper.

####   Data string formats:

-   **Single standard AI (8112) data string**: This will start with either 81120 or 81121 with a maximum length of 40 digits
-   **Fetch Code**: This will start with 8112 and have a fixed length (16 digits)
    -   The Fetch Code is a shortened data string that appears on a consumer's phone in the event the barcode cannot be scanned and an alternative data string is required for manual entry.
-   **Bundle**: This will start 81125.
    -   Bundling of coupons is one of the fundamental features that will enable consumers to select up to 15 coupons and then bundle them together into a single barcode to reduce the number of scans at the checkout counter. The Coupon Bureau supports two bundling formats.
        -   **Simple Bundle**: A single 27-28 digit data string starting with 81125
        -   **Expanded Bundle**: series of standard AI (8112) data strings in a single scannable barcode
            -   This execution is built for use of local database authentication. To learn more about local database authentication and expanded bundle assembly, visit Local MOF Validation.

####   Processing recommendations:

For retailers and accelerators not using local database validation, any code presented (fetch_code, 8112 data string, bundle id, expanded_bundle_id) will be passed using TCB redemption API. We recommend two phase redemption process. First phase with pre_process flag, which will return the exact same response without doing actual redemption. This process will help Retailers and Accelerators get the purchase requirements. Then the basket validation will happen and once the appropriate data stings are identified for the transaction, redeem the data strings with pre_process flag "no".

-   **Bundle ID**
    -   27-28 digit data string that starts with 8112
    -   Process
        -   Scanned as usual
        -   Sent to TCB
        -   TCB will unbundle and send offer purchase requirements to POS for processing as usual
-   **Fetch code**
    -   Manual code for broken phone
    -   Process
        -   Typed in by cashier
        -   Sent to TCB
        -   TCB will retrieve data string, authenticate and send offer purchase requirements to POS for processing as usual
-   **Series of coupons in a single QR code**
    -   Create to support local database validation
    -   If not using local database validation, all data strings sent to TCB as usual
    -   All QR codes will have back up code-128 barcode
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMjQ4NjI3NTYsLTEyODQxMzU1NzUsMT
MyOTkzMTI1MiwtODM4MDk0MTMzLC0xMzI3MDg2NTMwLC0xNTAw
MjMyMTM1LDkxNjIyNjA5NCwtMTc2OTUzNjE0NiwtMTYwMzE0OD
A1MywtOTkxMTI2NDM5LDIwMzE5OTY5ODMsLTUyODk2NTQ0OSwt
NzE3MjYwMjc4LDExNjAxMzE5MzYsMTUxNjY0NjQ3Nyw0MDk4ND
A4ODQsLTkyMjQzNDU5Nl19
-->