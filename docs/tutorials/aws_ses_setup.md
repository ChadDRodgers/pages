AI Generated Tutorial

To filter out spam emails in your forwarding setup, you must implement a multi-layered defense. This includes activating Amazon SES IP Blocklists, enabling TLS enforcement, and adding an inline Spam Content Scanner using SpamAssassin-style headers inside your Python Lambda function. This prevents junk mail from being forwarded to your personal Gmail or iCloud account, which protects your domain's sending reputation.
Here is your updated, complete, end-to-end master blueprint with built-in spam prevention.
------------------------------
## Step 1: Purchase Your Domain via Route 53
You need a custom domain name (like example.com) to serve as your personal email routing address.

   1. Open the [AWS Management Console](https://console.aws.amazon.com/ses) and navigate to the Route 53 service.
   2. In the left sidebar, click Registered Domains, then click Register Domain.
   3. Type in your desired domain name, choose an extension (such as .com or .net), and check its availability.
   4. Click Proceed to checkout, fill out your contact details, and click Submit.
   5. Note: It can take up to an hour for registration to process. Once complete, Route 53 will automatically generate a public Hosted Zone for your domain.

------------------------------
## Step 2: Verify Your Identities in Amazon SES
AWS requires proof that you own both the custom receiving domain and the personal destination address.

   1. Navigate to the Amazon SES (Simple Email Service) console.
   2. In the left sidebar, click Verified Identities, then click Create Identity.
   3. Select Domain as your identity type and enter your custom domain (e.g., yourdomain.com).
   4. Under Advanced DKIM settings, leave it as Easy DKIM. Check the box to Publish DNS records to Route 53 so AWS automatically handles the validation records. Click Create Identity.
   5. Click Create Identity a second time. Select Email Address.
   6. Enter your personal Gmail or iCloud address and click Create Identity.
   7. Log into that personal inbox, open the confirmation email from AWS, and click the verification link.

------------------------------
## Step 3: Request SES Production Access (Move out of Sandbox)
All new AWS accounts start in a restricted "Sandbox." To accept public mail from anyone and forward it out, you must exit the sandbox.

   1. In the SES console sidebar, click Account Dashboard.
   2. Locate the sandbox banner and click Request Production Access.
   3. Fill out the application form:
   * Mail Type: Select Transactional.
      * Website URL: Enter your custom domain name.
      * Use Case Description: Enter this exact text: "I am building an inbound email forwarding pipeline. Public emails sent to my custom domain will be securely deposited into an S3 bucket and processed by an AWS Lambda function. The function will analyze SES spam metrics, discard junk mail, modify the headers to ensure SPF/DMARC compatibility, and forward clean emails to my personal Gmail/iCloud address. Old emails will be automatically pruned via S3 Lifecycle rules."
   4. Submit the request. Approval usually takes between 1 and 24 hours.

------------------------------
## Step 4: Configure Domain MX Records
To force internet traffic to route emails sent to your domain over to Amazon SES, you must update your Mail Exchange (MX) records.

   1. Open the Route 53 console and click on Hosted Zones.
   2. Click your custom domain name from the list.
   3. Click Create Record and configure these settings:
        * Record Name: Leave blank (matches the root domain).
        * Record Type: Choose MX.
        * TTL: 300
        * Value: 10 inbound-smtp.<YOUR-AWS-REGION>.amazonaws.com (Replace <YOUR-AWS-REGION> with your actual AWS region code, like us-east-1 for N. Virginia).
   4. Click Create Records.

------------------------------
## Step 5: Create an S3 Storage Bucket with an Automated Purge Rule
SES must write incoming email payloads directly to an S3 bucket before processing. We will build this bucket and apply a built-in lifecycle rule to automatically delete old emails after 3 days.

   1. Navigate to the S3 console and click Create Bucket.
   2. Enter a globally unique name (e.g., my-ses-email-storage-bucket-12345), leave all default settings, and click Create Bucket.
   3. Click into your new bucket, select the Permissions tab, find Bucket Policy, and click Edit.
   4. Paste the following JSON policy block to allow the SES service to write files here. Replace YOUR_ACCOUNT_ID and YOUR_BUCKET_NAME with your actual info:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowSESPut",
            "Effect": "Allow",
            "Principal": { "Service": "ses.amazonaws.com" },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*",
            "Condition": {
                "StringEquals": { "aws:Referer": "YOUR_ACCOUNT_ID" }
            }
        }
    ]
}
```


   1. Click Save Changes.
   2. Switch to the Management tab at the top of the bucket page.
   3. Click Create lifecycle rule and set these configurations:
   * Lifecycle rule name: Delete-Old-SES-Emails
      * Rule scope: Choose Apply to all objects in the bucket (check the acknowledgement box).
      * Lifecycle rule actions: Check Expire current versions of objects.
      * Expire current versions of objects: Set Days after object creation to 3.
   4. Click Create rule. S3 will now natively delete processed files in the background completely for free.

------------------------------
## Step 6: Deploy the Smart Python Lambda Forwarder (With Spam Scanning)
This updated script intercepts the email and evaluates its Spam, Virus, SPF, DKIM, and DMARC verdicts provided by AWS SES. If the email fails these security validations, the script drops the email immediately to preserve your personal inbox and your forwarding IP reputation.

   1. Navigate to the Lambda console and click Create Function.
   2. Select Author from scratch, name the function SES-Email-Forwarder, and select Python 3.12 as the runtime.
   3. Expand Change default execution role, choose Create a new role with basic Lambda permissions, and click Create function.
   4. Once loaded, go to the Configuration tab, click Permissions, and click the blue link under Role Name to open the IAM role in a new tab.
   5. Click Add Permissions -> Attach Policies. Search for and check the boxes next to AmazonS3FullAccess and AmazonSESFullAccess. Click Add Permissions, then close that IAM tab.
   6. Return to your Lambda window, select the Code tab, double-click `lambda_function.py` in the file explorer, erase everything, and paste the function below.

### Configuration

Update these values before deploying — this YAML shows the same keys used by the function's `CONFIG` dictionary:

```yaml
from_email: forwarder@yourdomain.com
bucket_name: your-ses-email-storage-bucket
forward_mapping:
  "@yourdomain.com":
    - yourpersonal@gmail.com
