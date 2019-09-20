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
 

Store the API endpoint in a variable so it can be reused later.

**Command**
````
rias_endpoint=https://us-south.iaas.cloud.ibm.com
````
To verify that this variable was saved, run ````echo $rias_endpoint```` and make sure the response is not empty.
 


### Step 2: Create a new VPC instance
The command below will create a new VPC named 'cloudant-vpc' in the IBM Cloud. The Resource group parameter is optional.

**Command**
```
curl -X POST \
  $rias_endpoint/v1/vpcs?version=2019-01-01 \
  -H "Authorization: $iam_token" \
  -H "User-Agent: IBM_One_Cloud_IS_UI/2.4.0" \
  -H "Content-Type: application/json" \
  -H "Cache-Control: no-cache" \
  -H "accept: application/json" \
  -d "{\"name\":\"cloudant-vpc\",\"resource_group\":{\"id\":\"2f3e837a095943de958accf8ccb9bc19\"}}" | json_pp
  ```
  - If you need to create a Resource Group, follow [these instructions](https://cloud.ibm.com/docs/resources?topic=resources-rgs#creating-a-resource-group)


**Output**
```
{
   "href" : "https://us-south.iaas.cloud.ibm.com/v1/vpcs/e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
   "id" : "e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
   "created_at" : "2019-02-01T17:36:57Z",
   "default_network_acl" : {
      "name" : "allow-all-network-acl-e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
      "id" : "b50300aa-2014-426e-ade7-e5b134f91e41",
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/network_acls/b50300aa-2014-426e-ade7-e5b134f91e41"
   },
   "resource_group" : {
      "id" : "03bf4fd296f24eb19f0eced2a8fb6366",
      "href" : "https://resource-manager.bluemix.net/v1/resource_groups/03bf4fd296f24eb19f0eced2a8fb6366"
   },
   "is_default" : false,
   "classic_peered" : false,
   "default_security_group" : {
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/security_groups/2d364f0a-a870-42c3-a554-000001217857",
      "name" : "recluse-footbath-omit-ranch-refining-panther",
      "id" : "2d364f0a-a870-42c3-a554-000001217857"
   },
   "name" : "cloudant-vpc",
   "crn" : "crn:v1:bluemix:public:is:us-south:a/26a3d1a386bd2cc44df1997eb7ac0ef1::vpc:e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
   "status" : "available"
}
```

Set an environment variable equal to the VPC ID returned from IBM Cloud:
````
vpc_id="e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18"
````

### Step 3: Create a subnet in the new VPC

**Command**
```
curl -X POST \
  "https://us-south.iaas.cloud.ibm.com/v1/subnets" \
  -H "Authorization: $iam_token" \
  -H "User-Agent: IBM_One_Cloud_IS_UI/2.4.0" \
  -H "Content-Type: application/json" \
  -H "Cache-Control: no-cache" \
  -H "accept: application/json" \
  -d "{\"zone\":{\"name\":\"us-south-1\"},\"resource_group\":{\"id\":\"03bf4fd296f24eb19f0eced2a8fb6366\"},\"ip_version\":\"ipv4\",\"name\":\"cloudant-subnet\",\"ipv4_cidr_block\":\"10.240.1.0/24\",\"vpc\":{\"id\":\"$vpc_id\"}}"  
````


**Output**
````
{
  "id": "61641de2-37ce-497a-9390-6172eea22a69",
  "name": "cloudant-subnet",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/61641de2-37ce-497a-9390-6172eea22a69",
  "ipv4_cidr_block": "10.240.1.0/24",
  "available_ipv4_address_count": 251,
  "total_ipv4_address_count": 256,
  "ip_version": "ipv4",
  "zone": {
    "name": "us-south-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1"
  },
  "vpc": {
    "id": "e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
    "crn": "crn:v1:bluemix:public:is:us-south:a/26a3d1a386bd2cc44df1997eb7ac0ef1::vpc:e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
    "name": "cloudant-vpc",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18"
  },
  "status": "pending",
  "created_at": "2019-02-01T17:52:55Z",
  "network_acl": {
    "id": "b50300aa-2014-426e-ade7-e5b134f91e41",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/network_acls/b50300aa-2014-426e-ade7-e5b134f91e41",
    "name": "allow-all-network-acl-e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18"
  }
}
````


Set an environment variable equal to the SubNet ID returned from IBM Cloud:
````
vpc_subnet_id="61641de2-37ce-497a-9390-6172eea22a69"
````


### Step 4: Create an SSH Key
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
curl -X POST $rias_endpoint/v1/keys?version=2019-01-01 \
  -H "Authorization:$iam_token" \
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




### Step 5: Create a new Virtual Server in your VPC

**Command**
````
curl -X POST $rias_endpoint/v1/instances?version=2019-01-01 \
  -H "Authorization:$iam_token" \
  -d '{
        "name": "cloudant",
        "type": "virtual",
        "zone": {
          "name": "us-south-1"
        },
        "vpc": {
          "id": "'$vpc_id'"
        },
        "primary_network_interface": {
          "port_speed": 1000,
          "subnet": {
            "id": "'$vpc_subnet_id'"
          }
        },
        "keys":[{"id": "'$ssh_key_id'"}],
        "flavor": {
          "name": "c-2x4"
         },
        "image": {
          "id": "7eb4e35b-4257-56f8-d7da-326d85452591"
         },
        "userdata": ""
      }'
````

**Output**
````
{
  "cpu": {
    "architecture": "amd64",
    "cores": 2,
    "frequency": 2000
  },
  "created_at": "2019-02-04T15:33:37.134Z",
  "crn": "crn:v1:bluemix:public:is:us-south-1:a/26a3d1a386bd2cc44df1997eb7ac0ef1::instance:9c6e42e3-3661-40b2-bce3-f60e323533ba",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/instances/9c6e42e3-3661-40b2-bce3-f60e323533ba",
  "id": "9c6e42e3-3661-40b2-bce3-f60e323533ba",
  "image": {
    "crn": "crn:v1:bluemix:public:is:us-south:a/33c5711b8afbf7fd809a4529de613a08::image:7eb4e35b-4257-56f8-d7da-326d85452591",
    "href": "https://us-south.iaas.cloud.ibm.com:443/v1/images/7eb4e35b-4257-56f8-d7da-326d85452591",
    "id": "7eb4e35b-4257-56f8-d7da-326d85452591",
    "name": "ubuntu-16.04-amd64"
  },
  "memory": 4,
  "name": "cloudant",
  "network_interfaces": [
    {
      "href": "https://us-south.iaas.cloud.ibm.com/v1/instances/9c6e42e3-3661-40b2-bce3-f60e323533ba/network_interfaces/56a8ca12-4ab8-4f5e-9dc1-6715db514790",
      "id": "56a8ca12-4ab8-4f5e-9dc1-6715db514790",
      "name": "statutory-aspect-gumminess-twiddling-barrel-deploy",
      "primary_ipv4_address": "10.240.1.12",
      "subnet": {
        "crn": "crn:v1:bluemix:public:is:us-south-1:a/26a3d1a386bd2cc44df1997eb7ac0ef1::subnet:61641de2-37ce-497a-9390-6172eea22a69",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/61641de2-37ce-497a-9390-6172eea22a69",
        "id": "61641de2-37ce-497a-9390-6172eea22a69",
        "name": "cloudant-subnet"
      }
    }
  ],
  "primary_network_interface": {
    "href": "https://us-south.iaas.cloud.ibm.com/v1/instances/9c6e42e3-3661-40b2-bce3-f60e323533ba/network_interfaces/56a8ca12-4ab8-4f5e-9dc1-6715db514790",
    "id": "56a8ca12-4ab8-4f5e-9dc1-6715db514790",
    "name": "statutory-aspect-gumminess-twiddling-barrel-deploy",
    "primary_ipv4_address": "10.240.1.12",
    "subnet": {
      "crn": "crn:v1:bluemix:public:is:us-south-1:a/26a3d1a386bd2cc44df1997eb7ac0ef1::subnet:61641de2-37ce-497a-9390-6172eea22a69",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/61641de2-37ce-497a-9390-6172eea22a69",
      "id": "61641de2-37ce-497a-9390-6172eea22a69",
      "name": "cloudant-subnet"
    }
  },
  "profile": {
    "crn": "crn:v1:bluemix:public:is:us-south:a/26a3d1a386bd2cc44df1997eb7ac0ef1::instance-profile:c-2x4",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/instance/profiles/c-2x4",
    "name": "c-2x4"
  },
  "status": "pending",
  "vpc": {
    "crn": "crn:v1:bluemix:public:is::a/26a3d1a386bd2cc44df1997eb7ac0ef1::vpc:e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpcs/e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
    "id": "e8c341fc-4fc9-4bb0-94cc-e8e2fb15ea18",
    "name": "cloudant-vpc"
  },
  "zone": {
    "href": "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1",
    "name": "us-south-1"
  }
}
````

Set an environment variable equal to the Primary Network Interface ID
````
network_interface="56a8ca12-4ab8-4f5e-9dc1-6715db514790"
````
 
### Step 6: Create a floating IP for the new Virtual Server

**Command**
````
curl -X POST $rias_endpoint/v1/floating_ips?version=2019-01-01 \
  -H "Authorization:$iam_token" \
  -d '{
        "name": "my-server-floatingip",
        "target": {
            "id":"'$network_interface'"
        }
      }
'
````

**Output**
````
{
   "name" : "my-server-floatingip",
   "href" : "https://us-south.iaas.cloud.ibm.com/v1/floating_ips/5da08ea3-49f5-4a84-98c4-81575511c793",
   "status" : "pending",
   "created_at" : "2019-02-04T15:38:05Z",
   "crn" : "",
   "target" : {
      "id" : "56a8ca12-4ab8-4f5e-9dc1-6715db514790",
      "subnet" : {
         "name" : "cloudant-subnet",
         "href" : "https://us-south.iaas.cloud.ibm.com/v1/subnets/61641de2-37ce-497a-9390-6172eea22a69",
         "id" : "61641de2-37ce-497a-9390-6172eea22a69"
      },
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/instances/f158f4cc-626f-4593-9e3d-620ccea0f967/network_interfaces/56a8ca12-4ab8-4f5e-9dc1-6715db514790",
      "name" : "statutory-aspect-gumminess-twiddling-barrel-deploy",
      "primary_ipv4_address" : "10.240.1.12"
   },
   "address" : "169.61.244.118",
   "id" : "5da08ea3-49f5-4a84-98c4-81575511c793",
   "zone" : {
      "href" : "https://us-south.iaas.cloud.ibm.com/v1/regions/us-south/zones/us-south-1",
      "name" : "us-south-1"
   }
}
````

Make a note of the floating IP address created, in the above example the floating IP address is ````169.61.244.118````
     
## Links
- [Creating VPC resources using the REST APIs](https://cloud.ibm.com/docs/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis)
- [IBM Cloud VPC API Reference](https://cloud.ibm.com/docs/infrastructure/vpc/api-doc-wrapper.html#api-reference-for-vpc)

