# Module: Digital Rights Management (DRM) and Encryption

When working with videos for your service or Over the Top (OTT) platform, you will very likely need to secure and protect your content prior to delivering videos to your end users. Approaches for securing content include basic content _encryption_ or by applying highly secure Digital Rights Management (DRM) to the content. Examples of DRM include Fairplay, Widevine and PlayReady.

In this module, you'll use AWS Elemental MediaConvert, a file-based video transcoding service to secure and encrypt your videos. You'll learn about the Secure Packager and Encoder Key Exchange (SPEKE) API, deploy an AWS SPEKE reference server, and configure AWS Elemental MediaConvert to encrypt HLS packaged content using AES-128 encryption.

## Prerequisites
You'll need to have previously deployed the AWS SPEKE Reference Server.<br/>
https://github.com/awslabs/speke-reference-server

Once you've installed the AWS SPEKE Reference Server retrieve the SPEKE API URL from AWS API Gateway on your AWS account.
 API Gateway -> SPEKEReferenceAPI -> Stage -> CopyProtection -> POST

    https://{hostname}.execute-api.us-east-1.amazonaws.com/EkeStage/copyProtection

You'll need the following resources created in module 1:
* **MediaConvertRole** - the role created to give permission for MediaConvert to access resources in your account.
* **MediaBucket** - the bucket created to store outputs from MediaConvert

## 1. Testing the SPEKE API...

### Lambdas

#### Server Test

