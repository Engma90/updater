# Updater docs

- Download and parse `appcast.xml` file, its content will be something like:
  ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <rss version="2.0" xmlns:sparkle="http://www.andymatuschak.org/xml-namespaces/sparkle">
        <channel>
            <item>
                <enclosure
                    url="https://staging8.presentationtools.com/wp-content/uploads/aps/Auto-Presentation-Switcher-v.2.3.0.1.exe"
                    sparkle:version="2.3.0.1"
                    sparkle:signature="HSsxcZ58dbekiUVLR4Dq+Ke5e9kILtTArsYlozQdXcPLXHN67IK+zNZsga4XI8MX86w3ixZzuTxKNGwLGt/9kFBM9LudggWNzoldsU6MwmS/8fVxVoWc2UjNrRHvkiu06HSXqg9ukSualLkTHpn33JSm6mCsEE/gS2PkKuhyh+tJOlrX6d4DVccs9hgS58f5Yc3nsLI5LDxlZ89RR4SrNXZFQIUfbFhefARXVAd10C+ll3WSycDJTCntcJ/IQWBS4NOR2FyLtk4+2UjR3leQKJeKlXcYa1NEEWTE6Puhbz0aGaHIPVaCBYXD8ExLxxxChSl9Msc9Ld0hCJjuXr0m6A=="
                    />
                <sparkle:releaseNotesLink>https://staging8.presentationtools.com/wp-content/uploads/aps/2.3.0.1.md</sparkle:releaseNotesLink>
                <pubDate>Fri, 24 Nov 2023 23:19:20 +0000</pubDate>
            </item>
        </channel>
    </rss>
  ```
- `sparkle:version`: is the latest uploaded version if it is greater than the current running app version, show a dialog/notification to the user.
- `url`: the executable file download link
- `sparkle:signature`: Base64 encoded RSA digintal signature, that can be used to verify the downloaded file is trusted. You will be given a public RSA key to use it for validation
  
  Example of verification process in c#
  ```c#
  public ValidationResult VerifySignature(string signature, byte[] dataToVerify)
  {  
      using (var rsa = new RSACryptoServiceProvider())
      {
          // Load the public key information
          rsa.FromXmlString(PublicKey);
  
          // Convert the base64 signature to bytes
          byte[] signatureBytes = Convert.FromBase64String(signature);
  
          // Verify the signature
          if (rsa.VerifyData(dataToVerify, new SHA256CryptoServiceProvider(), signatureBytes))
          {
              return ValidationResult.Valid;
          }
          else
          {
              return ValidationResult.Invalid;
          }
      }
  }
  ```
- `sparkle:releaseNotesLink`: Link of release notes markdown file.
- `pubDate` when this release was published

## Security
There should be another digital signature verification (In addition to the one mentioned above). before opening the the `appcast.xml`, its content should be verified using the same RSA public key, the DS of this file is stored in another file called `appcast.xml.signature` which will be located in the same directory as `appcast.xml`

![image](https://github.com/Engma90/updater/assets/12226980/c833cb3a-68f1-447b-9bbe-54764d773815)

In conclusion, the steps should be:
- Download `.xml` and `.xml.signature` files
- Verify that the signature (From `.xml.signature`) is valid for the xml file using the public RSA key
- If valid, compare versions and download if server is greater
- Verify that the signature (From `sparkle:signature`) is valid for the exe/dmg file using the public RSA key
- If valid, allow installation


## Windows UI example
In windows we use [NetSparkle](https://github.com/NetSparkleUpdater/NetSparkle) library, for Mac we will need to implement something similar that has only the basic needed functionality (check, notify, verify DS, and install)

![image](https://github.com/Engma90/updater/assets/12226980/61d7b483-0d39-4f6a-b0ea-9ac68828c918)