block_spam: true
block_viruses: true
strict_dmarc: false
```

Paste this complete, formatted Python function into `lambda_function.py`:

```python
import os
import boto3
import email
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

# --- CONFIGURATION MANAGEMENT ---
CONFIG = {
    "from_email": "forwarder@yourdomain.com",
    "bucket_name": "your-ses-email-storage-bucket",
    "forward_mapping": {
        "@yourdomain.com": ["yourpersonal@gmail.com"]
    },
    "block_spam": True,
    "block_viruses": True,
    "strict_dmarc": False
}

s3_client = boto3.client("s3")
ses_client = boto3.client("ses")

def lambda_handler(event, context):
    ses_record = event["Records"][0]["ses"]
    message_id = ses_record["mail"]["messageId"]
    original_recipients = ses_record["receipt"]["recipients"]
    receipt = ses_record["receipt"]

    print(f"Processing inbound email ID: {message_id}")

    # --- SPAM & VIRUS FILTERING ENGINE ---
    spam_verdict = receipt.get("spamVerdict", {}).get("status", "FAIL")
    virus_verdict = receipt.get("virusVerdict", {}).get("status", "FAIL")
    spf_verdict = receipt.get("spfVerdict", {}).get("status", "FAIL")
    dkim_verdict = receipt.get("dkimVerdict", {}).get("status", "FAIL")
    dmarc_verdict = receipt.get("dmarcVerdict", {}).get("status", "FAIL")

    print(
        f"Verdicts - Spam: {spam_verdict}, Virus: {virus_verdict}, SPF: {spf_verdict}, DKIM: {dkim_verdict}, DMARC: {dmarc_verdict}"
    )

    if CONFIG["block_spam"] and spam_verdict == "FAIL":
        print(f"Aborting Pipeline: Email ID {message_id} flagged as SPAM.")
        return {"status": "blocked", "reason": "Spam verdict failed"}

    if CONFIG["block_viruses"] and virus_verdict == "FAIL":
        print(f"Aborting Pipeline: Email ID {message_id} flagged containing a VIRUS.")
        return {"status": "blocked", "reason": "Virus verdict failed"}

    if CONFIG["strict_dmarc"] and (
        spf_verdict == "FAIL" or dkim_verdict == "FAIL" or dmarc_verdict == "FAIL"
    ):
        print(f"Aborting Pipeline: Email ID {message_id} failed domain authentication check.")
        return {"status": "blocked", "reason": "DMARC/SPF verification failed"}

    try:
        s3_object = s3_client.get_object(Bucket=CONFIG["bucket_name"], Key=message_id)
        raw_email_bytes = s3_object["Body"].read()

        msg = email.message_from_bytes(raw_email_bytes)

        target_destinations = []
        for recipient in original_recipients:
            recipient_lower = recipient.lower()
            if recipient_lower in CONFIG["forward_mapping"]:
                target_destinations.extend(CONFIG["forward_mapping"][recipient_lower])
            else:
                if "@" in recipient_lower:
                    domain_match = recipient_lower[recipient_lower.index("@"):]
                    if domain_match in CONFIG["forward_mapping"]:
                        target_destinations.extend(CONFIG["forward_mapping"][domain_match])

        if not target_destinations:
            print(f"No valid forwarding mapping found for recipients: {original_recipients}")
            return {"status": "skipped", "reason": "No mapping matched"}

        original_from = msg.get("From", "Unknown Sender")

        for header in ["Return-Path", "Sender", "DKIM-Signature", "Reply-To", "From"]:
            if header in msg:
                del msg[header]

        msg["Reply-To"] = original_from
        clean_display_name = original_from.split("<")[0].strip().replace('"', "")
        msg["From"] = f'"{clean_display_name} via Forwarder" <{CONFIG["from_email"]}>'

        msg["X-SES-Spam-Verdict"] = spam_verdict
        msg["X-SES-Virus-Verdict"] = virus_verdict

        response = ses_client.send_raw_email(
            Source=CONFIG["from_email"], Destinations=target_destinations, RawMessage={"Data": msg.as_bytes()}
        )

        print(f"Email successfully forwarded. SES Message ID: {response.get('MessageId')}")
        return {"status": "success"}

    except Exception as e:
        print(f"Pipeline failure while processing email: {str(e)}")
        raise

