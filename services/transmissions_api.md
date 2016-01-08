# Group Transmissions

A transmission is a collection of messages belonging to the same campaign.  It is also known as a mailing.  The Transmissions API provides the means to manage transmissions.  Messages in the transmissions are generated and sent to a specified list of recipients using a specified message template. The recipient list can be a stored list created using the Recipient Lists API, or it can be created "inline" as part of a transmission.  Similarly, the message template can be a stored template created using the Templates API, or it can be created "inline" as part of a transmission.  Messages are generated for the transmission for all specified recipients using the message template and performing substitution of data as necessary.

In addition, engagement tracking options can be set in the transmission to track message opens and clicks.

## Transmission Attributes 

| Field         | Type     | Description                           | Required         | Notes   |
|--------------------|----------------      |---------------------------------------|--------------------------|--------|
|id |string |ID of the transmission |no |Read only.  A unique ID is generated for each transmission on submission. |
|state |string  |State of the transmission  | no | Read only.  Valid responses are "submitted", "Generating", "Success", or "Canceled". |
|options | JSON object | JSON object in which transmission options are defined | no | For a full description, see the Options Attributes.
|recipients | JSON array or JSON object | Inline recipient objects or object containing stored recipient list ID |yes | Specify a stored recipient list or specify recipients inline.  When using a stored recipient list, specify the "list_id" as described in Using a Stored Recipient List.  Otherwise, provide the recipients inline using the fields described in the Recipient List API documentation for Recipient Attributes. |
|campaign_id | string |Name of the campaign|no|Maximum length - 64 bytes| 
|description | string |Description of the transmission|no | Maximum length - 1024 bytes| 
|metadata|JSON object|Transmission level metadata containing key/value pairs |no| Metadata is available during events through the Webhooks and is provided to the substitution engine.  A maximum of 1000 bytes of merged metadata (transmission level + recipient level) is available with recipient metadata taking precedence over transmission metadata when there are conflicts.  |
|substitution_data|JSON object|Key/value pairs that are provided to the substitution engine| no | Recipient substitution data takes precedence over transmission substitution data. Unlike metadata, substitution data is not included in Webhook events. |
|return_path | string |Email to use for envelope FROM ( **Note:** SparkPost Elite only )| yes | To support Variable Envelope Return Path (VERP), this field can also optionally be specified inside of the address object of a specific recipient in order to give the recipient a unique envelope MAIL FROM. |
|content| JSON object | Content that will be used to construct a message | yes | Specify a stored template or specify inline template content. When using a stored template, specify the "template_id" as described in Using a Stored Template.  Otherwise, provide the inline content using the fields described in Inline Content Attributes.  Maximum size - 20MBs|
|total_recipients | number | Computed total recipients | no | Read only|
|num_generated | number | Computed total number of messages generated | no |Read only|
|num_failed_generation| number| Computed total number of failed messages | no | Read only|
|num_invalid_recipients | number | Number of recipients that failed input validation |no |Read only|


### Options Attributes
| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|start_time | string | Delay generation of messages until this datetime.  For additional information, see Scheduled Transmissions. |no - defaults to immediate generation | Format YYYY-MM-DDTHH:MM:SS+-HH:MM or "now". Example: '2015-02-11T08:00:00-04:00'.|
|open_tracking|boolean| Whether open tracking is enabled for this transmission| no |If not specified, the setting at template level is used, or defaults to true. | 
|click_tracking|boolean| Whether click tracking is enabled for this transmission| no |If not specified, the setting at template level is used, or defaults to true. | 
|transactional|boolean|Whether message is transactional or non-transactional for unsubscribe and suppression purposes | no |If not specified, the setting at template level is used, or defaults to false. |
|sandbox|boolean|Whether or not to use the sandbox sending domain ( **Note:** SparkPost only )| no |Defaults to false. |
|skip_suppression|boolean|Whether or not to ignore customer suppression rules, for this transmission only.  Only applicable if your configuration supports this parameter. ( **Note:** SparkPost Elite only )| no - Defaults to false |  Unlike most other options, this flag is omitted on a GET transmission response unless the flag's value is true. |

### Inline Content Attributes

