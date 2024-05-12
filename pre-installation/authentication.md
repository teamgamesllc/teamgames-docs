---
description: >-
  Welcome to the Authentication section of our documentation. This page provides
  comprehensive details on how to securely authenticate with our API using API
  keys. You’ll find step-by-step instructions
---

# Authentication

## Authentication

### Overview

Our system uses API keys to authenticate requests made to our servers. This document outlines how to use your API key to make authenticated requests and provides details about security practices to ensure safe transmission of sensitive data.

### Obtaining an API Key

To interact with our API, you need a valid API key. This key uniquely identifies your application and ensures that calls made to our services are permitted. To obtain an API key:

1. Log in to your account on our platform.
2. Navigate to the account dashboard.

### Using Your API Key

Your API key must be included in every API call made to our servers. It should be included in the HTTPS header of your requests. Here is how to include it:

#### Example of Adding API Key to HTTP Header



{% tabs %}
{% tab title="CURL" %}
```
bash
curl -X POST "http://api.teamgames.io/v3/example-endpoint" \
     -H "Authorization: Bearer YOUR_API_KEY_HERE" \
     -H "Content-Type: application/json" \
     -d '{"param1":"value1", "param2":"value2"}'
```
{% endtab %}

{% tab title="Java" %}
```
HttpsURLConnection conn = (HttpsURLConnection) url.openConnection();
String encodedKey = Base64.getEncoder().encodeToString((apiKey).getBytes("UTF-8"));
conn.setRequestMethod("POST");
conn.setRequestProperty("Authorization", "Bearer " + encodedKey);
```
{% endtab %}
{% endtabs %}

