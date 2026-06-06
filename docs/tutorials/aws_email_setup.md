AI Generated Tutorial

Master blueprint to configure a completely private, personal email service using AWS and Mail-in-a-Box.

This architecture balances data privacy with cost-efficiency: incoming email storage and IMAP access are handled on an isolated Amazon Lightsail virtual server for a flat $5.00/month, while outgoing mail is securely offloaded to Amazon SES to bypass residential IP blocks and guarantee high delivery rates.

------------------------------
## Step 1: Create and Secure Your AWS Account

   1. Navigate to the AWS Signup Page and register your personal account.
   2. Log into the AWS Console using your new Root User credentials.
   3. Search for IAM (Identity and Access Management) in the top search bar.
   4. Create an IAM Admin User for daily operations to ensure you avoid logging in as the root user.
   5. Enable Multi-Factor Authentication (MFA) on both your Root and IAM user accounts to keep your billing profile secure.

------------------------------
## Step 2: Register a Custom Domain via Amazon Route 53 [1] 
To use a custom email identity, you must own the domain. Registering directly inside AWS seamlessly auto-generates the necessary hosted routing zones. [2] 

   1. Search for Route 53 in the AWS console search bar and open the dashboard.
   2. Click Registered domains in the left sidebar and choose Register domain.
   3. Enter your desired personal domain name, select an extension (like .com), and click Check to verify availability.
   4. Click Add to cart next to your choice and click Continue.
   5. Provide your contact details. Ensure Privacy Protection is enabled (this keeps your identity hidden from public WHOIS marketing databases at no extra charge).
   6. Agree to the terms and click Complete order. Registration usually processes within a few minutes. [1, 3, 4] 

