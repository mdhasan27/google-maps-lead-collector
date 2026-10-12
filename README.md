# google-maps-lead-collector
{
  "name": "Google Maps Leads Collector V2",
  "nodes": [
    {
      "parameters": {
        "formTitle": "Google Maps Lead Collection Form",
        "formDescription": "Fill out this form to search and collect high-quality Google Maps business leads.",
        "formFields": {
          "values": [
            {
              "fieldLabel": "Business types"
            },
            {
              "fieldLabel": "Location"
            },
            {
              "fieldLabel": "Number of leads",
              "fieldType": "number"
            },
            {
              "fieldLabel": "Minimum Rating",
              "fieldType": "dropdown",
              "fieldOptions": {
                "values": [
                  {
                    "option": "4.8"
                  },
                  {
                    "option": "4.2"
                  },
                  {
                    "option": "3.9"
                  }
                ]
              }
            },
            {
              "fieldLabel": "Country",
              "fieldType": "dropdown",
              "fieldOptions": {
                "values": [
                  {
                    "option": "Bangladesh"
                  },
                  {
                    "option": "India"
                  },
                  {
                    "option": "USA"
                  },
                  {
                    "option": "UK"
                  },
                  {
                    "option": "Canada"
                  }
                ]
              }
            },
            {
              "fieldLabel": "Language",
              "fieldType": "dropdown",
              "fieldOptions": {
                "values": [
                  {
                    "option": "English"
                  },
                  {
                    "option": "Bangla"
                  }
                ]
              }
            }
          ]
        },
        "options": {}
      },
      "id": "af4e71fd-33b5-402b-bf66-27cb0d172d71",
      "name": "On form submission",
      "type": "n8n-nodes-base.formTrigger",
      "typeVersion": 2.6,
      "position": [
        -4208,
        1952
      ],
      "webhookId": "805d8581-1ca4-4bcb-8529-937f30b1fa63"
    },
    {
      "parameters": {
        "compare": "selectedFields",
        "fieldsToCompare": "title",
        "options": {}
      },
      "id": "fc8ced8b-02d6-43ae-9680-1bf2722f6bdc",
      "name": "Remove Duplicates",
      "type": "n8n-nodes-base.removeDuplicates",
      "typeVersion": 2,
      "position": [
        -3760,
        1952
      ]
    },
    {
      "parameters": {
        "amount": 15
      },
      "id": "7f71fc91-a53f-40a4-b1d5-18315d6b2196",
      "name": "Wait",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1.1,
      "position": [
        -3312,
        1872
      ],
      "webhookId": "ebd4c24e-0347-4e9f-bc57-4a994f41ec8e"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=You are an expert B2B Sales Consultant analyzing local business leads for automation/marketing services.\n\nBusiness:\n{{$json.title}}\n\nCategory:\n{{$json.categoryName}}\n\nWebsite:\n{{$json.website}}\n\nRating:\n{{$json.totalScore}}\n\nReviews Count:\n{{$json.reviewsCount}}\n\nAddress:\n{{$json.address}}\n\nAnalyze this business carefully based on the following criteria:\n- Rating and review count (higher rating + more reviews = stronger, more established business)\n- Website presence (no website or a poor website = strong opportunity for web/automation services)\n- Category relevance (some categories have higher automation/marketing needs than others)\n\nScoring rules:\n- qualified: true only if rating >= 4.0 AND reviewsCount >= 10, otherwise false\n- lead_score: an integer from 0-100. Base it on rating (out of 40 points), reviewsCount (out of 30 points), and website absence/quality (out of 30 points, no website = full 30 points opportunity)\n- priority: \"High\" if lead_score >= 80, \"Medium\" if lead_score is 50-79, \"Low\" if lead_score is below 50\n- reason: one specific sentence explaining why this score was given, referencing actual numbers (rating, reviews, website status)\n- recommended_service: one specific, concrete service this business would benefit from (e.g. \"Google Ads Setup\", \"Website Redesign\", \"Automated Review Management\", \"CRM Setup for Lead Follow-up\")\n\nReturn ONLY this JSON, with no extra text, no markdown formatting, no explanation outside the JSON:\n\n{\n  \"business_name\": \"{{$json.title}}\",\n  \"website\": \"{{$json.website}}\",\n  \"address\": \"{{$json.address}}\",\n  \"rating\": {{$json.totalScore}},\n  \"qualified\": true or false based on the rules above,\n  \"lead_score\": integer 0-100 based on the rules above,\n  \"priority\": \"High\" or \"Medium\" or \"Low\" based on the rules above,\n  \"reason\": \"specific one-sentence reason\",\n  \"recommended_service\": \"specific recommended service\"\n}",
        "hasOutputParser": true,
        "messages": {
          "messageValues": []
        },
        "batching": {}
      },
      "id": "ace74196-2462-4cc6-9603-fbbf1bb1f34a",
      "name": "Basic LLM Chain",
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.9,
      "position": [
        -3088,
        1952
      ]
    },
    {
      "parameters": {
        "schemaType": "manual",
        "inputSchema": "{\n  \"type\": \"object\",\n  \"properties\": {\n    \"business_name\": {\n      \"type\": \"string\"\n    },\n    \"website\": {\n  \"type\": [\"string\", \"null\"]\n},\n    \"address\": {\n      \"type\": \"string\"\n    },\n    \"rating\": {\n      \"type\": \"number\"\n    },\n    \"qualified\": {\n      \"type\": \"boolean\"\n    },\n    \"lead_score\": {\n      \"type\": \"integer\",\n      \"minimum\": 0,\n      \"maximum\": 100\n    },\n    \"priority\": {\n      \"type\": \"string\",\n      \"enum\": [\n        \"High\",\n        \"Medium\",\n        \"Low\"\n      ]\n    },\n    \"reason\": {\n      \"type\": \"string\"\n    },\n    \"recommended_service\": {\n      \"type\": \"string\"\n    }\n  },\n  \"required\": [\n    \"business_name\",\n    \"website\",\n    \"address\",\n    \"rating\",\n    \"qualified\",\n    \"lead_score\",\n    \"priority\",\n    \"reason\",\n    \"recommended_service\"\n  ]\n}"
      },
      "id": "2763331e-ea99-4543-bd47-095569995c2e",
      "name": "Structured Output Parser",
      "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
      "typeVersion": 1.3,
      "position": [
        -2952,
        2168
      ]
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 3
          },
          "conditions": [
            {
              "id": "2c622b38-8069-4524-b4c6-961e4ab5a695",
              "leftValue": "={{ $json.output.lead_score }}",
              "rightValue": 79,
              "operator": {
                "type": "number",
                "operation": "gt"
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "id": "47a91800-3ace-4dab-900e-3e7095fd3a2a",
      "name": "If",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [
        -2736,
        1952
      ]
    },
    {
      "parameters": {
        "resource": "spreadsheet",
        "title": "Google Maps lead collection",
        "sheetsUi": {
          "sheetValues": [
            {
              "title": "sheet1"
            }
          ]
        },
        "options": {}
      },
      "id": "47fefd7b-8488-444a-9290-3a3af0f16d2f",
      "name": "Save Leads to Sheet",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 4.7,
      "position": [
        -976,
        1936
      ],
      "alwaysOutputData": true,
      "credentials": {
        "googleSheetsOAuth2Api": {
          "id": "xvWShVwWiGZH4ARl",
          "name": "Google Sheets account"
        }
      }
    },
    {
      "parameters": {
        "options": {}
      },
      "id": "b2deb870-8968-4e4e-bae7-57b11d091f04",
      "name": "Loop Over Items",
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 3,
      "position": [
        -3536,
        1952
      ]
    },
    {
      "parameters": {
        "model": "llama-3.1-8b-instant",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatGroq",
      "typeVersion": 1,
      "position": [
        -3080,
        2168
      ],
      "id": "39816420-a9cc-4b5f-9aca-91b2216caad9",
      "name": "Groq Chat Model",
      "credentials": {
        "groqApi": {
          "id": "Oseiz5ilHK9EpYgR",
          "name": "Groq account"
        }
      }
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=You are an expert B2B sales copywriter. Write a professional cold email in HTML format.\n\nBusiness Name: {{ $('If').item.json.output.business_name }}\nWebsite: {{ $('If').item.json.output.website }}\nAddress: {{ $('If').item.json.output.address }}\nRating: {{ $('If').item.json.output.rating }}\nLead Score: {{ $('If').item.json.output.lead_score }}\nRecommended Service: {{ $('If').item.json.output.recommended_service }}\n\nWebsite Audit Findings:\n{{ $json.audit ? $json.audit.audit_summary : \"No website found — this is itself the biggest opportunity.\" }}\n- Clear CTA present: {{ $json.audit.has_clear_cta }}\n- Trust signals present: {{ $json.audit.trust_signals_present }}\n- Audit summary: {{ $json.audit.audit_summary }}\n\nSEO Findings:\n- SEO summary: {{ $json.seo.seo_summary }}\n\nSocial Media Findings:\n- Social summary: {{ $json.social.social_summary }}\n\nGreeting Name: {{ $json.decision_maker.people[0]?.name ? $json.decision_maker.people[0].name.split(' ')[0] : \"there\" }}\n\nRules:\n- Start the email with \"Dear {{ $json.decision_maker.people[0]?.name ? $json.decision_maker.people[0].name.split(' ')[0] : \"there\" }},\"\n- Do NOT mention or request any email address inside the email body\n- End with a simple CTA like \"Reply to this email to learn more\" — do not use a placeholder link",
        "batching": {}
      },
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.9,
      "position": [
        -1120,
        1472
      ],
      "id": "db73b662-4045-45cb-9334-25b08b085bec",
      "name": "Generate Email content"
    },
    {
      "parameters": {
        "model": "llama-3.3-70b-versatile",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatGroq",
      "typeVersion": 1,
      "position": [
        -1048,
        1688
      ],
      "id": "2c892c80-a644-481c-a8f8-c7946cb00eaf",
      "name": "Groq Chat Model1",
      "credentials": {
        "groqApi": {
          "id": "Oseiz5ilHK9EpYgR",
          "name": "Groq account"
        }
      }
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.apify.com/v2/actors/compass~crawler-google-places/run-sync-get-dataset-items",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpQueryAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"searchStringsArray\": [\"{{ $json['Business types'] }}\"],\n  \"locationQuery\": \"{{ $json['Location'] }}\",\n  \"maxCrawledPlacesPerSearch\": {{ Number($json['Number of leads']) }}\n}",
        "options": {}
      },
      "id": "fe3b18c2-ddd1-4aaf-b18a-b54c47fd63e1",
      "name": "Google Maps scraper (appify)",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.4,
      "position": [
        -3984,
        1952
      ],
      "credentials": {
        "httpQueryAuth": {
          "id": "iqyxA0P3ipMRKrBt",
          "name": "Query Auth account"
        }
      }
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.firecrawl.dev/v2/scrape",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": []
        },
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"url\": \"{{ $json.output.website }}\",\n  \"formats\": [\"markdown\"],\n  \"onlyMainContent\": true\n}",
        "options": {
          "timeout": 6000
        }
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.5,
      "position": [
        -2288,
        1472
      ],
      "id": "b8df516d-5845-41c6-bda7-86cea63a9ed0",
      "name": "Firecrawl Scrape Website",
      "credentials": {
        "httpHeaderAuth": {
          "id": "QmgCwnGsqG6CYNnw",
          "name": "Header Auth account"
        }
      },
      "onError": "continueErrorOutput"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=You are a website conversion expert. Analyze this website content and give a technical/business audit.\n\nWebsite Content:\n{{ $json.data?.markdown || \"Website content could not be retrieved.\" }}\n\nReturn ONLY this JSON:\n{\n  \"has_clear_cta\": true or false,\n  \"trust_signals_present\": true or false (testimonials, certifications, contact info),\n  \"mobile_friendly_indicators\": true or false,\n  \"audit_score\": integer 0-100,\n  \"audit_summary\": \"one sentence summary of biggest weakness\"\n}",
        "hasOutputParser": true,
        "batching": {}
      },
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.9,
      "position": [
        -2032,
        496
      ],
      "id": "3b59eb2b-b464-4915-b438-9dd1ae5854c0",
      "name": "Website Auditor LLM chain"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatCohere",
      "typeVersion": 1,
      "position": [
        -2056,
        728
      ],
      "id": "1db01de9-4d00-4e38-8b25-e3bc354548b7",
      "name": "Cohere Chat Model",
      "credentials": {
        "cohereApi": {
          "id": "fbaIDQqzXT88fc6G",
          "name": "Cohere account"
        }
      }
    },
    {
      "parameters": {
        "schemaType": "manual",
        "inputSchema": "{\n  \"type\": \"object\",\n  \"properties\": {\n    \"has_clear_cta\": {\n      \"type\": \"boolean\"\n    },\n    \"trust_signals_present\": {\n      \"type\": \"boolean\"\n    },\n    \"mobile_friendly_indicators\": {\n      \"type\": \"boolean\"\n    },\n    \"audit_score\": {\n      \"type\": \"integer\",\n      \"minimum\": 0,\n      \"maximum\": 100\n    },\n    \"audit_summary\": {\n      \"type\": \"string\"\n    }\n  },\n  \"required\": [\n    \"has_clear_cta\",\n    \"trust_signals_present\",\n    \"mobile_friendly_indicators\",\n    \"audit_score\",\n    \"audit_summary\"\n  ]\n}",
        "autoFix": true
      },
      "id": "ab6a7485-b24d-4a11-97dd-43b926f7ed66",
      "name": "Structured Output Parser1",
      "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
      "typeVersion": 1.3,
      "position": [
        -1928,
        728
      ]
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=You are an SEO expert. Analyze this website content for SEO quality.\n\nWebsite Content:\n{{ $json.data?.markdown || \"Website content could not be retrieved.\" }}\n\nReturn ONLY this JSON:\n{\n  \"has_meta_description\": true or false,\n  \"keyword_focus\": \"main keyword/topic detected\",\n  \"content_depth\": \"Thin\" or \"Moderate\" or \"Strong\",\n  \"seo_score\": integer 0-100,\n  \"seo_summary\": \"one sentence biggest SEO gap\"\n}",
        "hasOutputParser": true,
        "batching": {}
      },
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.9,
      "position": [
        -2032,
        1104
      ],
      "id": "3ab0edb5-f05a-452b-b2dd-4b6e167ff593",
      "name": "SEO Analysis LLM Chain"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatCohere",
      "typeVersion": 1,
      "position": [
        -1960,
        1544
      ],
      "id": "6147abb0-6e7a-49bd-b196-8dd924853332",
      "name": "Cohere Chat Model1",
      "credentials": {
        "cohereApi": {
          "id": "fbaIDQqzXT88fc6G",
          "name": "Cohere account"
        }
      }
    },
    {
      "parameters": {
        "schemaType": "manual",
        "inputSchema": "{\n  \"type\": \"object\",\n  \"properties\": {\n    \"has_meta_description\": {\n      \"type\": \"boolean\"\n    },\n    \"keyword_focus\": {\n      \"type\": \"string\"\n    },\n    \"content_depth\": {\n      \"type\": \"string\",\n      \"enum\": [\"Thin\", \"Moderate\", \"Strong\"]\n    },\n    \"seo_score\": {\n      \"type\": \"integer\",\n      \"minimum\": 0,\n      \"maximum\": 100\n    },\n    \"seo_summary\": {\n      \"type\": \"string\"\n    }\n  },\n  \"required\": [\n    \"has_meta_description\",\n    \"keyword_focus\",\n    \"content_depth\",\n    \"seo_score\",\n    \"seo_summary\"\n  ]\n}",
        "autoFix": true
      },
      "id": "ece36ca5-b008-4b19-95b1-51c8e9baa16d",
      "name": "Structured Output Parser2",
      "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
      "typeVersion": 1.3,
      "position": [
        -1960,
        1336
      ]
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatCohere",
      "typeVersion": 1,
      "position": [
        -1904,
        2736
      ],
      "id": "6d287164-0cf8-47c6-8ff9-7dc49b9b7b0e",
      "name": "Cohere Chat Model2",
      "credentials": {
        "cohereApi": {
          "id": "fbaIDQqzXT88fc6G",
          "name": "Cohere account"
        }
      }
    },
    {
      "parameters": {
        "schemaType": "manual",
        "inputSchema": "{\n  \"type\": \"object\",\n  \"properties\": {\n    \"decision_makers_found\": {\n      \"type\": \"boolean\"\n    },\n    \"people\": {\n      \"type\": \"array\",\n      \"items\": {\n        \"type\": \"object\",\n        \"properties\": {\n          \"name\": { \"type\": \"string\" },\n          \"title\": { \"type\": \"string\" },\n          \"email\": { \"type\": [\"string\", \"null\"] },\n          \"linkedin_url\": { \"type\": [\"string\", \"null\"] },\n          \"source_snippet\": { \"type\": \"string\" }\n        },\n        \"required\": [\"name\", \"title\", \"email\", \"linkedin_url\", \"source_snippet\"]\n      }\n    }\n  },\n  \"required\": [\"decision_makers_found\", \"people\"]\n}",
        "autoFix": true
      },
      "id": "a8d99005-189d-477f-9329-6ef37a5d830c",
      "name": "Structured Output Parser3",
      "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
      "typeVersion": 1.3,
      "position": [
        -1904,
        2528
      ]
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=You are an expert B2B decision-maker researcher.\n\nAnalyze the website content below and identify publicly available decision-maker information.\n\nLook for:\n- Founder\n- Co-Founder\n- Owner\n- CEO\n- President\n- Managing Director\n- Director\n- Marketing Director\n- General Manager\n- Principal\n- Partner\n- Senior leadership\n\nWebsite Content:\n{{ $json.data?.markdown || \"Website content could not be retrieved.\" }}\nRules:\n1. Only use information actually present in the website content.\n2. Never invent a person's name, job title, email, LinkedIn URL, or other information.\n3. If multiple people are found, list up to 3.\n4. If no person is found, return an empty array.\n5. Return ONLY the structured output.\nFor each person found, extract:\n- name\n- title\n- email (only if explicitly written on the page, otherwise null)\n- linkedin_url (only if explicitly written on the page, otherwise null)\n- source_snippet (the exact sentence/phrase where this was found)\n\nReturn ONLY this JSON:\n{\n  \"decision_makers_found\": true or false,\n  \"people\": [\n    {\n      \"name\": \"string\",\n      \"title\": \"string\",\n      \"email\": \"string or null\",\n      \"linkedin_url\": \"string or null\"\n    }\n  ]\n}",
        "hasOutputParser": true,
        "batching": {}
      },
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.9,
      "position": [
        -1984,
        2288
      ],
      "id": "279c1dfe-497d-40a5-beee-d8a8aee41baf",
      "name": "Decision Maker Finder"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatCohere",
      "typeVersion": 1,
      "position": [
        -1960,
        2152
      ],
      "id": "0fbf201a-4f57-4a24-9af7-ac879cefb304",
      "name": "Cohere Chat Model3",
      "credentials": {
        "cohereApi": {
          "id": "fbaIDQqzXT88fc6G",
          "name": "Cohere account"
        }
      }
    },
    {
      "parameters": {
        "schemaType": "manual",
        "inputSchema": "{\n  \"type\": \"object\",\n  \"properties\": {\n    \"social_links_found\": {\n      \"type\": \"array\",\n      \"items\": {\n        \"type\": \"string\"\n      }\n    },\n    \"social_presence_score\": {\n      \"type\": \"integer\",\n      \"minimum\": 0,\n      \"maximum\": 100\n    },\n    \"social_summary\": {\n      \"type\": \"string\"\n    }\n  },\n  \"required\": [\n    \"social_links_found\",\n    \"social_presence_score\",\n    \"social_summary\"\n  ]\n}",
        "autoFix": true
      },
      "id": "47f91053-668c-4310-996d-ee564a52da85",
      "name": "Structured Output Parser4",
      "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
      "typeVersion": 1.3,
      "position": [
        -1960,
        1944
      ]
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=You are a digital marketing analyst. Check this website content for social media presence signals.\n\nWebsite Content:\n{{ $json.data?.markdown || \"Website content could not be retrieved.\" }}\n\nReturn ONLY this JSON:\n{\n  \"social_links_found\": [\"list any social media URLs found, empty array if none\"],\n  \"social_presence_score\": integer 0-100,\n  \"social_summary\": \"one sentence about their social media gap or strength\"\n}",
        "hasOutputParser": true,
        "batching": {}
      },
      "type": "@n8n/n8n-nodes-langchain.chainLlm",
      "typeVersion": 1.9,
      "position": [
        -2032,
        1712
      ],
      "id": "89da7571-5fc9-4382-8e71-3a3f3fce10a5",
      "name": "Social Media presence check"
    },
    {
      "parameters": {
        "mode": "combine",
        "combineBy": "combineByPosition",
        "numberInputs": 4,
        "options": {}
      },
      "type": "n8n-nodes-base.merge",
      "typeVersion": 3.2,
      "position": [
        -1344,
        1584
      ],
      "id": "537c3bc9-91ac-4fbd-bd85-cd675daf95a9",
      "name": "Merge"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatCohere",
      "typeVersion": 1,
      "position": [
        -1848,
        936
      ],
      "id": "53d37617-d4c1-4234-b25b-075b3c39460a",
      "name": "Cohere Chat Model4",
      "credentials": {
        "cohereApi": {
          "id": "fbaIDQqzXT88fc6G",
          "name": "Cohere account"
        }
      }
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "fe7b7b36-b1ba-4921-b2f5-ff818db259c5",
              "name": "audit",
              "value": "={{ $json.output }}",
              "type": "object"
            },
            {
              "id": "4d9033a9-7601-45ff-b5e3-b182a9d52445",
              "name": "",
              "value": "",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        -1568,
        704
      ],
      "id": "48bfd7b5-bfb0-47e9-8a67-3870dfa7524c",
      "name": "Rename Audit"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "6c0b31c3-110f-471c-a9b1-d625db6bd232",
              "name": "seo",
              "value": "={{ $json.output }}",
              "type": "object"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        -1568,
        1312
      ],
      "id": "c1ad2b18-3a00-4120-b6d6-f53e5ad1e8b4",
      "name": "Rename SEO"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "e3b26d96-21bc-44dd-a0d7-37d318c896bf",
              "name": "social",
              "value": "={{ $json.output }}",
              "type": "object"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        -1568,
        1920
      ],
      "id": "942058ab-f96e-401f-817a-71a8508f8d01",
      "name": "Rename Social"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "e3b26d96-21bc-44dd-a0d7-37d318c896bf",
              "name": "decision_maker",
              "value": "={{ $json.output }}",
              "type": "object"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        -1520,
        2288
      ],
      "id": "3e1453be-57ac-4c01-ad53-353b23e4140b",
      "name": "Rename Decision Maker"
    },
    {
      "parameters": {
        "sendTo": "mdhasan27536459k@gmail.com",
        "subject": "={{ \"🔥 High Quality Lead - \" + $('If').item.json.output.business_name }}",
        "message": "={{ $json.text }}",
        "options": {
          "appendAttribution": false
        }
      },
      "type": "n8n-nodes-base.gmail",
      "typeVersion": 2.2,
      "position": [
        -768,
        1472
      ],
      "id": "8e30813c-58d9-465e-b0c4-deccfe76688a",
      "name": "Send a message",
      "webhookId": "139d4eed-67e1-4031-bc12-add1ea044f41",
      "credentials": {
        "gmailOAuth2": {
          "id": "lpImydvEbU6kbMRA",
          "name": "Gmail account"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 3
          },
          "conditions": [
            {
              "id": "e998ddde-8288-4c4d-9962-be1c95432bc8",
              "leftValue": "=website",
              "rightValue": "={{ $json.output.website }} is not empty",
              "operator": {
                "type": "string",
                "operation": "notEmpty",
                "singleValue": true
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.3,
      "position": [
        -2512,
        1952
      ],
      "id": "5a4e6f7a-7cf8-4808-a0f2-50d97a169833",
      "name": "Has Website?"
    }
  ],
  "pinData": {},
  "connections": {
    "On form submission": {
      "main": [
        [
          {
            "node": "Google Maps scraper (appify)",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Remove Duplicates": {
      "main": [
        [
          {
            "node": "Loop Over Items",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait": {
      "main": [
        [
          {
            "node": "Basic LLM Chain",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Basic LLM Chain": {
      "main": [
        [
          {
            "node": "If",
            "type": "main",
            "index": 0
          },
          {
            "node": "Loop Over Items",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Structured Output Parser": {
      "ai_outputParser": [
        [
          {
            "node": "Basic LLM Chain",
            "type": "ai_outputParser",
            "index": 0
          }
        ]
      ]
    },
    "If": {
      "main": [
        [
          {
            "node": "Has Website?",
            "type": "main",
            "index": 0
          }
        ],
        []
      ]
    },
    "Loop Over Items": {
      "main": [
        [],
        [
          {
            "node": "Wait",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Groq Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "Basic LLM Chain",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Generate Email content": {
      "main": [
        [
          {
            "node": "Send a message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Groq Chat Model1": {
      "ai_languageModel": [
        [
          {
            "node": "Generate Email content",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Google Maps scraper (appify)": {
      "main": [
        [
          {
            "node": "Remove Duplicates",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Firecrawl Scrape Website": {
      "main": [
        [
          {
            "node": "Website Auditor LLM chain",
            "type": "main",
            "index": 0
          },
          {
            "node": "SEO Analysis LLM Chain",
            "type": "main",
            "index": 0
          },
          {
            "node": "Social Media presence check",
            "type": "main",
            "index": 0
          },
          {
            "node": "Decision Maker Finder",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Cohere Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "Website Auditor LLM chain",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Structured Output Parser1": {
      "ai_outputParser": [
        [
          {
            "node": "Website Auditor LLM chain",
            "type": "ai_outputParser",
            "index": 0
          }
        ]
      ]
    },
    "Website Auditor LLM chain": {
      "main": [
        [
          {
            "node": "Rename Audit",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Cohere Chat Model1": {
      "ai_languageModel": [
        [
          {
            "node": "SEO Analysis LLM Chain",
            "type": "ai_languageModel",
            "index": 0
          },
          {
            "node": "Structured Output Parser2",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Structured Output Parser2": {
      "ai_outputParser": [
        [
          {
            "node": "SEO Analysis LLM Chain",
            "type": "ai_outputParser",
            "index": 0
          }
        ]
      ]
    },
    "Cohere Chat Model2": {
      "ai_languageModel": [
        [
          {
            "node": "Decision Maker Finder",
            "type": "ai_languageModel",
            "index": 0
          },
          {
            "node": "Structured Output Parser3",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Structured Output Parser3": {
      "ai_outputParser": [
        [
          {
            "node": "Decision Maker Finder",
            "type": "ai_outputParser",
            "index": 0
          }
        ]
      ]
    },
    "SEO Analysis LLM Chain": {
      "main": [
        [
          {
            "node": "Rename SEO",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Cohere Chat Model3": {
      "ai_languageModel": [
        [
          {
            "node": "Social Media presence check",
            "type": "ai_languageModel",
            "index": 0
          },
          {
            "node": "Structured Output Parser4",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Structured Output Parser4": {
      "ai_outputParser": [
        [
          {
            "node": "Social Media presence check",
            "type": "ai_outputParser",
            "index": 0
          }
        ]
      ]
    },
    "Social Media presence check": {
      "main": [
        [
          {
            "node": "Rename Social",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Cohere Chat Model4": {
      "ai_languageModel": [
        [
          {
            "node": "Structured Output Parser1",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Rename Audit": {
      "main": [
        [
          {
            "node": "Merge",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Rename SEO": {
      "main": [
        [
          {
            "node": "Merge",
            "type": "main",
            "index": 1
          }
        ]
      ]
    },
    "Rename Social": {
      "main": [
        [
          {
            "node": "Merge",
            "type": "main",
            "index": 2
          }
        ]
      ]
    },
    "Merge": {
      "main": [
        [
          {
            "node": "Generate Email content",
            "type": "main",
            "index": 0
          },
          {
            "node": "Save Leads to Sheet",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Decision Maker Finder": {
      "main": [
        [
          {
            "node": "Rename Decision Maker",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Rename Decision Maker": {
      "main": [
        [
          {
            "node": "Merge",
            "type": "main",
            "index": 3
          }
        ]
      ]
    },
    "Has Website?": {
      "main": [
        [
          {
            "node": "Firecrawl Scrape Website",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Generate Email content",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate",
    "availableInMCP": true
  },
  "versionId": "d8cd1a3c-0778-4dd8-96da-e229177ec169",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "7263593850bbed7684cd55923f523248c54c011801142265c8cf49d8ed05bd0d"
  },
  "nodeGroups": [],
  "id": "ZpkrYP1kjwOvcwDe",
  "tags": []
}
