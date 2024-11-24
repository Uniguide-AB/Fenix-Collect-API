# About Fenix Collect  API

This 

## Resources

These resources are used by the webhooks and the REST API.

#### <a name="webhook"></a> [webhook](#webhook)  

| **Field**              | **Type**                      | **Remarks**                                                  |
| ---------------------- | ----------------------------- | ------------------------------------------------------------ |
| `payload_hash`         | `string`                      | SHA1 string based. Used to ensure                            |
| `payload_generated_at` | `string`                      | ISO 8601 Date. Date is updated each time the request is sent even for duplicates |
| `campaign`             | [campaign](#campaign)         | Resource                                                     |
| `contact`              | [contact](#contact)           | Resource                                                     |
| `bank_details`         | [bank_details](#bank_details) | Resource                                                     |


#### <a name="campaign"></a> [campaign](#campaign)  

| Field                          | Type                  | Remarks                                                      |
| ------------------------------ | --------------------- | ------------------------------------------------------------ |
| `preferred_landing_page`       | `string`              | The preferred landing page for the given campaign            |
| `default_landing_page`         | `string`              | The default landing page for the given campaign              |
| `uuid`                         | `string`              | UUID V4                                                      |
| `name`                         | `string`              | Name for public view                                         |
| `internal_name`                | `string`              | Name for internal use                                        |
| `external_recruitment_id`      | `string` `(nullable)` | Optional metadata, determined by the customer how they should work. |
| `external_campaign_group_id`   | `string` `(nullable)` | Optional metadata, determined by the customer how they should work. |
| `external_donation_purpose_id` | `string` `(nullable)` | Optional metadata, determined by the customer how they should work. |

#### <a name="contact"></a> [contact](#contact)  

| Field                    | Type                  | Remarks                                                      |
| ------------------------ | --------------------- | ------------------------------------------------------------ |
| `preferred_landing_page` | `string`              | The preferred landing page for the given contact             |
| `default_landing_page`   | `string`              | The default landing page for the given campaign              |
| `uuid`                   | `string`              | UUID V4                                                      |
| `name`                   | `string`              | Name of contact                                              |
| `ssn`                    | `string`              | Social security number, `YYYYMMDDXXXX`                       |
| `last_name`              | `string` `(nullable)` | Last name of contact (campaign setting)                      |
| `external_id`            | `string` `(nullable)` | The customers CRM system ID. Manually inserted by Fenix Collect users and is not guaranteed to be correct |
| `phone`                  | `string` `(nullable)` |                                                              |
| `email`                  | `string` `(nullable)` |                                                              |
| `created_at`             | `string`              | ISO 8601. When the contact was created in the database, not when it was converted |

#### <a name="bank_details"></a> [bank_details](#bank_details)  

| Field            | Type                  | Remarks                                                      |
| ---------------- | --------------------- | ------------------------------------------------------------ |
| `name`           | `string`              | UUID V4                                                      |
| `clearing`       | `string`              | Named aimed towards prospect                                 |
| `account`        | `string`              | Social security number, `YYYYMMDDXXXX`                       |
| `amount_to_give` | `string` `numeric`    | No fractions                                                 |
| `frequency`      | `string` `(nullable)` | free text. How often money may be withdrawn, if null the default should be considered monthly. |

## Webhooks

### Process

1. A contact conversion is completed by the prospect *(agreement has been signed with BankId)*.
2. Fenix Collect will send a payload to the registered URL(s) for the customer.
2. Each webhook request assumes a status code of 2xx to be returned.


### The Payload

They payload is JSON and we are able to provide custom headers with the payload. Example

```json
{
  "payload_hash": "356a192b7913b04c54574d18c28d46e6395428ab",
  "payload_generated_at": "2023-11-08T22:31:13+01:00",
  "campaign": {
    "preferred_landing_page": "https://autogiro.savetheworld.com/ABC123",
    "default_landing_page": "https://app.fenixcollect.se/pub/cn/1234fa...",
    "uuid": "8",
    "name": "Vårkampanj 2022",
    "internal_name": "VK22Q1",
    "due_at": "2024-12-12",
    "external_recruitment_id": null,
    "external_campaign_group_id": null,
    "external_donation_purpose_id": null
  },
  "contact": {
    "preferred_landing_page": "https://autogiro.savetheworld.com/EFG456",
    "default_landing_page": "https://app.fenixcollect.se/pub/cn/8g21...",
    "uuid": "9a9114a5-83df-48b0-9306-40d744e65cc0",
    "name": "My Möller",
    "last_name": null,
    "ssn": "196702107787",
    "external_id": "1234567",
    "phone": "+16466637874",
    "email": "qrunolfsdottir@schultz.net",
    "created_at": "2023-11-08T22:31:13+01:00"
  },
  "bank_details": {
    "name": "SEB",
    "clearing": "12345",
    "account": "12334346657",
    "amount_to_give": "100",
    "frequency": "Månadsvis"
  }
}
```



### Backoff / retries

If the original webhooks fails, a  total of 8 retries will occur automatically, then it will be marked as failed and a headsup is sent to Uniguide.

There retries will also backoff after each unique fail and endpoint.

* 1 min
* 5 min
* 15 min
* 1 hour
* 6 hours
* 12 hours
* 18 hours
* 24 hours

## REST API

Uniguide can provide an API Account to create "links"

Base URL `https://app.fenixcollect.se/pub/v1`

Each request need three headers 
* `API-KEY`
* `Content-Type: application/json`
* `Accept: application/json` 

| **Uri**                                         | **Request**                        | **Returns**                    | **Remarks**                                                  |
| ----------------------------------------------- | ---------------------------------- | ------------------------------ | ------------------------------------------------------------ |
| `GET` `/my-campaigns`                           | `[]`                               | Array of [campaign](#campaign) | You will only see the campaigns  your API  account has access too. The customer is responsible for ensuring access. |
| `POST` `/campaigns/{campaign:uuid}/new-contect` | [New Contact Request](#newContact) | Object of [contact](#contact)  | Check the desired settings for each campaign. The customer is responsible for ensuring which fields are required |

<a name="newContact"></a> [New Contact Request](#newContact)  

| Name               | Type                   | Remarks                                            |
| ------------------ | ---------------------- | -------------------------------------------------- |
| `name`             | `string`               | Required. Used as first name if `last_name`        |
| `last_name`        | `string` `(nullable)`  | Sometimes required. Depends on campaign settings   |
| `ssn`              | `string` `(nullable)`  | Sometimes required. Depends on campaign settings   |
| `phone`            | `string` `(nullable)`  | Sometimes required. Depends on campaign settings   |
| `email`            | `string` `(nullable)`  | Sometimes required. Depends on campaign settings   |
| `external_id`      | `string` `(nullable)`  | Sometimes required. Depends on campaign settings   |
| `suggested_amount` | `numeric` `(nullable)` | Whole numbers only. Dependent on campaign settings |

### Example request with default campaign settings

```curl
curl --location 'https://api.fenixcollect.se/pub/v1/campaigns/9d918668-8058-43f0-bf14-bb819e3fe80a/new-contact' \
--header 'API-KEY: fe3a11a4-cbd8-4946-8e6c-5ba0de15adac' \
--header 'Content-Type: application/json' \
--header 'Accept: application/json' \
--data '{
  "name": "Tony",
  "ssn": "19800113-9297"
}'
```

### Status codes

* `200 Ok` Resources was retrieved
* `201 Created` Resource was created
* `403 Forbidden` Wrong and or missing API-KEY
* `422 Unprocessable Content` Some or all parameters needs attention

#### Example of 422 
```json
{
    "message": "Ogiltigt personnummer: format: YYYYMMDD-XXXX",
    "errors": {
        "ssn": [
            "Ogiltigt personnummer: format: YYYYMMDD-XXXX"
        ]
    }
}
```



