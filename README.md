# Custom CF resource to decrypt secrets in CF templates

[![PyPI](https://img.shields.io/pypi/v/humilis-decrypt.svg?style=flat)](https://pypi.python.org/pypi/humilis-decrypt)

A [humilis][humilis] plug-in layer that deploys a custom Cloudformation (CF) resource that can be used to decrypt secrets that are embedded in CF templates.

__CREDIT:__ All credit goes to [Casecommons lambda-cfn-kms][casecommons]. This
repo is just the result of bundling the `lambda-cfn-kms` repository as a humilis
plugin.

[humilis]: https://github.com/humilis/humilis
[casecommons]: https://github.com/Casecommons/lambda-cfn-kms


## Installation

```
pip install humilis-decrypt
```

To install the development version:

```
pip install git+https://github.com/humilis/humilis-decrypt
```


## Development

Assuming you have [virtualenv][venv] installed:

[venv]: https://virtualenv.readthedocs.org/en/latest/

```
make develop
```

Configure humilis:

```
make configure
```


## How does it work?

First create the Lambda function that backs the custom resource:

```
make create
```

The deployment will produce two artifacts:

* The ID of the KMS key associated with the custom resource.
* The ARN of the deployed Lambda function.

You can use the KMS key ID to encrypt your secrets locally, e.g. assuming you want to encrypt the dummy DB password `dummy` with key `3ea941bf-ee54-4941-8f77-f1dd417667cd`:

```
aws kms encrypt --key-id 3ea941bf-ee54-4941-8f77-f1dd417667cd --plaintext 'dummy'
```

The output will be something like this:

```
{
    "CiphertextBlob": "AQICAHi2zdvZYfUQOQV8yX/HLdcIMqHHkubAYAei2Qo498KheQFDELPYHds8169cc9EqggEuAAAAZjBkBgkqhkiG9w0BBwagVzBVAgEAMFAGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM+gDfP3DDVgbFYNidAgEQgCMrz1beR+s0XqWTVIuCbJ+E/cC7sqUzmPEB0weOhQ3GOE65rg==",
    "KeyId": "arn:aws:kms:eu-west-1:XXXXX:key/a86x4dd8-6b8e-41ce-aa65-4aa370d9ccbf"
}
```

Whenever you want to use your secret in a CF template you would do something like this:

```
---
resources:
  DbPasswordDecrypt:
    Type: "Custom::KMSDecrypt"
    Properties:
      ServiceToken: <lambda_function_arn>
      Ciphertext: "AQICAHi2zdvZYfUQOQV8yX/HLdcIMqHHkubAYAei2Qo498KheQFDELPYHds8169cc9EqggEuAAAAZjBkBgkqhkiG9w0BBwagVzBVAgEAMFAGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQM+gDfP3DDVgbFYNidAgEQgCMrz1beR+s0XqWTVIuCbJ+E/cC7sqUzmPEB0weOhQ3GOE65rg=="
  DbInstance:
    Type: "AWS::RDS::DBInstance"
    Properties:
      AllocatedStorage: "20"
      DBInstanceClass: "db.m1.small"
      Engine: "MySQL"
      EngineVersion: "5.5"
      MasterUsername: "admin"
      MasterUserPassword:
        Fn::Sub: ${DbPasswordDecrypt.Plaintext}
```

where you will need to replace `<lambda_function_arn>` with the ARN of the Lambda function that backs the custom CF resource that implements the decryption logic.


## More information

See [humilis][humilis] documentation.

[humilis]: https://github.com/humilis/blob/master/README.md


## Contact

If you have questions, bug reports, suggestions, etc. please create an issue on the [GitHub project page][github].

[github]: http://github.com/humilis/humilis-decrypt


## License

See the original license in the [lambda-cfn-kms][license] repository.

[license]: https://github.com/Casecommons/lambda-cfn-kms#license