The following attributes are used when specifying inline content in the transmission's "content" JSON object. Note that these attributes should not be present if using a stored template.

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|html    |string  |HTML content for the email's text/html MIME part|At a minimum, html or text is required.  |Expected in the UTF-8 charset with no Content-Transfer-Encoding applied.  Substitution syntax is supported. |
|text    |string  |Text content for the email's text/plain MIME part|At a minimum, html or text is required. |Expected in the UTF-8 charset with no Content-Transfer-Encoding applied.  Substitution syntax is supported.|
|subject |string  |Email subject line   | yes |Expected in the UTF-8 charset without RFC2047 encoding.  Substitution syntax is supported. |
|from |string or JSON  | Address _"from" : "deals@company.com"_ or JSON object composed of the "name" and "email" fields _"from" : { "name" : "My Company", "email" : "deals@company.com" }_ used to compose the email's "From" header| yes | Substitution syntax is supported. |
|reply_to |string  |Email address used to compose the email's "Reply-To" header | no | Substitution syntax is supported. |
|headers| JSON | JSON dictionary containing headers other than "Subject", "From", "To", and "Reply-To"  | no |See the Header Notes. |
|attachments| JSON | JSON array of attachments. | no | For a full description, see Attachment Attributes. |
|inline_images| JSON | JSON array of inline images. | no | For a full description, see Inline Image Attributes. |

#### Header Notes

* Headers such as "Content-Type" and "Content-Transfer-Encoding" are not allowed here as they are auto generated upon construction of the email.
* The "To" header should not be specified here, since it is generated from each recipient's _address.name_ and _address.email_.
* Each header value is expected in the UTF-8 charset without RFC2047 encoding.
* Substitution syntax is supported.

#### email_rfc822 Notes

Alternately, the content JSON object may contain a single "email_rfc822" field.  email_rfc822 is mutually exclusive with all of the above fields.

