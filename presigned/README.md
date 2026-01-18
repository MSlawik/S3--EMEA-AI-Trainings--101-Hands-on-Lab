# 8. Pre-Signed URLs

## 8.0 Pain points
- to control from a central point pre-signed URLs
- to identify abuse of non authenticated requests


## 8.1 What is an S3 pre-signed URL?
A pre-signed URL is meant to be easily shared and available for a limited amount of time. Think about it like a file transfert link without the need of sharing an access key and a secret key that you share with an external user for a couple of days. This link would be available to GET (download) a specific file or PUT a file (upload) to a bucket.
After the admin specified expiration date, the link is not available anymore.


a pre-signed URLcan be created for GET requests
```shell
mc share download minio-101/bucket1/grades.pdf --expire 7d
```

The curl command provided can be used in a script, to a user or just share the hashed header in a GET request code but it will be valid only for the specified amount of time.
Remember, attackers can host malwares or overwrite existing files with malicious contents.

```shell
curl -X PUT --data "malware" https://minio-101/bucket1/update.bin
```

It can also be created for PUT requests
```shell
mc share upload minio-101/bucket1/grades.pdf --expire 10m
```

## 8.2 Risks of Pre-Signed URLs
pre-signed URLs have several inherent risks:
- they don't have any size limit
- the expiration date is long
- there is no content validation
- they are oftenly shared with less caution thans sharing credentials or TLS keys because there are usually no formal processes so they are shared with less precautions.


## 8.3 Blocking or limiting Pre-Signed URLs using BIG-IP iRules

### 8.3.1 Creating an iRule

The following iRule blocks **SigV4 pre-signed URLs** for a **configurable list of S3 buckets**, while allowing:
- Normal authenticated S3 requests (Authorization header)
- Non-pre-signed access to public buckets

<br>
<br>
---

### 8.3.2 How pre-signed URLs are detected

A SigV4 pre-signed URL always contains one or more of the following query parameters:

- `X-Amz-Signature`
- `X-Amz-Credential`
- `X-Amz-Expires`

The iRule checks for these parameters to reliably identify pre-signed requests.

<br>
<br>
---

### 8.3.3 Prerequisites: 

First, we need to create a **string data group** that will contain the list of buckets where we want to deny pre-signed access:

- **Name:** `dg_presign_block_buckets`
- **Type:** String
- **Entries (keys only):**
  ```text
  finance
  backups
  private-data
  logs


```tcl
when HTTP_REQUEST {
    # Fast exit if no query string
    if {[HTTP::query] eq ""} {
        return
    }

    # Detect pre-signed URL (SigV4)
    if { !(
        [HTTP::query] contains "X-Amz-Signature=" ||
        [HTTP::query] contains "X-Amz-Credential=" ||
        [HTTP::query] contains "X-Amz-Expires="
    ) } {
        return
    }

    # -------------------------------
    # Extract bucket name
    # -------------------------------

    set bucket ""

    # Case 1: virtual-host style
    #   bucket.s3.example.com
    set host [string tolower [HTTP::host]]
    if {[regexp {^([a-z0-9\-]+)\.} $host -> bucket]} {
        # bucket extracted
    } else {
        # Case 2: path-style
        #   /bucket/object
        if {[regexp {^/([^/]+)/} [HTTP::path] -> bucket]} {
            # bucket extracted
        }
    }

    # If bucket still unknown, allow
    if {$bucket eq ""} {
        return
    }

    # -------------------------------
    # Enforce deny list
    # -------------------------------

    if {[class match $bucket equals dg_presign_block_buckets]} {

        log local0.warn "Blocked pre-signed URL for bucket=$bucket from [IP::client_addr]"

        HTTP::respond 403 \
            content "Pre-signed URLs are not allowed for this bucket." \
            "Content-Type" "text/plain"

        return
    }
}
```

## 8.4 Key takeaways
S3 Pre-signed URLs are a great feature that are very convenient for time limited access to a bucket or a specific object. But all great powers should be used carefully! Remember pre-signed URLs are non authenticated endpoints and could be abused, leaked, replayed, or over-privileged.

[Back to Agenda](/README.md)