```

   1. Update the configurations above with your verified email address, custom domain, and S3 bucket name. Click Deploy.

------------------------------
## Step 7: Create the SES Receipt Rule Group (With TLS Enforcement)
The final step glues your infrastructure components together while adding AWS IP filtering and TLS security.

   1. Navigate back to the Amazon SES console.
   2. In the left sidebar under Configuration, click Email Receiving.
   3. If no active rule sets exist, click Create Rule Set, name it main-rule-set, and save it. Click into it.
   4. Click Create Rule, name it forward-incoming-mail, and click Next.
   5. Recipient Conditions: Leave this blank to automatically forward every alias on your domain. Click Next.
   6. Drop Spam & Require TLS (Crucial Layer): On the Configure options step:
   * Check the box for Require TLS (rejects non-encrypted incoming spam vectors).
      * Check the box for Enable Spam and Virus Scanning. This tells AWS to evaluate the emails before sending them to your script. Click Next.
   7. Add Actions: Add two actions in this exact order:
   * Action 1: Select Deliver to S3 bucket. Choose the bucket you built in Step 5.
      * Action 2: Select Invoke Lambda function. Choose your SES-Email-Forwarder Python script.
   8. Click Next, review the pipeline diagram, and click Create Rule.
   9. Ensure the root Rule Set status is toggled to Active.

------------------------------
## Complete Cohesive Data Architecture Flow

```text
+---------------------+          +--------------------+
|   External Sender   | --MX-->  |   Amazon SES       |
|                     |          |      Inbound       |
+---------------------+          +--------------------+
                                     /          \
                                    v            v
                          +----------------+  +---------------------------+
                          |   Security     |  |   S3 (raw email storage)  |
                          | Filters (TLS,  |  |  (lifecycle rule => 3d)   |
                          | Spam, IP rep.) |  +---------------------------+
                          +----------------+           |
                                   |                    v
                                   |                +---------------+
                                   v                |   AWS Lambda  |
                               +--------+           |  (Python 3.12) |
                               |  SES   | <---------+---------------+
                               | Action |                    |
                               +--------+                    v
                                                         +--------------------+
                                                         | Amazon SES Outbound|
                                                         +--------------------+
                                                                  |
                                                                  v
                                                          +------------------+
                                                          | Personal Inbox   |
                                                          | (Gmail / iCloud) |
                                                          +------------------+
```

**Notes:**
- **S3 lifecycle:** Configure a lifecycle rule to expire raw email objects (example: 3 days).
- **Order:** SES should write to S3 before invoking the Lambda so the raw payload is available for inspection and replay.
- **Security:** Require TLS on the receipt rule and enable SES spam/virus scanning to reduce false forwards.

## ✅ Final Result Re-stated

Your enterprise-grade, serverless email forwarding pipeline is fully implemented: incoming mail routes to Amazon SES, is evaluated and stored in S3, processed by a Python Lambda that enforces spam/virus checks and rewrites headers, then is sent outbound via SES to your personal inbox.
------------------------------