| Field         | Type     | Description                           | Required   | Notes   |
|--------------------|:-:       |---------------------------------------|-----------------------|--------|
|email_rfc822    |string  |Pre-built message with the format as described by the [message/rfc822 Content-Type](http://tools.ietf.org/html/rfc2046#section-5.2.1) |no   |  See the email_rfc822 Notes. |

* Substitutions will be applied in the top-level headers and the first non-attachment text/plain and
first non-attachment text/html MIME parts only.
* Lone `LF`s and lone `CR`s are allowed. The system will convert line endings to `CRLF` where
necessary.
* The provided email_rfc822 should NOT be dot stuffed.  The system dot stuffs before sending the outgoing message.
* The provided email_rfc822 should NOT contain the SMTP terminator `\r\n.\r\n`.  The system always adds this terminator.
* The provided email_rfc822 in MIME format will be rejected if SparkPost and SparkPost Elite cannot parse the contents into a MIME tree.

### Attachment Attributes

Attachments for a transmission are specified in the content.attachments JSON array where each JSON object in the array is described by the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|--------------------|:-:       |---------------------------------------|-------------|------------------|
|type |string |The MIME type of the attachment; e.g., "text/plain", "image/jpeg", "audio/mp3", "video/mp4", "application/msword", "application/pdf", etc., including the "charset" parameter (text/html; charset="UTF-8") if needed. The value will apply "as-is" to the "Content-Type" header of the generated MIME part for the attachment. | yes |  |
|name |string |The filename of the attachment (for example, "document.pdf"). This is inserted into the filename parameter of the Content-Disposition header. | yes | Maximum length - 255 bytes |
|data |string |The content of the attachment as a Base64 encoded string.  The string should not contain \r\n line breaks.  The SparkPost systems will add line breaks as necessary to ensure the Base64 encoded lines contain no more than 76 characters each. | yes | The entirety of transmission content (text + html + attachments + inline images) is limited to 20 MBs |

### Inline Image Attributes

Inline images for a transmission are specified in the content.inline_images JSON array where each JSON object in the array is described by the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|--------------------|:-:       |---------------------------------------|-------------|------------------|
|type |string |The MIME type of the image; e.g., "image/jpeg".  The value will apply "as-is" to the "Content-Type" header of the generated MIME part for the image. | yes |  |
|name |string |The name of the inline image, which will be inserted into the Content-ID header. The image should be referenced in your HTML content using \<img src="cid:THIS_NAME"\>. The name must be unique within the content.inline_images array. | yes | Maximum length - 255 bytes |
|data |string | The content of the image as a Base64 encoded string.  The string should not contain \r\n line breaks.  The SparkPost systems will add line breaks as necessary to ensure the Base64 encoded lines contain no more than 76 characters each. | yes | The entirety of transmission content (text + html + attachments + inline images) is limited to 20 MBs |

### Using a Stored Template

The following attributes are used when specifying a stored template in the transmission's "content" JSON object. Note that these attributes should not be present when using inline content.

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|template_id|string| ID of the stored template to use | yes |Specify this field when using a stored template.  Maximum length -- 64 bytes|
|use_draft_template|boolean |Whether or not to use a draft template|no - defaults to false| If this field is set to true and no draft template exists, the transmission will fail.|

### Using a Stored Recipient List

The following recipients attribute is used when specifying a stored recipient list in the transmission. Note that this attribute should not be present when specifying recipients inline.

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|list_id | string  | Identifier of the stored recipient list to use | yes | Specify this field when using a stored recipient list. |

### Scheduled Transmissions
Use the _options.start_time_ attribute to delay generation of messages.  The scheduled time must be in the future and cannot be greater than 1 year from the time of submission.  If the scheduled time does not fall in that range, the transmission is not accepted.


## Create [/transmissions{?num_rcpt_errors}]
 
### Create a Transmission [POST]

You can create a transmission in a number of ways. In all cases, you can use the **num_rcpt_errors** parameter to limit the number of recipient errors returned.

**Note:** The "return_path" in the POST request body applies to SparkPost Elite only.

**Note:** Sending limits apply to SparkPost only. When a transmission is created in SparkPost, the number of messages in the transmission is compared to the sending limit of your account. If the transmission will cause you to exceed your sending limit, the entire transmission results in an error and no messages are sent.  Note that no messages will be sent for the given transmission, regardless of the number of messages that caused you to exceed your sending limit. In this case, the Transmission API will return an HTTP 420 error code with an error detailing whether you would exceed your hourly, daily, or sandbox sending limit. 

#### Using Inline Email Part Content

Create a transmission using inline email part content.

#### Using Inline RFC822 Content

Create a transmission using inline RFC822 content. Content headers are not generated for transmissions providing RFC822 content. They are expected to be provided as headers contained in the RFC822 content.

#### Using a Stored Recipients List

Create a transmission using a stored recipients list by specifying the "list_id" in the "recipients" attribute.

#### Using a Stored Template

Create a transmission using a stored template by specifying the "template_id" in the "content" attribute.  The "use_draft_template" field is optional and indicates whether to use a draft version or the published version of the template when generating messages.

#### Scheduling Transmissions

Create a scheduled transmission to be generated and sent at a future time by specifying "start_time" in the "options" attribute.

Scheduling a transmission that specifies a stored template will use the LATEST version of the template available at the time of scheduled generation.  The use of published versus draft versions follows the same logic in all transmission requests, whether scheduled or immediate generation. When "use_draft_template" is not specified (or set to false), the latest published version of the specified stored template is used. If "use_draft_template" is set to true, the latest draft version is used in the transmission instead.

Once message generation has been initiated, all messages in the transmission will use the template selected at the start of the generation. If a template update is made during the generation of a transmission that uses that template, the template update will succeed but the transmission will continue to use the version that was selected at the start of the generation.


+ Parameters
  + num_rcpt_errors (optional, number, `3`) ... Maximum number of recipient errors that this call can return, otherwise all validation errors are returned.


+ Request Create Transmission Using Inline Email Part Content (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
          "options": {
            "open_tracking": true,
            "click_tracking": true
          },

          "campaign_id": "christmas_campaign",
          "return_path": "bounces-christmas-campaign@flintstone.com",

          "metadata": {
            "user_type": "students"
          },

          "substitution_data": {
            "sender": "Big Store Team"
          },

          "recipients": [
            {
              "return_path": "123@bounces.flintstone.com",
              "address": {
                "email": "wilma@flintstone.com",
                "name": "Wilma Flintstone"
              },
              "tags": [
                "greeting",
                "prehistoric",
                "fred",
                "flintstone"
              ],
              "metadata": {
                "place": "Bedrock"
              },
              "substitution_data": {
                "customer_type": "Platinum"
              }
            }
          ],
          "content": {
            "from": {
              "name": "Fred Flintstone",
              "email": "fred@flintstone.com"
            },
            "subject": "Big Christmas savings!",
            "reply_to": "Christmas Sales <sales@flintstone.com>",
            "headers": {
              "X-Customer-Campaign-ID": "christmas_campaign"
            },
            "text": "Hi {{address.name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{user_type}}\n {{sender}}",
            "html": "<p>Hi {{address.name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n</p><p>Hurry, this offer is only to {{user_type}}\n</p><p>{{sender}}</p>"
          }
        }
        ```

+ Response 200 (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 1,
            "id": "11668787484950529"
          }
        }
        ```

+ Response 403 (application/json)

    + Body

        ```
        {
          "errors" : [
            {
              "description" : "Unconfigured or unverified sending domain.",
              "code" : "1100",
              "message" : "permission denied"
            }
          ]
        }
        ```

+ Request Create Transmission with Inline RFC822 Content (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body
  
            {
              "options": {
                "open_tracking": true,
                "click_tracking": true
              },
              "campaign_id": "christmas_campaign",
              "return_path": "bounces-christmas-campaign@flintstone.com",
              "metadata": {
                "user_type": "students"
              },
              "substitution_data": {
                "sender": "Big Store Team"
              },
              "recipients": [
                {
                  "return_path": "123@bounces.flintstone.com",
                  "address": {
                    "email": "wilma@flintstone.com",
                    "name": "Wilma Flintstone"
                  },
                  "tags": [
                    "greeting",
                    "prehistoric",
                    "fred",
                    "flintstone"
                  ],
                  "metadata": {
                    "place": "Bedrock"
                  },
                  "substitution_data": {
                    "name": "Will Smith"
                  }
                },
                {
                  "address": {
                    "email": "abc@flintstone.com",
                    "name": "Fred Fintstone"
                  },
                  "tags": [
                    "greeting",
                    "prehistoric",
                    "fred",
                    "flintstone"
                  ],
                  "metadata": {
                    "place": "MD"
                  },
                  "substitution_data": {
                    "name": "Fred"
                  }
                }
              ],
              "content": {
                "email_rfc822": "Content-Type: text\/plain\r\nTo: \"{{address.name}}\" <{{address.email}}>\r\n\r\n Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{user_type}}\n {{sender}}\r\n"
              }
            }

+ Response 200 (application/json)

  + Body
  
            {
              "results": {
                "total_rejected_recipients": 0,
                "total_accepted_recipients": 2,
                "id": "11668787484950529"
              }
            }

+ Response 400 (application/json)

  + Body

            {
              "errors" : [
                {
                  "description" : "Unconfigured or unverified sending domain.",
                  "code" : "7001",
                  "message" : "Invalid domain"
                }
              ]
            }


+ Request Create Transmission with Stored Recipient List (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body
  
            {
                "campaign_id": "christmas_campaign",
                "return_path": "bounces-christmas-campaign@flintstone.com",

                "recipients": {
                  "list_id": "christmas_sales_2013"
                },

                "content": {
                  "from": {
                    "name": "Fred Flintstone",
                    "email": "fred@flintstone.com"
                  },

                  "subject": "Big Christmas savings!",

                  "text": "Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{user_type}}\n {{sender}}",
                  "html": "<p>Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n</p><p>Hurry, this offer is only to {{user_type}}\n</p><p>{{sender}}</p>"
                }
            }

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "total_rejected_recipients": 0,
                "total_accepted_recipients": 10,
                "id": "11668787484950529"
              }
            }

+ Response 404 (application/json)

  + Body
  
            {
              "errors": [
                {
                  "message": "resource not found",
                  "description": "List 'christmas_sales_2013' does not exist",
                  "code": "1600"
                }
              ]
            }

+ Request Create Transmission with Stored Template (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body
  
            {
              "options": {
                "open_tracking": true,
                "click_tracking": true
              },

              "campaign_id": "thanksgiving_campaign",

              "content": {
                "template_id": "christmas_offer",
                "use_draft_template": false
              },

              "return_path": "bounces-christmas-campaign@flintstone.com",

              "metadata": {
                "user_type": "students"
              },
              "substitution_data": {
                "subkey": "subvalue"
              },

              "recipients": [
                {
                  "return_path": "123@bounces.flintstone.com",
                  "address": {
                    "email": "wilma@flintstone.com",
                    "name": "Wilma Flintstone"
                  },
                  "tags": [
                    "greeting",
                    "prehistoric",
                    "fred",
                    "flintstone"
                  ],
                  "metadata": {
                    "place": "Bedrock"
                  },
                  "substitution_data": {
                    "subrcptkey": "subrcptvalue"
                  }
                },
                {
                  "return_path": "456@bounces.flintstone.com",
                  "address": {
                    "email": "abc@flintstone.com"
                  },
                  "tags": [
                    "greeting",
                    "prehistoric",
                    "fred",
                    "flintstone"
                  ],
                  "metadata": {
                    "place": "MD"
                  }
                }
              ]
            }

+ Response 200 (application/json)

  + Body
  
            {
              "errors": [
                {
                  "message": "transmission created, but with validation errors",
                  "code": "2000"
                }
              ],
              "results": {
                "rcpt_to_errors": [
                  {
                    "message": "required field is missing",
                    "description": "address.email is required for each recipient",
                    "code": "1400"
                  }
                ],
                "total_rejected_recipients": 1,
                "total_accepted_recipients": 1,
                "id": "11668787484950530"
              }
            }

+ Response 404 (application/json)

  + Body
  
            {
              "errors": [
                {
                  "message": "resource not found",
                  "description": "template 'christmas_offer' does not exist",
                  "code": "1600"
                }
              ]
            }

+ Request Number of Messages Exceeds Sending Limit (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body
  
            {
                "campaign_id": "christmas_campaign",

                "recipients": {
                  "list_id": "list_exceeds_sending_limit"
                },

                "content": {
                  "from": {
                    "name": "Fred Flintstone",
                    "email": "fred@flintstone.com"
                  },

                  "subject": "Big Christmas savings!",

                  "text": "Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n Hurry, this offer is only to {{user_type}}\n {{sender}}",
                  "html": "<p>Hi {{name}} \nSave big this Christmas in your area {{place}}! \nClick http://www.mysite.com and get huge discount\n</p><p>Hurry, this offer is only to {{user_type}}\n</p><p>{{sender}}</p>"
                }
            }

+ Response 420 (application/json)

  + Body
  
            {
              "errors": [
                {
                  "message": "Exceed Sending Limit (hourly)",
                  "code": "2101"
                }
              ]
            }

+ Response 420 (application/json)

  + Body
  
            {
              "errors": [
                {
                  "message": "Exceed Sending Limit (daily)",
                  "code": "2102"
                }
              ]
            }

+ Response 420 (application/json)

  + Body
  
            {
              "errors": [
                {
                  "message": "Exceed Sending Limit (sandbox)",
                  "code": "2103"
                }
              ]
            }

+ Request Create Scheduled Transmission (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body
  
            {
                "name" : "Fall Sale",
                "campaign_id": "fall",
                "return_path": "deals@company.com",

                "options": {
                  "start_time" : "2015-10-11T08:00:00-04:00",
                  "open_tracking": true,
                  "click_tracking": true
                },

                "recipients": {
                  "list_id": "all_subscribers"
                },

                "content": {
                  "template_id" : "fall_deals"
                }
            }

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "total_rejected_recipients": 1000,
                "total_accepted_recipients": 0,
                "id": "11668787484950529"
              }
            }

+ Request Create Transmission with attachments (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
          "campaign_id" : "attachment_example",
          "recipients": [
            {
              "address": "wilma@flintstone.com"
            }
          ],
          "content": {
            "from": {
              "email": "billing@company.example",
              "name": "Example Company"
            },

            "subject": "Billing statement",
            "html": "<b>Please see your attached billing statement</b>",
            "attachments" : [
              {
                "type" : "application/pdf",
                "name" : "billing.pdf",
                "data" : "Q29uZ3JhdHVsYXRpb25zLCB5b3UgY2FuIGJhc2U2NCBkZWNvZGUh"
              },
              {
                "type" : "text/plain; charset=UTF-8",
                "name" : "explanation.txt",
                "data" : "TW92ZSBhbG9uZy4gIE5vdGhpbmcgdG8gc2VlIGhlcmUu"
              }
            ]
          }
        }
        ```

+ Response 200 (application/json)

    + Body

        ```
        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 1,
            "id": "11668787484950529"
          }
        }

