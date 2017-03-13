# AWS_CLOUD_FORMATION_TEMP
In this tutorial we will create AWS CloudFormation Template which used to launching AWS Instance and installing Tomcat Application Server on it.
Here is AWS CloudFormation Template:

{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description" : "Sample CloudFormation script to install Tomcat7. Defaults only work for US West (Oregon)",

    "Parameters" : {
        "AWSImageID" : {
            "Default" : "ami-f173cc91",
            "Type" : "String"
    },
    "KeyName" : {
        "Description" : "Name of an existing EC2 KeyPair to enable SSH access to the instances",
        "Type" : "AWS::EC2::KeyPair::KeyName",
        "ConstraintDescription" : "Must be the name of an existing EC2 KeyPair"
    },
    "EC2InstanceType" : {
        "Default" : "t2.micro",
        "Description" : "We default to t2.micro for convenience",
        "Type" : "String"
        }
    },

    "Resources" : {
        "TomcatInstance" : {
            "Type" : "AWS::EC2::Instance",
        "Properties" : {
            "ImageId" : {
                "Ref" : "AWSImageID"
    },
            "KeyName" : {
                "Ref" : "KeyName"
    },
            "SecurityGroups" : [
                    {
                "Ref" : "TomcatSecurityGroup"
            }
    ],
            "InstanceType" : {
                "Ref" : "EC2InstanceType"
    },
            "UserData" : {
                "Fn::Base64" : {
                "Fn::Join" : [
                "",
                        [
                        "#!/bin/bash -ex",
                        "\n",
                        "yum -y update",
                        "\n",                        
                        "cd /tmp",
                        "\n",
                        "wget http://apache.volia.net/tomcat/tomcat-7/v7.0.75/bin/apache-tomcat-7.0.75.tar.gz",
                        "\n",
                        "tar xzf apache-tomcat-7.0.75.tar.gz",
                        "\n",
                        "mv apache-tomcat-7.0.75 /usr/local/tomcat7",
                        "\n",
                        "cd /usr/local/tomcat7",
                        "\n",
                        "./bin/startup.sh",
                        "\n"
                        ]
                    ]
                }
            }
        }
    },

    "TomcatSecurityGroup" : {
            "Type" : "AWS::EC2::SecurityGroup",
        "Properties" : {
            "GroupDescription" : "Enable HTTP access via port 80/443 and SSH access",
            "SecurityGroupIngress" : [
                {"IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "CidrIp" : "0.0.0.0/0"},
                {"IpProtocol" : "tcp", "FromPort" : "8080", "ToPort" : "8080", "CidrIp" : "0.0.0.0/0"},
                {"IpProtocol" : "tcp", "FromPort" : "443", "ToPort" : "443", "CidrIp" : "0.0.0.0/0"},
                {"IpProtocol" : "tcp", "FromPort" : "22", "ToPort" : "22", "CidrIp" : "0.0.0.0/0"},
                {"IpProtocol" : "icmp", "FromPort" : "-1", "ToPort" : "-1", "CidrIp" : "0.0.0.0/0"}
                ]
            }
        }
    },

    "Outputs" : {
        "URL" : {
        "Description" : "Public URL",
        "Value" : { "Fn::Join" : ["", ["http://", { "Fn::GetAtt" : [ "TomcatInstance", "PublicIp" ]}, ":8080"]]}
        }
    }
}
