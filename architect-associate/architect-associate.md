# AWS Solution Architect Associate


## IAM

* __Roles__

* __Groups__

* __Resources__
  * Prevent users to access non-authorized resources

### Hints

* Always delete Root Access keys and enable MFA for root user

* Never use root-user

## EC2

= Elastic Cloud Computing

### SSH to EC2 machines

* Access keys are defined during machine set-up
* Make sure to run `CHMOD 0400` on your machine to enable access to private key, but protect against misusage
* Invoke particular key file usage during SSH login: `ssh -i "/path/to/key-file.pem"`