# Use GUI, CLI or API to deploy a VPC and connect a VPC Application to a MongoDB using a private endpoint.

### Purpose
Here we will illustrate the use of the [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started#overview) to create the required infrastructure components for a Virtual Private Cloud needed for this scenario.

## Section 1 - Install the IBM Cloud CLI tools and login to IBM Cloud
1. Install the [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started#overview)
2. Open a Terminal/Command Prompt 
   *Use this terminal/command prompt throughout this excercise*
3. Install the IBM Cloud CLI [Infrastructure plugin](https://cloud.ibm.com/docs/infrastructure/vpc/cli-reference.html)

    **Command**
   ```
   ibmcloud plugin install infrastructure-service
   ```
   **Output**
   ```
   Looking up 'infrastructure-service' from repository 'IBM Cloud'...
   Plug-in 'infrastructure-service 0.2.0' found in repository 'IBM Cloud'
   Attempting to download the binary file...
   14.10 MiB / 14.10 MiB [================================================================================================================] 100.00% 2s
   14784336 bytes downloaded
   Installing binary...
   OK
   Plug-in 'infrastructure-service 0.2.0' was successfully installed into /Users/dlentz/.bluemix/plugins/infrastructure-service. Use 'ibmcloud plugin show infrastructure-service' to show its details.
   ```
4. Login to the IBM Cloud:
   - For a federated account use single sign on:  
     `ibmcloud login -sso`  
   - Otherwise use the default login:  
     `ibmcloud login`  
   - If you have an API Key, use --apikey:  
     `ibmcloud login --apikey [your API Key]`  


### Section 2 - Create a new SSH key
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
4. Add your new SSH Key to your IBM Cloud account

    **Command**
    ```
    ibmcloud is key-create vpcCompute-key @vpcCompute.pub
                           <key name>     <SSH file>

    Parameters:
    Key Name - Unique name for your SSH Key
    SSH file - The filename containing the public key
    ```
    **Output**
    ```
    Creating key vpcCompute-key under account IBM - Mac's Org as user <Your IBM Cloud ID>...
                 
    ID            636f6d70-0000-0001-0000-000000142747   
    Name          vpcCompute-key   
    Type          rsa   
    Length        2048   
    FingerPrint   SHA256:04aNt/xusyQx4rf33leMNYEzABsw+wqYpfxlAp4z+qI   
    Key           ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDe7jK7fOHrKU81z6cWELXAOHDZHGqqUZHd4ox3De0HjeO8NPqgKmn0mSR1oZZiWqR7QQi1RR0MQ/bzlD2e35gozFbbtI/S45m2KErCYvd9jbgpPYj7X9TEoLLM3ediCqdY6VJC+abDtX+C0mzQ1Hp2wuxQ07AoVWkiRMCLVFH6zbwHsvaAWc8NG0XXjA8XKLtcZYTqDxvl9mN/xwv032KwcuuvIVGdu5ctHImHjqTAiVtkPbs5VLS0+NkPiPcwU0h9kQYtlzfMav0BnHOgYrgIGUXG8RQZ0Rqi38EWeQWIErnN9eVF6LCSwbzX1Am8qyc/9p1XeHnbebRq28/atHur root   
                    
    Created       now
    ```


### Section 3. Create a new VPC instance, SubNet and Virtual Server

1. Create a new VPC for this scenerio by entering the following command:

    **Command**
    ```
    ibmcloud is vpc-create vpc-1 --resource-group-id d0bd962c2eea48d8b18b5d5e82e2fc60
                       <vpc name>       <Optional Resource Group ID>

    
    Parameters:
    vpc name - The name for the VPC
    Resource Group ID - Optional parameter used to specify which resource group to create the VPC in
    ```
    **Output**
    ```
    Creating vpc vpc-1 in resource group d0bd962c2eea48d8b18b5d5e82e2fc60 under account Phillip Trent's Account as user msalas@us.ibm.com...

    ID                       4e557ffd-e34f-4ded-8215-3149817643f8
    Name                     vpc-1
    Classic access           no
    Default network ACL      allow-all-network-acl-4e557ffd-e34f-4ded-8215-3149817643f8(f2faed50-4c32-4075-b7ea-4406e71ebbc0)
    Default security group   seminar-blazing-thank-primal-yearning(2d364f0a-a870-42c3-a554-000001997366)
    Resource group           d0bd962c2eea48d8b18b5d5e82e2fc60
    Created                  2019-09-20T15:11:17-04:00
    Status                   available
    ```
    - [Command Documentation](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#vpc-create)
    - If you need to create a Resource Group, follow [these instructions](https://console.bluemix.net/docs/resources/resourcegroups.html#creating-a-resource-group)


2. Create a base SubNet in the new VPC

    **Command**
    ```
    ibmcloud is subnet-create subnet1 4e557ffd-e34f-4ded-8215-3149817643f8 --zone us-south-1 --ipv4-cidr-block 10.240.0.0/24
                              <subnet name>  <VPC ID>                             <Zone>     <IPV4 Range>

    Parameters:
    Subnet Name - The name to give the new subnet
    VPC ID - The ID of the VPC created in the previous step
    Zone -  The IBM Cloud zone to create the Subnet in
    IPV4 Range - The block of IPV4 addresses that can be used in the subnet
    ```
    **Output**
    ```
    Creating subnet subnet1 under account Phillip Trent's Account as user msalas@us.ibm.com...

    ID                  b00b8899-10e4-45cf-a5ce-f101e055d58a
    Name                subnet1
    IPv4 CIDR           10.240.0.0/24
    Address available   251
    Address total       256
    ACL                 allow-all-network-acl-4e557ffd-e34f-4ded-8215-3149817643f8(f2faed50-4c32-4075-b7ea-4406e71ebbc0)
    Public Gateway      -
    Created             2019-09-20T17:49:36-04:00
    Status              pending
    Zone                us-south-1
    VPC                 vpc-1(4e557ffd-e34f-4ded-8215-3149817643f8)
    ```
    - [Command Documentation](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#subnet-create)
    - [Finding a Zone](#appendix-1---finding-a-zone) in the Appendix for more info)
  
3. Create a Virtual Server in the new VPC SubNet

    **Command**
    ```
    ibmcloud is instance-create vsi-1 4e557ffd-e34f-4ded-8215-3149817643f8 us-south-1 cc1-2x4 b00b8899-10e4-45cf-a5ce-f101e055d58a --image-id 7eb4e35b-4257-56f8-d7da-326d85452591 --key-ids 636f6d70-0000-0001-0000-000000142747
                                <name>          <VPC ID>                     <Zone>     <Profile> <Subnet ID>                         <Image ID>                                      <SSH Key ID>
    
    Parameters:
    Name - The name of the new Virtual Server
    VPC ID - The ID of the VPC created previously.
    Zone - The Zone to create the Virtual Server in (see [Finding a Zone](Appendix 1 - Finding a Zone) in the Appendix for more info)
    Subnet ID - The ID of the Subnet created previously.
    Image ID - The ID of the Image to use in the new Virtual Server
    SSH Key ID - The ID of the SSH Key created previously.
    ```
    **Output**
    ```
    Creating instance vsi-1 in resource group vpc under account Phillip Trent's Account as user msalas@us.ibm.com...

    ID                       fd884586-97c0-4b14-ac7c-0c32af39e4d8
    Name                     vsi-1
    Status                   pending
    Profile                  cc1-2x4
    vCPU architecture        amd64
    vCPUs                    2
    Memory                   4
    Image                    ubuntu-16.04-amd64(7eb4e35b-4257-56f8-d7da-326d85452591)
    VPC                      vpc-1(4e557ffd-e34f-4ded-8215-3149817643f8)
    Zone                     us-south-1
    Resource group           d0bd962c2eea48d8b18b5d5e82e2fc60
    Created                  2019-09-20T18:04:19.726-04:00

    Primary interface        primary(0b68b740-09fb-416f-ba38-f413e327ca1f)
    Primary address          10.240.0.8
    Attached Floating IP:    No Floating IP attached
    ```
    - [Command Documentation](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#instance-create)
    - The `Key Oject ID` is obtained from the output of the `ibmcloud is key-create` command executing on Section 2.
    - To see a list of available Image IDs, enter the command
      `ibmcloud is images` [(Reference)](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#images-cli)
    - To see a list of available Profiles, enter the command
      `ibmcloud is instance-profiles` [(Reference)](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#instance-profiles)
    
4. Reserve a floating IP address for the new Virtual Server

    **Command**
    ```
    ibmcloud is floating-ip-reserve fip-1 --zone us-south-1
                                    <name>           <zone>

    Parameters:
    Name - The name of the floating IP reservation
    Zone - The zone to reserve the IP in
    ```
    **Output**
    ```
    Creating floating IP fip-1 in resource group vpc under account Phillip Trent's Account as user msalas@us.ibm.com...

    ID               3468c995-8492-41ba-a3b6-11d22d07642a
    Address          169.61.244.97
    Name             fip-1
    Target           -
    Target type      -
    Target IP
    Created          2019-09-20T18:08:47-04:00
    Status           pending
    Zone             us-south-1
    Resource group   -
    ```
    - [Command Documentation](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#floating-ip-reserve)

5. Assign the reserved floating IP address to the new Virtual Server

    **Command**
    ```
    ibmcloud is instance-network-interface-floating-ip-add fd884586-97c0-4b14-ac7c-0c32af39e4d8 0b68b740-09fb-416f-ba38-f413e327ca1f 3468c995-8492-41ba-a3b6-11d22d07642a
                                                           <Virtual Server Instance ID>         <NIC ID>                             <Floating IP ID>

    Parameters:
    Virtual Server Instance ID - The ID of the Virtual Server instance created previously.
    NIC ID - The ID of the Primary Network Interface card on the Virtual Server (See 'Finding the NIC ID' below)
    Floating IP ID - The ID of the Floating IP created previously.
    ```
    **Output**
    ```
    Creating floatingip 3468c995-8492-41ba-a3b6-11d22d07642a for instance fd884586-97c0-4b14-ac7c-0c32af39e4d8 under account Phillip Trent's Account as user msalas@us.ibm.com...

    ID               3468c995-8492-41ba-a3b6-11d22d07642a
    Address          169.61.244.97
    Name             fip-1
    Target           primary(0b68b740-09fb-416f-ba38-f413e327ca1f)
    Target type      network_interface (10.240.0.8)
    Target IP        10.240.0.8
    Created          2019-09-20T18:08:47-04:00
    Status           available
    Zone             us-south-1
    Resource group   -
    ```
   - [Command Documentation](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#instance-network-interface-floating-ip-add)
   - See [Finding the NIC ID](#appendix-2---finding-the-nic-id) in the Appendix for help identifying the correct NIC ID to use.

6. Enable communitcation through ports 22 and 8080.

SSH access to the VSI is through port 22. The application used to access the MongoDB uses port 8080. Open communication for these ports by updating the Inbound Rules of the VPC's Security Group.

**Port 8080 Command**
    ```
    ibmcloud is security-group-rule-add 2d364f0a-a870-42c3-a554-000001997366 inbound tcp --port-max 8080 --port-min 8080
                                                  <Security Group ID>        <Dir>   <Proto> <Max Port>       <Min Port>

    Parameters:
    Security Group ID - The Security Group ID of the VPC created previously.
    Dir - Direction of traffic
    Proto - Protocol
    Max / Min Port - Port
    ```
   **Output**
    ```
    Creating rule for security group 2d364f0a-a870-42c3-a554-000001997366 under account Phillip Trent's Account as user msalas@us.ibm.com...

    ID                     b597cff2-38e8-4e6e-999d-000006199280
    Direction              inbound
    IP version             ipv4
    Protocol               tcp
    Min destination port   8080
    Max destination port   8080
    Remote                 -
    ```
   **Port 22 Command**
    ```
    ibmcloud is security-group-rule-add 2d364f0a-a870-42c3-a554-000001997366 inbound tcp --port-max 22 --port-min 22
    ```
    **Output**
    ```
    Creating rule for security group 2d364f0a-a870-42c3-a554-000001997366 under account Phillip Trent's Account as user msalas@us.ibm.com...

    ID                     b597cff2-38e8-4e6e-999d-000006199192
    Direction              inbound
    IP version             ipv4
    Protocol               tcp
    Min destination port   22
    Max destination port   22
    Remote                 -
    ```
   - [Command Documentation](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#instance-network-interface-floating-ip-add)
   - See [Finding the NIC ID](#appendix-2---finding-the-nic-id) in the Appendix for help identifying the correct NIC ID to use.

## Links
- [Creating VPC resources using the CLI](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)
- [IBM Cloud VPC CLI Reference](https://cloud.ibm.com/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference)

## Appendix
### Appendix 1 - Finding a Zone
To see a list of available Zones, use the following commands:

**Command 1**
```
ibmcloud is regions
```
**Output**
```
Listing regions under account IBM - Mac's Org as user <Your IBM Cloud ID>...
Name       Endpoint                              Status   
us-south   https://us-south.iaas.cloud.ibm.com   available
```

**Command 2**
```
ibmcloud is zones <region name>
```
*Where <region name> is the name of a region obtained in Command 1 above*
**Output**
```
Listing zones in region us-south under account IBM - Mac's Org as user <Your IBM Cloud ID>...
Name         Region     Status   
us-south-3   us-south   available   
us-south-1   us-south   available   
us-south-2   us-south   available  
```


### Appendix 2 - Finding the NIC ID
To find the NIC ID, issue the following command to get details about the Virtual Server. The NIC ID is shown in the output next to 'Primary Intf' and only includes the text within the parenthesis. For example, the NIC ID in the output below is `2aa29acc-4241-4b28-8377-c8c57fc9b538`
   
**Command**
```
ibmcloud is instance <Virtual Server Instance ID>
```
**Output**
```
Getting instance e72f6bfe-4271-4731-bedd-90f438cb8d6a under account IBM - Mac's Org as user <Your IBM Cloud ID>...
                    
ID                e72f6bfe-4271-4731-bedd-90f438cb8d6a   
Name              vpc-monitor-app   
Profile           c-2x4   
CPU Arch          amd64   
CPU Cores         2   
CPU Frequency     2000   
Memory            4   
Primary Intf      primary(2aa29acc-4241-4b28-8377-c8c57fc9b538)   
Primary Address   10.240.10.12   
Image             ubuntu-16.04-amd64(7eb4e35b-4257-56f8-d7da-326d85452591)   
Status            running   
Created           6 minutes ago   
VPC               vpc-monitor-demo(4f81711c-1cf3-4ee6-aff4-2af1f0bc2794)   
Zone              us-south-1   
```