------------------------------
## Step 3: Spin Up a Lightsail Server and Open Firewall Ports

   1. Open the Amazon Lightsail Console.
   2. Click Create instance, select Linux/Unix, and click OS Only.
   3. Choose Ubuntu 22.04 LTS as the operating system.
   4. Select the entry-level plan (roughly $5.00/month, which features a public IPv4 address, 512 MB RAM, and a generous 1 TB data transfer limit).
   5. Name your instance (e.g., PersonalMailServer) and click Create instance.
   6. Navigate to the Networking tab inside your instance settings, click Create static IP, and attach it to your instance. Write down this IP address.
   7. Under the IPv4 Firewall rules on that same Networking tab, add custom rules to open these mandatory traffic ports:
   * TCP 143 & 993 (Standard IMAP and Secure IMAP)
      * TCP 25 (Incoming Mail traffic from other servers)
      * TCP 80 & 443 (For Webmail account management and automated Let's Encrypt SSL security certificates)
   
------------------------------
## Step 4: Install Mail-in-a-Box

   1. In the Lightsail dashboard card for your instance, click the Connect using SSH button to open a terminal browser window. [1] 
   2. Run system package updates:
   
   sudo apt-get update && sudo apt-get upgrade -y
   
   3. Execute the automated Mail-in-a-Box infrastructure script:
   
   curl -s https://mailinabox.email | sudo bash
   
   [5] 
   4. Follow the interactive installer prompts:
   * Provide your primary admin email address (e.g., me@yourdomain.com).
      * Set your server's system hostname to ://yourdomain.com.
      * Create a strong administrative password to lock down your future mail management panel.
   
------------------------------
## Step 5: Verify Your Domain in Amazon SES [6] 

   1. Open the [Amazon SES Console](https://console.aws.amazon.com/ses) and select Identities under the Configuration sidebar menu. [7] 
   2. Click Create identity, choose Domain, and type in your newly purchased root domain (e.g., yourdomain.com). [7, 8] 
   3. Check the box for Easy DKIM and hit Create identity. [6, 7] 
   4. AWS will display a set of CNAME records. Leave this tab open. [6] 
   5. In an alternate tab, open your Route 53 Hosted Zone for your domain. Click Create record, copy the SES CNAME names and values, and paste them as new records. This tells AWS you authorize email delivery. [6] 
   6. Return to the SES left sidebar, click SMTP Settings, and write down your region's unique SMTP Endpoint (e.g., email-smtp.us-east-1.amazonaws.com).
   7. Click Create SMTP credentials, accept the profile defaults, click create, and Download Credentials. Store this CSV file safely; these are your internal server-to-SES relay passwords.

------------------------------
## Step 6: Map Domain DNS Records
Go to your Route 53 Hosted Zone dashboard for your domain. Create the final records pointing your web identity to your server infrastructure: [4] 

   1. A Record: Name/Subdomain: box | Value: [Your Lightsail Static IP]
   2. MX Record: Name/Subdomain: @ (or blank) | Value: 10 ://yourdomain.com
   3. TXT Record (SPF Security): Name/Subdomain: @ | Value: v=spf1 include:amazonses.com ~all
   4. TXT Record (DMARC Protection): Name/Subdomain: _dmarc | Value: v=DMARC1; p=none; rua=mailto:dmarc-reports@yourdomain.com

------------------------------
## Step 7: Configure Postfix Relay and Exit the SES Sandbox
To bypass the AWS outbound port restrictions, your server must securely push outbound mail to SES via port 587.

   1. Back in your Lightsail SSH terminal window, open the Postfix mail configuration engine:
   
   sudo nano /etc/postfix/main.cf
   
   2. Scroll to the absolute bottom of the file and paste these relay rules:
   
   relayhost = [email-smtp.us-east-1.amazonaws.com]:587
   smtp_sasl_auth_enable = yes
   smtp_sasl_security_options = noanonymous
   smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
   smtp_use_tls = yes
   smtp_tls_security_level = encrypt
   smtp_tls_note_starttls_offer = yes
   
   3. Save and close the editor (Ctrl+O, Enter, Ctrl+X), then construct the password credential file:
   
   sudo nano /etc/postfix/sasl_passwd
   
   4. Map your SES endpoint to the downloaded SMTP keys using this format:
   
   [email-smtp.us-east-1.amazonaws.com]:587 YOUR_SES_SMTP_USERNAME:YOUR_SES_SMTP_PASSWORD
   
   5. Save, close, restrict directory read privileges, and apply the new configuration database:
   
   sudo chmod 0600 /etc/postfix/sasl_passwd
   sudo postmap /etc/postfix/sasl_passwd
   sudo systemctl restart postfix
   
   6. Navigate to your Amazon SES Dashboard. Click the Request production access link pinned inside the sandbox warning banner. State clearly that you are running a single-user private mailbox on an isolated server and submit the request to remove regional sending restrictions. [6] 

------------------------------
## Step 8: Create Mailboxes and Sync macOS Mail
Your private cloud network is now fully live. You can add infinite unique addresses or throwaway tracking aliases for no extra charge.

   1. Open your web browser and go to https://yourdomain.com. Log in using your Step 4 admin password. [5] 
   2. To create separate standalone mailboxes: Navigate to Mail > Users and click Add User.
   3. To generate privacy-tracking aliases: Navigate to Mail > Aliases and click Add Alias. You can map shadow addresses (like banking@yourdomain.com) to dump silently into your main inbox without giving away your real handle.
   4. Launch the native Mail app on your macOS computer.
   5. Select Mail > Add Account > Other Mail Account... [1] 
   6. Type your name, your custom email identity, and your Mail-in-a-Box account password.
   7. Fill out the manual settings when prompted:
   * Account Type: IMAP
      * Incoming Mail Server: ://yourdomain.com
      * Outgoing Mail Server: ://yourdomain.com (Your server intercepts this request and hands it to Amazon SES automatically behind the scenes).
   
------------------------------
## Step 9: Establish Free AWS Cost Alerts

   1. Search for Billing in your main AWS console top search bar.
   2. Select Budgets from the left navigation panel and click Create budget.
   3. Choose the Fixed Cost Budget template option.
   4. Define your monthly cost spending threshold as $6.00.
   5. Add your personal phone number or personal backup email address under the alert notification form. AWS will ping you immediately if your server ever generates an anomalous data fee.

------------------------------

### Reference

- [1] [https://www.youtube.com](https://www.youtube.com/watch?v=5IfDzpkLlYY)
- [2] [https://docs.aws.amazon.com](https://docs.aws.amazon.com/hands-on/latest/get-a-domain/get-a-domain.html)
- [3] [https://docs.aws.amazon.com](https://docs.aws.amazon.com/hands-on/latest/get-a-domain/get-a-domain.html)
- [4] [https://www.youtube.com](https://www.youtube.com/watch?v=nEi6Rb5XupA&vl=en)
- [5] [https://mailinabox.email](https://mailinabox.email/guide.html)
- [6] [https://aws.amazon.com](https://aws.amazon.com/blogs/opensource/fully-automated-deployment-of-an-open-source-mail-server-on-aws/)
- [7] [https://www.replyup.com](https://www.replyup.com/blog/amazon-ses-tutorial/)
- [8] [https://docs.aws.amazon.com](https://docs.aws.amazon.com/hands-on/latest/send-an-email-with-amazon-ses/send-an-email-with-amazon-ses.html)