+ Request Create Transmission with inline images (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
          "campaign_id" : "inline_image_example",
          "recipients": [
            {
              "address": "wilma@flintstone.com"
            }
          ],
          "content": {
            "from": {
              "email": "marketing@company.example",
              "name": "Example Company"
            },

            "subject": "Inline image example",
            "html": "<html><body>Here is your inline image!<br> <img src=\"cid:my_image.jpeg\"></body></html>",
            "inline_images" : [
              {
                "type" : "image/jpeg",
                "name" : "my_image.jpeg",
                "data" : "VGhpcyBkb2Vzbid0IGxvb2sgbGlrZSBhIGpwZWcgdG8gbWUh"
              }
            ]
          }
        }
        ```

+ Response 200 (application/json)

    + Body

        ```
        {
          "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 1,
            "id": "11668787484950529"
          }
        }



## Retrieve [/transmissions/{id}]

### Retrieve a Transmission [GET]

Retrieve the details about a transmission by specifying its ID in the URI path.

The response for a transmission using an inline template will include "template_id":"inline".  Inline templates cannot be specifically queried.

**Note:** The "return_path" is returned in the response for SparkPost Elite only.

+ Parameters
    + id (required, number, `11714265276872`) ... ID of the transmission

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    + Body

        ```
        {
          "results": {
            "transmission": {
              "id": "11750520427380741",
              "description": "",
              "state": "Success",
              "campaign_id": "white_christmas",
              "content": {
                "template_id": "Bob's template",
                "use_draft_template": false
              },
              "return_path": "fred@flintstone.com",
              "rcpt_list_chunk_size": 100,
              "rcpt_list_total_chunks": 1,
              "num_rcpts": 10,
              "num_generated": 10,
              "num_failed_gen": 0,
              "generation_start_time": "2014-05-22T15:12:59+00:00",
              "generation_end_time": "2014-05-22T15:13:00+00:00",
              "substitution_data": "",
              "metadata": {
                "key1": "value1"
              },
              "options": {
                "open_tracking": "",
                "click_tracking": ""
              }
            }
          }
        }
        ```

