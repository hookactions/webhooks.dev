---
title: "Authentication"
date: 2019-07-03T22:26:48+02:00
draft: false
weight: 100
---

Unlike normal API authentication, webhooks usually use an HMAC signature to verify the webhook is coming from who you think it is.

## HMAC

HMAC stands for _hash-based message authentication code_. This involves a cryptographic hash function such as SHA-1 and a secret key. If you are integrating webhooks into your application there are a few things to look for.

1. Hash function – The most common are MD5, SHA-1, and SHA-256. There was an [RFC](https://tools.ietf.org/html/rfc6151) published in 2011 which talks about the security of using MD5 as a hash function for HMAC signatures. It said that while MD5 itself is not secure, there doesn't appear to be any practical vulnerability when used as an HMAC hash function. That being said, when creating new webhooks it's advised to at least use SHA-1.

2. Secret key – The best way to ensure your HMAC cannot be brute forced is by using a cryptographically secure secret key. This means that using `1d1d3c2e-9dd2-11e9-a7ca-3c15c2eb7774` as a secret key instead of `s3cr3t` is more secure. If you are not able to choose a custom secret key and the key you are provided is too short, you should ask the provider if they can make it more secure. A good rule of thumb is to have a key with at least 32 bytes (256 bits). This will be secure enough for use with the SHA-256 hash function, read more about it [here](https://crypto.stackexchange.com/a/34866).

If you are developing webhooks for your application to send to your users, it's best to use at least SHA-1 or SHA-256 for the hash function and allow users to generate their own secret key. Or if you generate the key for them, make sure there is a way to regenerate it in case their key is compromised.

## Code

### Python

```python
import hmac
from hashlib import md5, sha1, sha256
from functools import partial


def compare_signature(digestmod, key: bytes, message: bytes, expected_signature: str) -> bool:
    mac = hmac.new(key, message, digestmod)
    return mac.hexdigest() == expected_signature


compare_md5_signature = partial(compare_signature, md5)
compare_sha1_signature = partial(compare_signature, sha1)
compare_sha256_signature = partial(compare_signature, sha256)
```

It's important to note that the key and message **must** be `bytes`. `mac.hexdigest()` will return a hexadecimal representation of the signature like `aa52ae4fa3fd04ec91bec056f2cd426b` which is generally how third parties provide the signature.

#### Django

```python
HEADER_NAME = "X-WEBHOOK-SIGNATURE"
# CHANGE: use your own unique secret!
WEBHOOK_SECRET = b"443c9cec-9dd6-11e9-b610-3c15c2eb7774"


class MyWehook(View):
    def get(self, request, *args, **kwargs):
        expected_sig = request.META.get(HEADER_NAME)
        # CHANGE: compare_sha1_signature for the signature your integration uses
        if not compare_sha1_signature(WEBHOOK_SECRET, request.body, expected_sig):
            return HttpResponseForbidden()
        # ... process data
        return JsonResponse({})
```

#### Flask

```python
from flask import request, abort


HEADER_NAME = "X-WEBHOOK-SIGNATURE"
# CHANGE: use your own unique secret!
WEBHOOK_SECRET = b"443c9cec-9dd6-11e9-b610-3c15c2eb7774"


@app.route('/my-webhook')
def my_webhook():
    expected_sig = request.headers.get(HEADER_NAME)
    # CHANGE: compare_sha1_signature for the signature your integration uses
    if not compare_sha1_signature(WEBHOOK_SECRET, request.data.encode(), expected_sig):
        abort(403)
    # ... process data
    return 'ok'
```

## Appendix I - Authorization

There is a difference between authentication and authorization. Authentication is "I am this person", whereas authorization is "I have access to this item". When working with webhooks, it can be tempting to assume that since our application is using HMAC signature verification that it will never receive malicious data. It's important to think defensively in regards to webhooks.

When receiving a webhook it's best to assume there is no "authenticated" nor "authorized" user, and treat the incoming data as malicious until proven otherwise. When using other best practices for web applications to avoid things like SQL injection, this is easier but still important to be aware of as you develop your integration.
