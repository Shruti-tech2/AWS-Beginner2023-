# AWS-Beginner2023-

## SAML Authentiation:
SAML is used to enable SSO(Single Sign On). Where sso is a term used for a login method where a company configures all it's applications in a such a way that the user can log in to all of these apps by just signing in once.
SAML Authetication Flow:
**SAML Request** ======> **SAML Response** ======> **Key Generation**
**SAML Request**: terms included:
1. NameID- The username/email address or phone number which is used to identify a user.
2. AssertionConsumerServiceURL- The SAML URL interface of the SP(`Service Provider` like Gmail) where IP(`Identity Provider` like Google) sends the auth token.
**SAML Response**:
1. Assertion- is an XML document that has the details of the user. This contains the timestamp of the user login and the method of authentication used(eg: two factor authentication & etc.)
2. Signature- is a Base64 encoded string which  protects integrity of the assertion. Like if an attacker tries to change the username in assertion to the victim's username, the signature will prevent the hacker from logging in as the user.
**Key Generation**:
The identity Provider(like Google) generate the `private key` and a `public key`. It signs the assertion with private key. The public key is shared with the Service Provider(like Gmail) which used it to verify the SAML response and then log the user in.

## IAM Role `AWS::IAM::Role`
Identity Access Management, is used to control access to AWS resources. It helps you to manage and secure, who can do what within your AWS environment.IAM allows you to define and manage permissions, roles, and policies for your AWS users, groups, and resources.
Whenever we are creating infrastructure with AWS CloudFormation, IAM plays important role in defining permissions and access controls for resources.
Like in case to create `EventBus`, we need to create one `IAM ROLE` corresponding to the event bus, to grant other AWS service and resources to interact with Event Bus.
Example:
```java
Resources:
  EventBusRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: EventBusRole
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - events.amazonaws.com  # Service principal for EventBridge
            Action:
              - sts:AssumeRole

      Policies:
        - PolicyName: EventBusPolicy
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action:
                  - events:PutEvents
                Resource:
                  - "arn:aws:events:us-east-1:123456789012:event-bus/YourEventBusName"
```
- Type: AWS::IAM::Role: Specifies that you are creating an IAM role.
- AssumeRolePolicyDocument: This property defines who can assume the role. In this case, you are allowing the EventBridge service principal to assume the role.
- Policies: This property attaches policies to the role. In this example, you are attaching a policy called EventBusPolicy. This policy allows the IAM role to perform the events:PutEvents action on the specified Event Bus resource.
- Resource: The ARN (Amazon Resource Name) of the Event Bus to which the IAM role is granted access. Replace "arn:aws:events:us-east-1:123456789012:event-bus/YourEventBusName" with the ARN of your Event Bus.

**Result**: Whichever AWS resource like lambda, sqs or more will assume this IAM role, will be able to perform `PutEvent` action i.e. will be able to `Publish events` to this `eventBus`. This IAM role can be assumed in same or even in different account by resources.

### EventBusPolicy: `AWS::Event::EventBusPolicy`
This resource is used to define `EventBus-Level Permissions` as which AWS account or principal like IAM Role can perform specific actions directly on EventBus. The EventBusPolicy allows you to specify who can perform actions directly on the Event Bus, such as events:PutEvents. This policy can restrict access to specific actions on the Event Bus and not necessarily actions on other AWS resources.

**NOTE**:::: While working with EventBus, if we are defining IAM role, to be assumed by other AWS resources, then we need to give permissions to this IAM Role to perform specific actions on EventBus by defining it as Principal in EventBusPolicy.
EXAMPLE:
```java
Resources:
  EventBusPolicyForRole:
    Type: AWS::Events::EventBusPolicy
    Properties:
      Action:
        - events:PutEvents
      StatementId: EventBusPolicyStatement #is just unique string to differentiate statements in same policy.
      EventBusName: YourEventBusName
      Principal: "arn:aws:iam::123456789012:role/EventBusRole"
```
**NOTE**:::::::::::We don't need to use IAM Role of eventBus with other AWS reources if both EventBridge and AWS Resources are of same account.

### VPC (Virtual Private Cloud):
Is a logically isolated portion of the AWS cloud within a region.
Custom VPCs allow you to design your network architecture and security settings to meet your organization's specific needs. You have full control over the IP address range, subnets, route tables, security groups, and other networking configurations. This level of control is essential for isolating resources

### Subnet:
Subnet paly important role to create secure infrastructure in AWS. Each Subnet have associated rout table which control the routing of traffic within the subnet. Subnets are used to route traffic between resources.
- **Public Subnet**: hold all resources which can be directly accessed on internet.
- **Private subnet**: hold all resources which can not be directly accessed by internet. But they can be accessed by internet through use of NAT(Network Address Translation) Gateway , mean NAT Gateway can be deployed in Public IP instance and then Private IP instance of Private subnet will comminicate with it and accessed to internet.

 **NOTE**::::::::::::::::::In simple, Public Subnet Route Table hold `Internet Gateway` IP-Address as target for communication rule on traffic and Private Subnet Route Table take `nat-gateway-id` as target. These route table conssits rule for tarffic to access the internet.
 In more simple Intsance with the private address wants to connect to the internet, so to connect to the internet, it will need a public address that can access the internet, then here NAT gateway forward the connections to the internet.
 **NAT Gateway**: forwards connections to the internet.

### Security Groups and Network ACLs:
Both components used to control network traffic and enhance the security of resources withing VPC. Where `NACLs`are the first which apply at `Subnet`level, they apply only to traffic enterning / exiting the subnet. So if the instance A of diifer availabilty zones of same VPC or of differ VPC, want to communicate with other instance B, then that traffic will exit from it's NACL(,there will be some rules for that traffic to move out)and then at  B instance's side's NACL will have rule to allow same traffic(From instance A) to be in. In NACL case rules for inbound and outbound for a traffic is must as NACL is stateless.
Other side Security Groups are statefull firewall, that mean only one rule is required for traffic. As security groups apply at instance level they know that if the connection is allowed for inbound traffic, it is ok for outbound too.
In Simple, NACLs andSecurityGroup will allow communication of traffic between two instances with use of some Rules. NACLs use alwyas two rule for both of communcation between instances where Security Group use only single rule to allow both way communication.

### EC2:(Elastic Compute Cloud)