+ Response 404 (application/json)

    + Body

            {
              "errors": [
                {
                  "message": "resource not found",
                  "description": "Resource not found:transmission id 123",
                  "code": "1600"
                }
              ]
            }


## List [/transmissions{?campaign_id,template_id}]

### List all Transmissions [GET]
List an array of transmission summary objects.  A transmission summary object contains the "template_id", "id", "campaign_id", "description", and "state".  The following lists are supported:

* All multi-recipient transmissions
* Multi-recipient transmissions using a specific template
* Multi-recipient transmissions for a campaign
* Multi-recipient transmissions for a campaign that use a specific template

Note that single recipient transmissions are not returned.

By default, the list includes all transmissions for all campaigns.  Use the **template_id** parameter to specify a template and the **campaign_id** parameter to specify a campaign.

The response for transmissions using an inline template will include "template_id":"inline".  Inline templates cannot be specifically queried.

The example response shows a query on _campaign_id=thanksgiving_, with **template_id** not specified as part of the query.

+ Parameters
  + campaign_id (optional, string,`thanksgiving`) ... ID of the campaign used by the transmissions
  + template_id (optional, string,`thanksgiving_template`) ... ID of the template used by the transmissions

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    + Body

        ```
        {
          "results": [
            {
              "content" : {
                "template_id": "winter_sale"
              },
              "id": "11713562166689858",
              "campaign_id": "thanksgiving",
              "description": "",
              "state": "submitted"
            },
            {
              "content" : {
                "template_id": "inline"
              },
              "id": "11713562166689979",
              "campaign_id": "thanksgiving",
              "description": "",
              "state": "submitted"
            },
            {
              "content" : {
                "template_id": "thanksgiving_template"
              },
              "id": "11713048079237202",
              "campaign_id": "thanksgiving",
              "description": "",
              "state": "submitted"
            }
          ]
        }
        ```

