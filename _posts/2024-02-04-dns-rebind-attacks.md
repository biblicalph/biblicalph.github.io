---
layout: post
title: "DNS Rebind Attack"
author: "Kwabena Boadu"
categories: journal
tags: [dns,ssrf,rebind attack]
---

DNS rebinding is an example of a blind server side request forgery vulnerability.

Blind Server Side Request Forgery (SSRF) vulnerabilities arise when an application can be induced into issuing a back-end HTTP request to a supplied URL, but the response from the back-end is not returned in the application's front-end response. Source [portswigger](https://portswigger.net/web-security/ssrf/blind).

Applications that accept URLs from users and make calls to them may be susceptible to DNS rebinding.
Example: a payment application that notifies a client via webhook of payment transactions.

## How is DNS rebinding attack executed?

Let's use a fictitious payment application, call it RebindPay, as a case study here.

1. A client of RebindPay must provide a webhook URL to post notifications to.
2. The client provides a URL with a very low time-to-live (TTL). Example: https://myhost.mydomain.com/resources. The domain `myhost.domain.com` first resolves to an external IP address `51.195.136.169` with a TTL of 5 seconds. After 5 seconds, the URL resolves to `127.0.0.1`, an internal/local IP address
3. The RebindPay app validates the provided URL by verifying the domain resolves to an external IP address. The validation passes and the URL is saved
4. A payment transaction occurs and the RebindPay app makes a request to notify the client
5. The client domain, `myhost.domain.com`, now resolves to `127.0.0.1`, an internal IP addresss
6. The RebindPay app makes a request to the resolved IP, `127.0.0.1/resources`

## Consequences of rebind attacks
 
Attackers can use DNS rebinding to attack internal networks, potentially forcing requests to memory or CPU intensive internal addresses

## How to prevent DNS rebinding?

1. Reject domain resolutions with low TTL, say below *30 seconds*
2. Validate all IP resolutions, verifying that the resolve to external address. In the above described scenario, validation must occur before data is posted to the URL

## Final thoughts

Stay safe and build secure applications