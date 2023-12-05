# WMS Feed

The WMS feed is a fixed byte length format rather than a CSV or a JSON.

## What is a fixed byte file format?

A fixed byte file format, also known as a fixed-width file format, is a type of flat file where each field or column has a fixed width or length. In contrast to delimited file formats (such as CSV or tab-delimited), where fields are separated by a specific delimiter character (like a comma or tab), fixed-width files allocate a predetermined number of characters for each field, and the data is organized accordingly.

Here are key characteristics of a fixed byte file format:

1. **Field Width:** Each field in the file has a fixed width or length specified in terms of the number of characters. For example, a field might be defined as 10 characters wide.

2. **Padding:** If the actual data for a field does not use the full allocated width, padding characters (typically spaces) are added to fill the remaining space. This ensures that each field has a consistent length.

3. **Positional Structure:** The position of each field is crucial, as the file format relies on the exact position of characters to interpret the data correctly. Fields are aligned based on their starting positions.

4. **No Delimiters:** Unlike delimited file formats, fixed-width files do not use special characters (e.g., commas or tabs) to separate fields. Instead, the file's structure is determined by the position and width of each field.

5. **Header Row:** Fixed-width files often include a header row that defines the layout and format of the data. The header specifies the starting position and width of each field.

6. **Readability:** While fixed-width files may be less human-readable than delimited files, they offer advantages in terms of simplicity, especially when dealing with data of a consistent and known structure.

Examples of systems that commonly use fixed-width file formats include legacy systems, mainframes, and certain database systems. In such cases, the fixed-width format provides a straightforward and efficient way to structure and store data.

## Field details

# Fixed Byte File Format Documentation

## Record Kind (S-1)

- **Description:** Header Indicator
- **Position:** 1-1

## Record Line (S-2)

- **Description:** Header Line #
- **Position:** 7-7
- **Example Value:** 0000000
- **Additional Information:** This field represents the sequential number assigned to each order header line. It is incremented for each record in the file, ensuring a unique identifier for each header line.


## CUST ORDER NO. (S-3)
- **Description:** SAP Delivery Number
- **Position:** 16-16
- **Mandatory:** Yes
- **Example Value:** [Value from the 'orderName' column in the orderHeader]
- **Additional Information:**
  - This field should contain the value of the Shopify Order Name.
  - The combination of the order number and the facility of fulfillment creates as a unique identifier even partial order fulfillment.


## Cust Ref No. (S-4)
- **Description:** Not used, leave empty
- **Position:** 16-16

## Order Date (S-5)
- **Description:** Order Date
- **Position:** 10-10
- **Format:** yyyy/mm/dd
- **Example Value:** 2023/04/20
- **Additional Information:**
  - This field represents the date of the sales order.
  - It contains the Order Date information for the Order Header.

## Plan Shipped Date (S-6)

- **Description:** Not used, leave empty
- **Position:** 10-10

## ETA (S-7)
{% hint style="danger" %}
New Era has not marked this field as not required. Need to verify if this can be left empty.
{% endhint %}
- **Description:** Requested Delivery Date
- **Position:** 10-10

## ETA (S-9)
{% hint style="danger" %}
New Era has not marked this field as not required. Need to verify if this can be left empty.
{% endhint %}
- **Description:** Delivery Time driven by VAS instruction
- **Position:** 2-2

## PCS (S-10)
{% hint style="danger" %}
At the header level does this include a rollup of total items shipped?
{% endhint %}
- **Description:** Delivery Quantity
- **Position:** 10-10
- **Example Value:** 8
- **Additional Information:**
  - The value is prepared at the line item level and includes the order item quantity.
  - Empty spaces will be added for this field on the header row.
  - Ensure that the data at the line item level is appropriately reflected in this field.
  - It provides information about the total quantity to be delivered for each order.

## BUSINESS TYPE (S-12)

- **Description:** Wholesale/EC Indicator
- **Position:** 1-1
- **Example Value:** 2
- **Additional Information:**
  - This field indicates the business type or category of the order.
  - The values are mapped as follows:
    - 1 = EC(B2B)
    - 2 = EC(B2C)
    - 3 = NAV(B2B)
    - 4 = Transfer Order
  - For Transfer Order, the value will be 4; for other types, the value will be 2.
  - Consider using OrderTypeId from OrderHeader, mapping it to:
    - 2 for SALES_ORDER
    - 4 for TRANSFER_ORDER.

## DLV SERVICE (S-13)