1. Navigate to the AWS Lambda Console
1. Select the region deployed with the SPEKE Reference Server
1. Select the function that contains the name {STACKNAME}-SPEKEServerLambda-{}
1. Pull down the test events list at the top right
1. Choose Configure test events
1. Set the Saved Test Event name to ServerKeyRequest
1. Replace the **hostname** and Copy the following exactly into the text area for the event
```
{
  "resource": "/copyProtection",
  "path": "/copyProtection",
  "httpMethod": "POST",
  "headers": {
    "Accept": "*/*",
    "content-type": "application/xml",
    "Host": "hostname.execute-api.us-east-1.amazonaws.com"
  },
  "requestContext": {
    "path": "/EkeStage/copyProtection",
    "stage": "EkeStage",
    "resourcePath": "/copyProtection",
    "httpMethod": "POST"
  },
  "body": "PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiPz48Y3BpeDpDUElYIGlkPSI1RTk5MTM3QS1CRDZDLTRFQ0MtQTI0RC1BM0VFMDRCNEUwMTEiIHhtbG5zOmNwaXg9InVybjpkYXNoaWY6b3JnOmNwaXgiIHhtbG5zOnBza2M9InVybjppZXRmOnBhcmFtczp4bWw6bnM6a2V5cHJvdjpwc2tjIiB4bWxuczpzcGVrZT0idXJuOmF3czphbWF6b246Y29tOnNwZWtlIj48Y3BpeDpDb250ZW50S2V5TGlzdD48Y3BpeDpDb250ZW50S2V5IGtpZD0iNmM1ZjUyMDYtN2Q5OC00ODA4LTg0ZDgtOTRmMTMyYzFlOWZlIj48L2NwaXg6Q29udGVudEtleT48L2NwaXg6Q29udGVudEtleUxpc3Q+PGNwaXg6RFJNU3lzdGVtTGlzdD48Y3BpeDpEUk1TeXN0ZW0ga2lkPSI2YzVmNTIwNi03ZDk4LTQ4MDgtODRkOC05NGYxMzJjMWU5ZmUiIHN5c3RlbUlkPSI4MTM3Njg0NC1mOTc2LTQ4MWUtYTg0ZS1jYzI1ZDM5YjBiMzMiPiAgICA8Y3BpeDpDb250ZW50UHJvdGVjdGlvbkRhdGEgLz4gICAgPHNwZWtlOktleUZvcm1hdCAvPiAgICA8c3Bla2U6S2V5Rm9ybWF0VmVyc2lvbnMgLz4gICAgPHNwZWtlOlByb3RlY3Rpb25IZWFkZXIgLz4gICAgPGNwaXg6UFNTSCAvPiAgICA8Y3BpeDpVUklFeHRYS2V5IC8+PC9jcGl4OkRSTVN5c3RlbT48L2NwaXg6RFJNU3lzdGVtTGlzdD48Y3BpeDpDb250ZW50S2V5UGVyaW9kTGlzdD48Y3BpeDpDb250ZW50S2V5UGVyaW9kIGlkPSJrZXlQZXJpb2RfZTY0MjQ4ZjYtZjMwNy00Yjk5LWFhNjctYjM1YTc4MjUzNjIyIiBpbmRleD0iMTE0MjUiLz48L2NwaXg6Q29udGVudEtleVBlcmlvZExpc3Q+PGNwaXg6Q29udGVudEtleVVzYWdlUnVsZUxpc3Q+PGNwaXg6Q29udGVudEtleVVzYWdlUnVsZSBraWQ9IjZjNWY1MjA2LTdkOTgtNDgwOC04NGQ4LTk0ZjEzMmMxZTlmZSI+PGNwaXg6S2V5UGVyaW9kRmlsdGVyIHBlcmlvZElkPSJrZXlQZXJpb2RfZTY0MjQ4ZjYtZjMwNy00Yjk5LWFhNjctYjM1YTc4MjUzNjIyIi8+PC9jcGl4OkNvbnRlbnRLZXlVc2FnZVJ1bGU+PC9jcGl4OkNvbnRlbnRLZXlVc2FnZVJ1bGVMaXN0PjwvY3BpeDpDUElYPg==",
  "isBase64Encoded": true
}
```
8. Select the ServerKeyRequest saved test event
1. Click the Test button
1. Expand the details of the execution result
1. Find and verify the following XML data in the Log output compartment (formatted for readability); this data includes the encoded encryption key value and the key ID (kid)
```
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:cpix="urn:dashif:org:cpix" xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:speke="urn:aws:amazon:com:speke" id="5E99137A-BD6C-4ECC-A24D-A3EE04B4E011">
   <cpix:ContentKeyList>
      <cpix:ContentKey kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe">
         <cpix:Data>
            <pskc:Secret>
               <pskc:PlainValue>ALzP1aOTJvzfqg9I12k2Vw==</pskc:PlainValue>
            </pskc:Secret>
         </cpix:Data>
      </cpix:ContentKey>
   </cpix:ContentKeyList>
   <cpix:DRMSystemList>
      <cpix:DRMSystem kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe" systemId="81376844-f976-481e-a84e-cc25d39b0b33">
         <speke:KeyFormat />
         <speke:KeyFormatVersions />
         <cpix:URIExtXKey>aHR0cHM6Ly9kMnVod2Jqc3p1ejF2Ny5jbG91ZGZyb250Lm5ldC81RTk5MTM3QS1CRDZDLTRFQ0MtQTI0RC1BM0VFMDRCNEUwMTEvNmM1ZjUyMDYtN2Q5OC00ODA4LTg0ZDgtOTRmMTMyYzFlOWZl</cpix:URIExtXKey>
      </cpix:DRMSystem>
   </cpix:DRMSystemList>
   <cpix:ContentKeyPeriodList>
      <cpix:ContentKeyPeriod id="keyPeriod_e64248f6-f307-4b99-aa67-b35a78253622" index="11425" />
   </cpix:ContentKeyPeriodList>
   <cpix:ContentKeyUsageRuleList>
      <cpix:ContentKeyUsageRule kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe">
         <cpix:KeyPeriodFilter periodId="keyPeriod_e64248f6-f307-4b99-aa67-b35a78253622" />
      </cpix:ContentKeyUsageRule>
   </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```
### API Gateway

#### Server Test