## Delete [/transmissions/{id}]

### Delete a Transmission [DELETE]

Delete a transmission by specifying its ID in the URI path.

Only transmissions which are scheduled for future generation may be deleted.

Scheduled transmissions cannot be deleted if the transmission is within 10 minutes of the scheduled generation time.


+ Parameters
    + id (required, string, `11714265276872`) ... ID of the transmission 

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    +  Body

        {
        }

+ Response 404 (application/json)

  + Body
          {
            "errors": [
              {
                "message": "resource not found",
                "code": "1600",
                "description": "Resource not found:transmission id 999999999"
              }
            ]
          }

+ Response 409 (application/json)

  + Body
          {
            "errors": [
              {
                "message": "too close to generation time to delete transmission",
                "code": "2003",
                "description": "Deletion time window (660 seconds) doesn't permit transmission deletion"
              }
            ]
          }

+ Response 409 (application/json)

  + Body
          {
            "errors": [
              {
                "message": "transmission database record is in an invalid state for deletion",
                "code": "2006",
                "description": "Unable to delete a transmission that is in progress (state=Generating)"
              }
            ]
          }

+ Response 409 (application/json)

  + Body
          {
            "errors": [
              {
                "message": "transmission database record is in an invalid state for deletion",
                "code": "2006",
                "description": "Unable to delete a transmission that has completed (state=Success)"
              }
            ]
          }
