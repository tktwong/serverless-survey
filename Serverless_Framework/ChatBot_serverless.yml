service: lambda-function

provider:
  name: aws
  runtime: nodejs6.10
  region: 'us-east-1'
  vpc:
    securityGroupIds:
      - "Fn::GetAtt": ServerlessSecurityGroup.GroupId
    subnetIds:
      - Ref: ServerlessPublicSubnetA
      - Ref: ServerlessPublicSubnetB
  iamRoleStatements:
    - Effect: Allow
      Action:
        - ec2:CreateNetworkInterface
        - ec2:DescribeNetworkInterfaces
        - ec2:DeleteNetworkInterface
      Resource: "*"
functions:
  ChatBot:
    handler: handler.ChatBot
    events:
     - http:
         path: /webhook
         method: GET   
         integration: lambda

     - http:
         path: /webhook
         method: POST   
         integration: lambda
  DB:
    handler: handler.DB
    events:
     - http:
         path: /DB
         method: ANY  
         integration: lambda 

resources:
  Resources:

    ServerlessVPC:
      Type: AWS::EC2::VPC
      Properties:
        CidrBlock: "10.0.0.0/16"

    ServerlessPublicSubnetA:
      DependsOn: ServerlessVPC
      Type: AWS::EC2::Subnet
      Properties:
        VpcId:
          Ref: ServerlessVPC
        AvailabilityZone: ${self:provider.region}a
        CidrBlock: "10.0.0.0/24"


    ServerlessPublicSubnetB:
      DependsOn: ServerlessVPC
      Type: AWS::EC2::Subnet
      Properties:
        VpcId:
          Ref: ServerlessVPC
        AvailabilityZone: ${self:provider.region}b
        CidrBlock: "10.0.1.0/24"

    ServerlessSecurityGroup:
      DependsOn: ServerlessVPC
      Type: AWS::EC2::SecurityGroup
      Properties:
        GroupDescription: SecurityGroup for Serverless Functions
        VpcId:
          Ref: ServerlessVPC
    ServerlessStorageSecurityGroup:
      DependsOn: ServerlessVPC
      Type: AWS::EC2::SecurityGroup
      Properties:
        GroupDescription: Ingress for RDS Instance
        VpcId:
          Ref: ServerlessVPC
        SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '3306'
          ToPort: '3306'
          SourceSecurityGroupId:
            Ref: ServerlessSecurityGroup
    ServerlessRDSSubnetGroup:
      Type: AWS::RDS::DBSubnetGroup
      Properties:
        DBSubnetGroupDescription: "RDS Subnet Group"
        SubnetIds:
        - Ref: ServerlessPublicSubnetA
        - Ref: ServerlessPublicSubnetB
    ServerlessRDSDBParameterGroup:
      Type: AWS::RDS::DBParameterGroup
      Properties: 
        Description: parameter group
        Family: MySQL5.7
        Parameters:
          character_set_database: "utf8mb4"
          character_set_client: "utf8mb4"
          character_set_connection: "utf8mb4"
          character_set_results: "utf8mb4"
          character_set_server: "utf8mb4"
          skip-character-set-client-handshake : "TRUE"

    ServerlessRDSCluster:
      DependsOn: ServerlessStorageSecurityGroup
      Type: AWS::RDS::DBInstance
      Properties:
        Engine: mysql
        EngineVersion: 5.7.19
        DBName: Emojis
        MasterUsername: bacem
        MasterUserPassword: 'dictionnaire'
        DBInstanceClass: db.t2.micro
        AllocatedStorage: 20
        VPCSecurityGroups:
        - "Fn::GetAtt": ServerlessStorageSecurityGroup.GroupId
        DBSubnetGroupName:
          Ref: ServerlessRDSSubnetGroup
        DBParameterGroupName:
          Ref: ServerlessRDSDBParameterGroup