1. Navigate to the AWS API Gateway Console
1. Select the region deployed with the SPEKE Reference Server
1. Select the SPEKEReferenceAPI
1. Select the POST method on the /copyProtection resource
1. Click the Test link on the left side of the main compartment
1. Replace the **hostname** Copy the following into the Headers {copyProtection} compartment
```
Host:hostname.execute-api.us-east-1.amazonaws.com
```
1. Copy the following into the Request Body compartment
```
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX id="5E99137A-BD6C-4ECC-A24D-A3EE04B4E011" 
    xmlns:cpix="urn:dashif:org:cpix" 
    xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" 
    xmlns:speke="urn:aws:amazon:com:speke">
    <cpix:ContentKeyList>
        <cpix:ContentKey kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe"></cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe" systemId="81376844-f976-481e-a84e-cc25d39b0b33">
            <cpix:ContentProtectionData />
            <speke:KeyFormat />
            <speke:KeyFormatVersions />
            <speke:ProtectionHeader />
            <cpix:PSSH />
            <cpix:URIExtXKey />
        </cpix:DRMSystem>
    </cpix:DRMSystemList>
    <cpix:ContentKeyPeriodList>
        <cpix:ContentKeyPeriod id="keyPeriod_e64248f6-f307-4b99-aa67-b35a78253622" index="11425"/>
    </cpix:ContentKeyPeriodList>
    <cpix:ContentKeyUsageRuleList>
        <cpix:ContentKeyUsageRule kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe">
            <cpix:KeyPeriodFilter periodId="keyPeriod_e64248f6-f307-4b99-aa67-b35a78253622"/>
        </cpix:ContentKeyUsageRule>
    </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```
6. Click the Test button
1. Review the Response Body for the encoded key value
```
<cpix:CPIX xmlns:cpix="urn:dashif:org:cpix" xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:speke="urn:aws:amazon:com:speke" id="5E99137A-BD6C-4ECC-A24D-A3EE04B4E011">
    <cpix:ContentKeyList>
        <cpix:ContentKey kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe"><cpix:Data><pskc:Secret><pskc:PlainValue>ALzP1aOTJvzfqg9I12k2Vw==</pskc:PlainValue></pskc:Secret></cpix:Data></cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe" systemId="81376844-f976-481e-a84e-cc25d39b0b33">
            <speke:KeyFormat />
            <speke:KeyFormatVersions />
            <cpix:URIExtXKey>aHR0cHM6Ly9kMnVod2Jqc3p1ejF2Ny5jbG91ZGZyb250Lm5ldC81RTk5MTM3QS1CRDZDLTRFQ0MtQTI0RC1BM0VFMDRCNEUwMTEvNmM1ZjUyMDYtN2Q5OC00ODA4LTg0ZDgtOTRmMTMyYzFlOWZl</cpix:URIExtXKey>
        </cpix:DRMSystem>
    </cpix:DRMSystemList>
    <cpix:ContentKeyPeriodList>
        <cpix:ContentKeyPeriod id="keyPeriod_e64248f6-f307-4b99-aa67-b35a78253622" index="11425" />
    </cpix:ContentKeyPeriodList>
    <cpix:ContentKeyUsageRuleList>
        <cpix:ContentKeyUsageRule kid="6c5f5206-7d98-4808-84d8-94f132c1e9fe">
            <cpix:KeyPeriodFilter periodId="keyPeriod_e64248f6-f307-4b99-aa67-b35a78253622" />
        </cpix:ContentKeyUsageRule>
    </cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```
## 2. Configuring DRM on a MediaPackage EndPoint

1. Login to the AWS Console
1. Navigate to MediaPackage
1. Select the **live-livestream** channel
1. Scroll down to **Endpoints** section of the Channel Detals
![s3 link](../images/live_mediapackage-endpoints.png)


## 3. Play the videos

To play the videos, you will use the S3 HTTPS resource **Link** on the videos S3 object **Overview** page.

![s3 link](../images/module-2-s3-link.png)


#### HLS

The HLS manifest file is located in your ouput s3 bucket in the object: s3://YOUR-MediaBucket/assets/VANLIFE/HLS/VANLIFE.m3u8

You can play the HLS using:
* Safari browser by clicking on the **Link** for the object.
* **JW Player Stream Tester** - by copying the link for the object and inputing it to the player.  https://developer.jwplayer.com/tools/stream-tester/ 


## Completion

Congratulations!  You have successfully created an encrypted video asset using  AWS Elemental MediaConvert. 