# CloudFormation Templates

FriendlyBot infrastructure-as-code.

## Stacks

### Google MX Record + Site Verification

Email is handled by Google, which also requires domain ownership verification
through adding a TXT DNS record to the zone.

* Add the domain in Google Admin, and retrive the site verification code.
* Run the stack with the site verification code.
* Complete the site verification within Google Admin.
* Ensure you view instructions for MX records and click "I've done this" in Google Admin.

```console
user@host:cloudformation$ aws cloudformation deploy \
  --template-file=./templates/friendlybot-google.template \
  --stack-name=friendlybot-google \
  --parameter-overrides GoogleSiteVerification=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  --region=us-east-2
```

### SSL Certificate

Constraints:
* CloudFormation: Automatic DNS verification unsupported, so we're using email
  verification.
* CloudFront: currently only able to use certificates from the `us-east-1`
  region, so the Certificate must be managed there.

* First, ensure emails to admin@domain go to a mailbox you control.
* Run the stack, it will pause at in-progess stage.
* Check email, click the verification link to confirm ownership of the domain for ACM.

```console
user@host:cloudformation$ aws cloudformation deploy \
  --template-file=./templates/friendlybot-certificate.template \
  --stack-name=friendlybot-certificate \
  --region=us-east-1
```

### Website

* Static site hosted in S3 bucket.
* Fronted by CloudFront distribution.
* SSL Enabled.
* Redirects non-www (apex) to www domain.
* Redirects http to https.

```console
user@host:cloudformation$ aws cloudformation deploy \
  --template-file=./templates/friendlybot-website.template \
  --stack-name=friendlybot-website \
  --parameter-overrides AcmCertificateArn=arn:aws:acm:us-east-1:ACCOUNTNUMBER:certificate/CERTIFICATE_ID \
  --region=us-east-2
```