- **Description:** Delivery Service
- **Position:** 2-2
- **Example Value:** 3
- **Data Type:** Numeric
- **Additional Information:**
  - This field represents the type delivery company used.
  - Numeric values are assigned as follows:
    - 1=Perican
    - 2=Air
    - 3=Sagawa
  - It is usually mapped with the Carrier Party of the shipgroup, and the value is populated based on the carrier party (1, 2, or 3).
  - Provides information about the selected delivery service for the order.


## DLV PAYMENT (S-14)

- **Description:** Payment Type
- **Position:** 1-1
- **Example Value:** 1
- **Additional Information:**
  - This field represents the type of payment for the order.
  - Numeric values are assigned as follows:
    - 1=Prepaid (Credit Card)
    - 2=Cash on Delivery
    - 3=Amazon Pay
    - 4=Paidy
  - The payment method of the order is checked, and the value is populated accordingly (1, 2, 3, or 4).
  - The field is associated with the `paymentMethodTypeId` in the `OrderPaymentPreferenceAndType`.
{% hint style="warning" %}
There is no default value in case of unexpected payment methods; however, it is not expected to encounter such scenarios frequently.
{% endhint %}
{% hint style="danger" %}
Further clarification may be needed on how to handle orders using gift card payments.
{% endhint %}

## DLV PAYMENT (S-15)

- **Description:** Amount of Payment Including VAT (Only Cash on Delivery)
- **Position:** 19-23
- **Example Value:** (Numeric value representing the amount)
- **Additional Information:**
  - This field is applicable only for orders with the payment type "Cash on Delivery" (DLV PAYMENT S-14 = 2).
  - It represents the total payment amount, including VAT, for cash on delivery orders.
  - The amount should correspond to the order total, not individual item totals.
  - For payment methods other than Cash on Delivery, the field should be left empty.
  - The field is associated with the `grandTotal` in the `OrderHeader`.

## DLV PAYMENT (S-16)

