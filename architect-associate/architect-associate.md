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
* Make sure to run `CHMOD 0400 file-name.pem` on your machine to enable access to private key, but protect against misusage
* Invoke particular key file usage during SSH login: `ssh -i "/path/to/key-file.pem"`
* __SSH Browser Connect__ is AWS utility to connect from browser session; It temporarly upload Access Key to machine enabling connection; Ingress port `22` needs to be open in __Security Group__

## Security Groups

* It's stateful security measurement comparing to ACL, which is stateless measurement. (Steful means that once connection is established, security rules are not evaluated each time. E.g. if there is inboud traffic allows for http/8080, also response request is allowed even if not allowed by rules)

* Security Groups are defined within Region/VPC combination

* Best practise is to separate dedicated security group for SSH only

* Security group blocking access leads to to __timeouts__; `Connection refused` is usually caused by application/server issue

* You can authorize incoming access based on client IP/port, but also based on Security Group assigned to client machine within same VPC. e.g. all machines with assigned Security Group XXX, can access my server

## Networking

### Public, Private IP

* __Elastic IP__ is static Public IP; It's needed to be released back to pool from Networking/Elastic IP menu once it's not associated to any machine, otherwise it's still charged
