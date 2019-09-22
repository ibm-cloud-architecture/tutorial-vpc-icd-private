# Use GUI, CLI or API to deploy a VPC and connect a VPC Application to a MongoDB using a private endpoint.

### Purpose
Here we will illustrate the use of the IBM Cloud API to create the required infrastructure components for a Virtual Private Cloud needed for this scenario.

### Step 1: Login to the IBM Cloud and obtain an IAM key

Log into the IBM Cloud console

**Command**
````
ibmcloud login
````

Obtain an IAM token by following [these directions](https://cloud.ibm.com/docs/vpc?topic=vpc-set-up-environment#api-prerequisites-setup).
**Important:** You must repeat this step to refresh your IAM token every hour, because the token expires.
 

Store the API endpoint and version in an environment variable so it can be reused later.

**Command**
````
api_endpoint=https://us-south.iaas.cloud.ibm.com
api_version="2019-05-31"
````
To verify that this variable was saved, run ````echo $api_endpoint```` and make sure the response is not empty.
 
### Step 2: Create an SSH Key
An SSH Key is required to access Virtual Servers in a VPC. To generate a new SSH Key and add it to your IBM Cloud account, follow these instructions:

1. Open a Terminal/Command Prompt
2. Issue the following command

    **Command**
    ```
    ssh-keygen -t rsa -C "root"
    ```
   **Output**
   ```
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/dlentz/.ssh/id_rsa): vpcCompute
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in vpcCompute.
    Your public key has been saved in vpcCompute.pub.
    The key fingerprint is:
    SHA256:04aNt/xusyQx4rf33leMNYEzABsw+wqYpfxlAp4z+qI root
    The key's randomart image is:
    +---[RSA 2048]----+
    |       o.o... .  |
    |        o o  + . |
    |   . . . .    o .|
    |  o B   .=     ..|
    |   X o oS.B    +.|
    |  . + =..* +  . o|
    | .   . .. = .   .|
    | ..      . =+  ..|
    |E ..      .++=o o|
    +----[SHA256]-----+
    ```
3. Add the SSH Key to the IBM Cloud

**Command** (Replace <vpcCompute.pub> with the contents of the vpcCompute.pub file
````
curl -X POST $api_endpoint/v1/keys?version=$api_version \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "dlentz-key",
        "public_key": "<vpcComputer.pub>",
        "type": "rsa"
      }'
````  

**Output**
````
{
  "created_at": "2019-02-04T15:29:50.000Z",
  "crn": "crn:v1:bluemix:public:is:us-south:a/26a3d1a386bd2cc44df1997eb7ac0ef1::key:636f6d70-0000-0001-0000-000000144a5b",
  "fingerprint": "SHA256:b3cQoQXOv9Hj1fn0DdmZTtYhscp4Qb+5df+ZuGINiLQ",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/keys/636f6d70-0000-0001-0000-000000144a5b",
  "id": "636f6d70-0000-0001-0000-000000144a5b",
  "length": 2048,
  "name": "dlentz-key",
  "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDDjMFASMv+ePuKeHSLbAb65n9OJRb6FuGKJacsigp6qWzjH5yuCsy2THTz01gTXorQbRR6Jfi1MC018Z/vfNsIflkT1Efk4X83k4rCaD5zGuvDdPsWI8O16u6WKaM08X4aJapwSbtcBy6/+SeK1hXV4cErC1OTfsBrP2BsLnH7uXe3ASCsHwHzwIglxIz/fAw5orNGZrr/leGBLU4T86Zoe+xGtbrJUydLnYu9zCv60qkvnwVQBhvuycGuAxGusBAkb5MN3YW6ZZVhawqtlPSOdVSJAfVR4Jo9/DHIjlHrPGElcbsN6Is+fDt9gy8UkAcn9/tMMSNoT2ale48eEHYb root",
  "type": "rsa"
}
````

Set an environment variable to hold the ID of the key created
````
ssh_key_id="636f6d70-0000-0001-0000-000000144a5b"
````

### Step 3: Create a new VPC instance
The command below will create a new VPC named 'vpc-1' in the IBM Cloud. The Resource group parameter is optional.

**Command**
```
curl -X POST $api_endpoint/v1/vpcs?version=$api_version \
     -H "Authorization: $iam_token" \
     -d "{ \"name\" : \"vpc-1\",  \
           \"resource_group\" : { \"id\" : \"d0bd962c2eea48d8b18b5d5e82e2fc60\"} \
     }" | json_pp
  
```
  - If you need to create a Resource Group, follow [these instructions](https://cloud.ibm.com/docs/resources?topic=resources-rgs#creating-a-resource-group)

**Output**
```
{
   "resource_group" : {
      "id" : "d0bd962c2eea48d8b18b5d5e82e2fc60",
      "href" : "https://resource-controller.cloud.ibm.com/v1/resource_groups/d0bd962c2eea48d8b18b5d5e82e2fc60"
   },
   "name" : "vpc-1",
   "crn" : "crn:v1:bluemix:public:is:us-south:a/843f59bad5553123f46652e9c43f9e89::vpc:a23f85bc-efd8-472c-be4f-51cc73c77f91",
   "default_network_acl" : {
      "name" : "allow-all-network-acl-a23f85bc-efd8-472c-be4f-51cc73c77f91",
      "crn" : "",
      "id" : "a662e613-d5d6-4192-b347-6fc908b1550b",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/network_acls/a662e613-d5d6-4192-b347-6fc908b1550b"
   },
   "classic_access" : false,
   "created_at" : "2019-09-21T05:11:21Z",
   "href" : "https://us-south.iaas.cloud.ibm.com/v1/vpcs/a23f85bc-efd8-472c-be4f-51cc73c77f91",
   "id" : "a23f85bc-efd8-472c-be4f-51cc73c77f91",
   "status" : "available",
   "default_security_group" : {
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/security_groups/2d364f0a-a870-42c3-a554-000001999214",
      "id" : "2d364f0a-a870-42c3-a554-000001999214",
      "name" : "decidable-quintuple-depose-comment"
   }
}
```

Set an environment variable equal to the VPC ID returned from IBM Cloud:
````
vpc_id="a23f85bc-efd8-472c-be4f-51cc73c77f91"
````

### Step 4: Create a subnet in the new VPC

**Command**
```
curl -X POST "$api_endpoint/v1/subnets?version=$api_version" \
     -H "Authorization: $iam_token" \
     -d "{ \"name\" : \"subnet1\", \
           \"ipv4_cidr_block\" : \"10.240.1.0/24\", \
           \"zone\" : { \"name\" : \"us-south-1\" }, \
           \"vpc\" : { \"id\": \"$vpc_id\" } \
         }"  | json_pp
```
**Output**
```
{
   "crn" : "crn:v1:bluemix:public:is:us-south-1:a/843f59bad5553123f46652e9c43f9e89::subnet:649543e3-f627-45e3-9c98-b65bfb69545e",
   "ip_version" : "ipv4",
   "zone" : {
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1",
      "name" : "us-south-1"
   },
   "id" : "649543e3-f627-45e3-9c98-b65bfb69545e",
   "href" : "https://us-south.iaas.cloud.ibm.com/v1/subnets/649543e3-f627-45e3-9c98-b65bfb69545e",
   "ipv4_cidr_block" : "10.240.1.0/24",
   "name" : "subnet1",
   "vpc" : {
      "crn" : "crn:v1:bluemix:public:is:us-south:a/843f59bad5553123f46652e9c43f9e89::vpc:a23f85bc-efd8-472c-be4f-51cc73c77f91",
      "id" : "a23f85bc-efd8-472c-be4f-51cc73c77f91",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/vpcs/a23f85bc-efd8-472c-be4f-51cc73c77f91",
      "name" : "vpc-1"
   },
   "status" : "pending",
   "total_ipv4_address_count" : 256,
   "network_acl" : {
      "name" : "allow-all-network-acl-a23f85bc-efd8-472c-be4f-51cc73c77f91",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/network_acls/a662e613-d5d6-4192-b347-6fc908b1550b",
      "id" : "a662e613-d5d6-4192-b347-6fc908b1550b",
      "crn" : ""
   },
   "created_at" : "2019-09-21T05:38:05Z",
   "available_ipv4_address_count" : 251
}
```

Set an environment variable equal to the SubNet ID returned from IBM Cloud:
```
vpc_subnet_id="649543e3-f627-45e3-9c98-b65bfb69545e"
```


### Step 5: Create a new Virtual Server in your VPC

**Command**
```
curl -X POST "$api_endpoint/v1/instances?version=$api_version" \
     -H "Authorization: $iam_token" \
     -d "{ \"name\" : \"vsi-1\", \
           \"vpc\" : { \"id\" : \"$vpc_id\" }, \
           \"zone\" : { \"name\" : \"us-south-1\" }, \
           \"resource_group\": { \"id\": \"d0bd962c2eea48d8b18b5d5e82e2fc60\" }, \
           \"image\": { \"id\" : \"cfdaf1a0-5350-4350-fcbc-97173b510843\" }, \
           \"profile\" : { \"name\" : \"cc1-2x4\" },
           \"keys\" : [ { \"id\" : \"$ssh_key_id\" } ], \
           \"primary_network_interface\" : { \
              \"name\" : \"eth0\", \
              \"subnet\" : { \"id\" : \"$vpc_subnet_id\"}, \
              \"security_groups\" : [ { \"id\" : \"2d364f0a-a870-42c3-a554-000001999214\" } ] \
            } \
         }" | json_pp
```

**Output**
```
{
   "id" : "1c8a08fd-7b32-4ca5-8716-4903ef3f93c8",
   "crn" : "crn:v1:bluemix:public:is:us-south-1:a/843f59bad5553123f46652e9c43f9e89::instance:1c8a08fd-7b32-4ca5-8716-4903ef3f93c8",
   "resource_group" : {
      "href" : "https://resource-controller.cloud.ibm.com/v1/resource_groups/d0bd962c2eea48d8b18b5d5e82e2fc60",
      "id" : "d0bd962c2eea48d8b18b5d5e82e2fc60"
   },
   "cpu" : {
      "cores" : 2,
      "frequency" : 2000,
      "architecture" : "amd64"
   },
   "network_interfaces" : [
      {
         "subnet" : {
            "crn" : "crn:v1:bluemix:public:is:us-south-1:a/843f59bad5553123f46652e9c43f9e89::subnet:649543e3-f627-45e3-9c98-b65bfb69545e",
            "name" : "subnet1",
            "id" : "649543e3-f627-45e3-9c98-b65bfb69545e",
            "href" : "https://us-south.iaas.cloud.ibm.com/v1/subnets/649543e3-f627-45e3-9c98-b65bfb69545e"
         },
         "name" : "eth0",
         "primary_ipv4_address" : "10.240.1.4",
         "resource_type" : "network_interface",
         "href" : "https://us-south.iaas.cloud.ibm.com/v1/instances/1c8a08fd-7b32-4ca5-8716-4903ef3f93c8/network_interfaces/2ea8e558-27b8-4943-88d5-ff3d02d77da8",
         "id" : "2ea8e558-27b8-4943-88d5-ff3d02d77da8"
      }
   ],
   "profile" : {
      "name" : "cc1-2x4",
      "crn" : "crn:v1:bluemix:public:is:us-south:a/843f59bad5553123f46652e9c43f9e89::instance-profile:cc1-2x4",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/instance/profiles/cc1-2x4"
   },
   "vpc" : {
      "name" : "vpc-1",
      "crn" : "crn:v1:bluemix:public:is::a/843f59bad5553123f46652e9c43f9e89::vpc:a23f85bc-efd8-472c-be4f-51cc73c77f91",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/vpcs/a23f85bc-efd8-472c-be4f-51cc73c77f91",
      "id" : "a23f85bc-efd8-472c-be4f-51cc73c77f91"
   },
   "zone" : {
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1",
      "name" : "us-south-1"
   },
   "href" : "https://us-south.iaas.cloud.ibm.com/v1/instances/1c8a08fd-7b32-4ca5-8716-4903ef3f93c8",
   "memory" : 4,
   "image" : {
      "crn" : "crn:v1:bluemix:public:is:us-south:::image:cfdaf1a0-5350-4350-fcbc-97173b510843",
      "name" : "ubuntu-18.04-amd64",
      "id" : "cfdaf1a0-5350-4350-fcbc-97173b510843"
   },
   "name" : "vsi-1",
   "primary_network_interface" : {
      "subnet" : {
         "crn" : "crn:v1:bluemix:public:is:us-south-1:a/843f59bad5553123f46652e9c43f9e89::subnet:649543e3-f627-45e3-9c98-b65bfb69545e",
         "name" : "subnet1",
         "href" : "https://us-south.iaas.cloud.ibm.com/v1/subnets/649543e3-f627-45e3-9c98-b65bfb69545e",
         "id" : "649543e3-f627-45e3-9c98-b65bfb69545e"
      },
      "name" : "eth0",
      "id" : "2ea8e558-27b8-4943-88d5-ff3d02d77da8",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/instances/1c8a08fd-7b32-4ca5-8716-4903ef3f93c8/network_interfaces/2ea8e558-27b8-4943-88d5-ff3d02d77da8",
      "primary_ipv4_address" : "10.240.1.4",
      "resource_type" : "network_interface"
   },
   "created_at" : "2019-09-21T05:52:27.680Z",
   "status" : "pending"
}
```

Set an environment variable equal to the Primary Network Interface ID
````
network_interface="2ea8e558-27b8-4943-88d5-ff3d02d77da8"
````
 
### Step 6: Create a floating IP for the new Virtual Server

**Command**
```
curl -X POST $api_endpoint/v1/floating_ips?version=$api_version \
  -H "Authorization:$iam_token" \
  -d '{
        "name": "fip-1",
        "target": {
            "id":"'$network_interface'"
        }
      }' | json_pp
```

**Output**
```
{
   "id" : "6d097d84-1fbd-48ad-9181-a37788486edc",
   "name" : "fip-1",
   "status" : "pending",
   "target" : {
      "id" : "2ea8e558-27b8-4943-88d5-ff3d02d77da8",
      "name" : "eth0",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/instances/0397e0e0-0f3d-40c7-babd-be17685cb607/network_interfaces/2ea8e558-27b8-4943-88d5-ff3d02d77da8",
      "resource_type" : "network_interface",
      "primary_ipv4_address" : "10.240.1.4"
   },
   "address" : "169.61.224.246",
   "created_at" : "2019-09-21T05:55:53Z",
   "crn" : "crn:v1:bluemix:public:is:us-south-1:a/843f59bad5553123f46652e9c43f9e89::floating-ip:6d097d84-1fbd-48ad-9181-a37788486edc",
   "href" : "https://us-south.iaas.cloud.ibm.com/v1/floating_ips/6d097d84-1fbd-48ad-9181-a37788486edc",
   "zone" : {
      "name" : "us-south-1",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1"
   }
}
```

Make a note of the floating IP address created, in the above example the floating IP address is `169.61.224.246`

6. Enable communitcation through ports 22 and 8080.

SSH access to the VSI is through port 22. The application used to access the MongoDB uses port 8080. Open communication for these ports by updating the Inbound Rules of the VPC's Security Group.

**Port 8080 Command**
```
curl -X POST "$api_endpoint/v1/security_groups/2d364f0a-a870-42c3-a554-000001999214/rules?version=$api_version" \
     -H "Authorization: $iam_token" \
     -d "{ \"direction\" : \"inbound\", \
           \"protocol\" : \"tcp\", \
           \"port_max\" : 8080, \
           \"port_min\" : 8080 \
         }" | json_pp
```
**Output**
```
{
   "protocol" : "tcp",
   "port_min" : 8080,
   "port_max" : 8080,
   "direction" : "inbound",
   "id" : "b597cff2-38e8-4e6e-999d-000006201056",
   "ip_version" : "ipv4"
}
```

**Port 22 Command**
```
curl -X POST "$api_endpoint/v1/security_groups/2d364f0a-a870-42c3-a554-000001999214/rules?version=$api_version" \
     -H "Authorization: $iam_token" \
     -d "{ \"direction\" : \"inbound\", \
           \"protocol\" : \"tcp\", \
           \"port_max\" : 22, \
           \"port_min\" : 22 \
         }" | json_pp
```
**Output**
```
{
   "ip_version" : "ipv4",
   "port_min" : 22,
   "port_max" : 22,
   "id" : "b597cff2-38e8-4e6e-999d-000006201058",
   "direction" : "inbound",
   "protocol" : "tcp"
}
```

## Links
- [Creating VPC resources using the REST APIs](https://cloud.ibm.com/docs/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis)
- [IBM Cloud VPC API Reference](https://cloud.ibm.com/apidocs/vpc-on-classic)