- **Description:** Amount of Payment Including VAT and VAT Amount (Only Cash on Delivery)
- **Position:** 19-23
- **Example Value:** (Numeric value representing the amount)
- **Additional Information:**
  - This field is applicable only for orders with the payment type "Cash on Delivery" (DLV PAYMENT S-14 = 2).
  - It represents the total payment amount, including VAT, for cash on delivery orders, as well as the corresponding VAT amount.
  - There is a distinction between S-15 and S-16:
    - S-15 represents the total payment amount.
    - S-16 represents the sum of sales tax on the item and the shipping sales tax.
  - The value for this field is calculated based on the sum of order adjustments where `orderAdjustmentTypeId` is 'SHIPPING_SALES_TAX'.
  - Please refer to the provided Ruby code for specific details on the calculation: [Ruby Code](https://github.com/flagship-llc/newera_wms/blob/master/app/services/generate_order_file.rb#L268).
{% hint style="warning" %}
The distinction between S-15 and S-16 is not clear
{% endhint %}

## Consignee (S-17 to S-25)
(BILL-TO INFORMATION)

- **S-17:** SAP Bill-to Customer Code (Length: 10, Bytes: 10)
- **S-18:** Bill-to Name (Length: 35, Bytes: 35)
- **S-19:** Bill-to Address1 (Length: 75, Bytes: 75)
- **S-20:** Bill-to Address2 (Length: 75, Bytes: 75)
- **S-21:** Not used (Length: 35, Bytes: 35)
- **S-22:** Not used (Length: 35, Bytes: 35)
- **S-23:** Bill-to Postal Code (Length: 9, Bytes: 9)
- **S-24:** Not used (Length: 35, Bytes: 35)
- **S-25:** Bill-to Phone Number (Length: 14, Bytes: 14)


## Delivery (S-26 to S-34)

- **S-26:** SAP Ship-to Customer Code (Length: 10, Bytes: 10)
- **S-27:** Ship-to Name (Length: 35, Bytes: 35)
- **S-28:** Ship-to Address1 (Length: 75, Bytes: 75)
- **S-29:** Ship-to Address2 (Length: 75, Bytes: 75)
- **S-30:** Not used (Length: 35, Bytes: 35)
- **S-31:** Not used (Length: 35, Bytes: 35)
- **S-32:** Ship-to Postal Code (Length: 9, Bytes: 9)
- **S-33:** Not used (Length: 35, Bytes: 35)
- **S-34:** Ship-to Phone Number (Length: 14, Bytes: 14)

## STATUS (S-38)

- **S-38:** Order Status (Length: 2, Bytes: 2)
  - **Description:** Delivery status indicating the progress of the order fulfillment.
  - **Mandatory:** Yes
  - **Validity:** Should be one of the following values:
    - 60: SHIP ORDER (NECJ → Nittsu)
    - 70: SHIP RECEIVED (Nittsu → NECJ)
    - 80: SHIP CONFIRMED (Nittsu → NECJ)
    - 90: INVOICE NO. (NECJ → Nittsu)
  - **Default Value:** 60 (fixed value for warehouse brokered order items to be fulfilled)

## Invoice (S-39)

- **Description:** Delivery Slip Instuctions - Driven by VAS
- **Position:** 2-2
- **Default Value:** Always a fixed value of 2

<details>
<summary>All Mappings Options</summary>
- 1: NAVISION (B2B) Delivery Slip Only
- 2: EC Delivery Slip Only
- 30: Chain Store Slip (TSA)
- 31: Chain Store Slip (Robot)
- 32: Chain Store Slip (DAISEN)
- 33: Chain Store Slip (Mycal)
- 34: Chain Store Slip (neuve-a)
- 35: Chain Store Slip (KUROKAWA Sports)
- 36: Chain Store Slip (MITSUHASHI)
- 37: Chain Store Slip (AKATSUKA)
- 38: Chain Store Slip (fithouse)
- 4: NAVISION (B2B) Delivery Slip + Chain Store Slip
- 5: NAV Delivery Slip without Amount
- 6: Delivery Slip of MURASAKI Sports
- 70: Chain Store Slip Type 1 (Mycal)
- 71: Chain Store Slip Type 1 (KOTOBUKI SHOJI)
- 72: Chain Store Slip Type 1 (GEAR'S JAM)
- 73: Chain Store Slip Type 1 (YOSHIMOTO)
- 74: Chain Store Slip Type 1 (BIG AMERICAN SHOP)
- 75: Chain Store Slip Type 1 (YATOGOLF)
- 99: NO Delivery Slip
</details>

## DC Incidental Operation Flag (S-40 to S-42)

- **S-40:** DC Incidental Operation Flag (Length: 1, Bytes: 1)
  - **Description:** Instructions for incidental operations related to the delivery.
  - **Examples:**
    - 1: Attach Special Tag
    - 2: Bundle Special Tag
    - 3: Without Tag (Detach NECJ Tag)
    - 4: Attach 8% TAX Tag (*Print specific sentence according to NAV Flag on Jun 2012)
{% hint style="danger" %}
As of now we expect this to be blank. Need to confirm with New Era.
{% endhint %}

- **S-41:** DC Incidental Operation Flag (Length: 1, Bytes: 1)
  - **Description:** Instructions for incidental operations related to the delivery of special slips to the customer.
  - **Notes:**
    - 1: Send mail of Customer Special Slip
    - 2: Deliver Customer Special Slip
    - 3: Deliver Customer Special Slip in handwriting (*Print specific sentence according to NAV Flag on Jun 2012)
{% hint style="danger" %}
As of now we expect this to be blank. Need to confirm with New Era.
{% endhint %}

- **S-42:** DC Incidental Operation Flag (Length: 1, Bytes: 1)
  - **Description:** Instructions for incidental operations related to the attachment of packing lists.
  - **Notes:**
    - 1: Attach NewEra Packing List
    - 2: Attach each customer's Packing List
    - 3: Attach appendix
    - 4: XXX (Not fixed yet)
{% hint style="danger" %}
As of now we expect this to be blank. Need to confirm with New Era.
{% endhint %}

## DC Incidental Operation Comment (S-43 to S-44)

- **S-43:** DC comments 1 Driven by VAS
  - **OMS Mapping:**```Ship Group handling instructions```
- **S-44:** DC comments 2 Driven by VAS
  - Empty

## Delivery Slip / Invoice (S-45 to S-56)

- **S-45:** Not used
- **S-46:** Customer PO
- **S-47:** Not used
- **S-48:** Not used
- **S-49:** Sales Order Date
- **S-50:** Payment Terms Code
- **S-51:** Hard coded
- **S-52:** Total retail price (Including VAT)
- **S-53:** Sales with VAT
- **S-54:** Sales no VAT
- **S-55:** VAT
- **S-56:** Total Price (JPY)

## Shipping Cost (S-57 to S-103)

- **S-57 to S-59:** Not used
- **S-84 to S-92:** Not used
- **S-93 to S-103:** Not used

## Record Kind (S-61)

- **Description:** Detail indicator
- **Position:** 1-1

## Item Info (S-62 to S-103)

- **S-62:** SAP delivery schedule line
- **S-63:** SAP material + size
- **S-65:** Quantity
- **S-66:** HARDCODE = WH
- **S-67:** Nueve A JAN code
- **S-68 to S-103:** Not used