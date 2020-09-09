# VPC Networking

## Task 1. Explore the default network
Each Google Cloud project has a default network with subnets, routes, and firewall rules.

### View the subnets
The default network has a subnet in each Google Cloud region.

- Run the following command in the terminal:
```bash
gcloud compute networks subnets list
```
Notice the default network with its subnets. Each subnet is associated with a Google Cloud region and a private RFC 1918 CIDR block for its internal IP addresses range and a gateway.

### View the routes
Routes tell VM instances and the VPC network how to send traffic from an instance to a destination, either inside the network or outside Google Cloud. Each VPC network comes with some default routes to route traffic among its subnets and send traffic from eligible instances to the internet.

- To view all routes, run the following command:
```bash
gcloud compute routes list
```
Notice that there is a route for each subnet and one for the Default internet gateway (0.0.0.0/0). These routes are managed for you, but you can create custom static routes to direct some packets to specific destinations. For example, you can create a route that sends all outbound traffic to an instance configured as a NAT gateway.

### View the firewall rules
Each VPC network implements a distributed virtual firewall that you can configure. Firewall rules allow you to control which packets are allowed to travel to which destinations. Every VPC network has two implied firewall rules that block all incoming connections and allow all outgoing connections.

- To see all firewalls, run the following command:
```bash
gcloud compute firewall-rules list
```
Notice that there are 4 Ingress firewall rules for the default network:
	- default-allow-icmp
	- default-allow-rdp
	- default-allow-ssh
	- default-allow-internal

These firewall rules allow ICMP, RDP, and SSH ingress traffic from anywhere (0.0.0.0/0) and all TCP, UDP, and ICMP traffic within the network (10.128.0.0/9). The Targets, Filters, Protocols/ports, and Action columns explain these rules.

### Delete the Firewall rules

1. To delete all firewall rules, run the following command:
```bash
gcloud compute firewall-rules delete default-allow-icmp \
default-allow-rdp default-allow-ssh default-allow-internal
```

### Delete the default network
1. To delete the default network, run the following command:
```bash
gcloud compute networks delete default
```
Wait for the network to be deleted before continuing.

2. To view the routes, run the following command:
```bash
gcloud compute routes list
```
Notice that there are no routes.

3. To view firewalls, run the following command in the terminal:
```bash
gcloud compute firewall-rules list
```

Notice that there are no firewall rules.
> Without a VPC network, there are no routes!

Without a VPC network, you cannot create VM instances, containers, or App Engine applications.

- True
- False

### Try to create a VM instance
Verify that you cannot create a VM instance without a VPC network.

- To create a VM instance, run the following command in the terminal:
```bash
gcloud compute instances create instance-1
```

Notice the error.
> As expected, you cannot create a VM instance without a VPC network!

## Task 2. Create an auto mode network
You have been tasked to create an auto mode network with two VM instances. Auto mode networks are easy to set up and use because they automatically create subnets in each region. However, you don't have complete control over the subnets created in your VPC network, including regions and IP address ranges used. Feel free to explore more considerations for choosing an auto mode network, but for now, assume that you are using the auto mode network for prototyping purposes.

### Create an auto mode VPC network with firewall rules
<!-- On the Navigation menu (Navigation menu), click VPC network > VPC networks.
Click Create VPC network.
For Name, type mynetwork
For Subnet creation mode, click Automatic. Auto mode networks create subnets in each region automatically.
For Firewall rules, select all available rules. -->

1. Create an auto mode network named 'mynetwork'.
```bash
gcloud compute networks create mynetwork

gcloud compute firewall-rules create mynetwork-allow-icmp --network mynetwork --direction=INGRESS \
--priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp

gcloud compute firewall-rules create mynetwork-allow-internal --network mynetwork --direction=INGRESS \
--priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all

gcloud compute firewall-rules create mynetwork-allow-rdp --network mynetwork --direction=INGRESS \
--priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389

gcloud compute firewall-rules create mynetwork-allow-ssh --network mynetwork --direction=INGRESS \
--priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22

```

These are the same standard firewall rules that the default network had. The deny-all-ingress and allow-all-egress rules are also displayed, but you cannot select or disable them because they are implied. These two rules have a lower Priority (higher integers indicate lower priorities) so that the allow ICMP, internal, RDP, and SSH rules are considered first.

2. When the new network is ready, check the subnets. Notice that a subnet was created for each region.
```bash
gcloud compute networks subnets list
```

3. Record the IP address range for the subnets in us-central1 and europe-west1. These will be referred to in the next steps.

> Tip: If you ever delete the default network, you can quickly re-create it by creating an auto mode network as you just did.

### Create a VM instance in us-central1
Create a VM instance in the us-central1 region. Selecting a region and zone determines the subnet and assigns the internal IP address from the subnet's IP address range.

1. To create the VM instance, run the following command:
```bash
gcloud compute instances create mynet-us-vm \
--network mynetwork --machine-type=n1-standard-1 \
--zone=us-central1-c
```

2. Verify that the Internal IP for the new instance was assigned from the IP address range for the subnet in us-central1 (10.128.0.0/20).

The Internal IP should be 10.128.0.2 because 10.128.0.1 is reserved for the gateway, and you have not configured any other instances in that subnet.

### Create a VM instance in europe-west1
Create a VM instance in the europe-west1 region.

1. To create the VM instance, run the following command:
```bash
gcloud compute instances create mynet-eu-vm \
--network mynetwork --machine-type=n1-standard-1 \
--zone=europe-west1-c
```

2. Verify that the Internal IP for the new instance was assigned from the IP address range for the subnet in europe-west1 (10.132.0.0/20).

The Internal IP should be 10.132.0.2 because 10.132.0.1 is reserved for the gateway, and you have not configured any other instances in that subnet.

> The External IP addresses for both VM instances are ephemeral. If an instance is stopped, any ephemeral external IP addresses assigned to the instance are released back into the general Compute Engine pool and become available for use by other projects. When a stopped instance is started again, a new ephemeral external IP address is assigned to the instance. Alternatively, you can reserve a static external IP address, which assigns the address to your project indefinitely until you explicitly release it.

### Verify connectivity for the VM instances
The firewall rules that you created with mynetwork allow ingress SSH and ICMP traffic from within mynetwork (internal IP) and outside that network (external IP).

1. Check the IP addresses of mynet-eu-vm. Note the external and internal IP addresses.
```bash
gcloud compute instances list
```

2. SSH into mynet-us-vm.
```bash
gcloud compute ssh mynet-us-vm --zone=us-central1-c
```
Accept all defaults.
> You can SSH because of the allow-ssh firewall rule, which allows incoming traffic from anywhere (0.0.0.0/0) for tcp:22. The SSH connection works seamlessly because Compute Engine generates an SSH key for you and stores it in one of the following locations:

> - By default, Compute Engine adds the generated key to project or instance metadata.
> - If your account is configured to use OS Login, Compute Engine stores the generated key with your user account.

> Alternatively, you can control access to Linux instances by creating SSH keys and editing public SSH key metadata.

3. To test connectivity to mynet-eu-vm's internal IP, run the following command, replacing mynet-eu-vm's internal IP:
```bash
ping -c 3 <Enter mynet-eu-vm's internal IP here>
```
You can ping mynet-eu-vm's internal IP because of the allow-internal firewall rule.

4. Repeat the same test by running the following:
```bash
ping -c 3 mynet-eu-vm
```

You can ping mynet-eu-vm by its name because VPC networks have an internal DNS service that allows you to address instances by their DNS names instead of their internal IP addresses. This is very useful because the internal IP address can change when you delete and re-create an instance.

5. To test connectivity to mynet-eu-vm's external IP, run the following command, replacing mynet-eu-vm's external IP:
```bash
ping -c 3 <Enter mynet-eu-vm's external IP here>
```

Which firewall rule allows the ping to mynet-eu-vm's external IP address?

- mynetwork-allow-icmp
- mynetwork-allow-ssh
- mynetwork-allow-rdp
- mynetwork-allow-internal

> You can SSH to mynet-us-vm and ping mynet-eu-vm's internal and external IP addresses as expected. Alternatively, you can SSH to mynet-eu-vm and ping mynet-us-vm's internal and external IP addresses, which also works.

6. Exit mynet-us-vm.
```bash
exit
```

### Convert the network to a custom mode network
The auto mode network worked great so far, but you have been asked to convert it to a custom mode network so that new subnets aren't automatically created as new regions become available. This could result in overlap with IP addresses used by manually created subnets or static routes, or could interfere with your overall network planning.

```bash
 gcloud compute networks update mynetwork --switch-to-custom-subnet-mode
```
Confirm by pressing Enter.
> Converting an auto mode network to a custom mode network is an easy task, and it provides you with more flexibility. We recommend that you use custom mode networks in production.

## Task 3. Create custom mode networks
You have been tasked to create two additional custom networks, managementnet and privatenet, along with firewall rules to allow SSH, ICMP, and RDP ingress traffic and VM instances as shown in this diagram (with the exception of vm-appliance):

### Create the managementnet network

1. To create the managementnet network, run the following command:
```bash
gcloud compute networks create managementnet --subnet-mode=custom
```

2. To create the managementsubnet-us subnet, run the following command:
```bash
gcloud compute networks subnets create managementsubnet-us \
--network=managementnet --region=us-central1 --range=10.130.0.0/20
```

### Create the privatenet network

1. To create the privatenet network, run the following command:
```bash
gcloud compute networks create privatenet --subnet-mode=custom
```

2. To create the privatesubnet-us subnet, run the following command:
```bash
gcloud compute networks subnets create privatesubnet-us \
--network=privatenet --region=us-central1 --range=172.16.0.0/24
```

3. To create the privatesubnet-eu subnet, run the following command:
```bash
gcloud compute networks subnets create privatesubnet-eu \
--network=privatenet --region=europe-west1 --range=172.20.0.0/20
```

4. To list the available VPC networks, run the following command:
```bash
gcloud compute networks list
```

The output should look like this (do not copy; this is example output):
```bash
NAME           SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
managementnet  CUSTOM       REGIONAL
mynetwork      CUSTOM       REGIONAL
privatenet     CUSTOM       REGIONAL
```

5. To list the available VPC subnets (sorted by VPC network), run the following command:
```bash
gcloud compute networks subnets list --sort-by=NETWORK
```

The output should look like this (do not copy; this is example output):
```bash
NAME                REGION                   NETWORK        RANGE
managementsubnet-us us-central1              managementnet  10.130.0.0/20
mynetwork           asia-northeast1          mynetwork      10.146.0.0/20
mynetwork           us-west1                 mynetwork      10.138.0.0/20
mynetwork           southamerica-east1       mynetwork      10.158.0.0/20
mynetwork           europe-west4             mynetwork      10.164.0.0/20
mynetwork           asia-east1               mynetwork      10.140.0.0/20
mynetwork           europe-north1            mynetwork      10.166.0.0/20
mynetwork           asia-southeast1          mynetwork      10.148.0.0/20
mynetwork           us-east4                 mynetwork      10.150.0.0/20
mynetwork           europe-west1             mynetwork      10.132.0.0/20
mynetwork           europe-west2             mynetwork      10.154.0.0/20
mynetwork           europe-west3             mynetwork      10.156.0.0/20
mynetwork           australia-southeast1     mynetwork      10.152.0.0/20
mynetwork           asia-south1              mynetwork      10.160.0.0/20
mynetwork           us-east1                 mynetwork      10.142.0.0/20
mynetwork           us-central1              mynetwork      10.128.0.0/20
mynetwork           northamerica-northeast1  mynetwork      10.162.0.0/20
privatesubnet-eu    europe-west1             privatenet     172.20.0.0/20
privatesubnet-us    us-central1              privatenet     172.16.0.0/24
```

The managementnet and privatenet networks only have the subnets that you created because they are custom mode networks. mynetwork is also a custom mode network, but it started out as an auto mode network, resulting in subnets in each region.

### Create the firewall rules for managementnet
Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network.

1. To create the managementnet-allow-icmp-ssh-rdp firewall rule, run the following command:
```bash
gcloud compute firewall-rules create managementnet-allow-icmp-ssh-rdp \
--direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW \
--rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```

### Create the firewall rules for privatenet
Create the firewall rules for privatenet network.

1. To create the privatenet-allow-icmp-ssh-rdp firewall rule, run the following command:
```bash
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp \
--direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW \
--rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```
The output should look like this (do not copy; this is example output):
```bash
NAME                           NETWORK     DIRECTION  PRIORITY  ALLOW                 DENY
privatenet-allow-icmp-ssh-rdp  privatenet  INGRESS    1000      icmp,tcp:22,tcp:3389
```

2. To list all the firewall rules (sorted by VPC network), run the following command:
```bash
gcloud compute firewall-rules list --sort-by=NETWORK
```

The output should look like this (do not copy; this is example output):
```bash
NAME                              NETWORK        DIRECTION  PRIORITY  ALLOW                        
managementnet-allow-icmp-ssh-rdp  managementnet  INGRESS    1000      icmp,tcp:22,tcp:3389
mynetwork-allow-icmp              mynetwork      INGRESS    1000      icmp
mynetwork-allow-internal          mynetwork      INGRESS    65534     all                         
mynetwork-allow-rdp               mynetwork      INGRESS    1000      tcp:3389
mynetwork-allow-ssh               mynetwork      INGRESS    1000      tcp:22
privatenet-allow-icmp-ssh-rdp     privatenet     INGRESS    1000      icmp,tcp:22,tcp:3389
```
The firewall rules for mynetwork network have been created for you. You can define multiple protocols and ports in one firewall rule (privatenet and managementnet) or spread them across multiple rules (default and mynetwork).

Next, create two VM instances:

- managementnet-us-vm in managementsubnet-us
- privatenet-us-vm in privatesubnet-us

### Create the managementnet-us-vm instance
Create the managementnet-us-vm instance using the Cloud Console.

1. To create managementnet-us-vm instance, run the following:
```bash
gcloud compute instances create managementnet-us-vm --zone=us-central1-c \
--machine-type=f1-micro --subnet=managementsubnet-us
```

### Create the privatenet-us-vm instance
1. To create the privatenet-us-vm instance, run the following command:
```bash
gcloud compute instances create privatenet-us-vm --zone=us-central1-c \
--machine-type=f1-micro --subnet=privatesubnet-us
```
The output should look like this (do not copy; this is example output):
```bash
NAME              ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
privatenet-us-vm  us-central1-c  f1-micro                   172.16.0.2   34.66.197.202  RUNNING
```

2. To list all the VM instances (sorted by zone), run the following command:
```bash
gcloud compute instances list --sort-by=ZONE
```
The output should look like this (do not copy; this is example output):
```bash
NAME                 ZONE            MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
mynet-eu-vm          europe-west1-c  n1-standard-1              10.132.0.2   34.76.115.41   RUNNING
managementnet-us-vm  us-central1-c   f1-micro                   10.130.0.2   35.239.68.123  RUNNING
mynet-us-vm          us-central1-c   n1-standard-1              10.128.0.2   35.202.101.52  RUNNING
privatenet-us-vm     us-central1-c   f1-micro                   172.16.0.2   34.66.197.202  RUNNING
```

There are three instances in us-central1-c and one instance in europe-west1-c. However, these instances are spread across three VPC networks (managementnet, mynetwork, and privatenet), with no instance in the same zone and network as another. In the next task, you explore the effect this has on internal connectivity.

> You can explore more networking information on each VM instance by clicking the nic0 link in the Internal IP column. The resulting network interface details page shows the subnet along with the IP CIDR range, the firewall rules and routes that apply to the instance, and other network analysis.

## Task 4. Explore the connectivity across networks
Explore the connectivity between the VM instances. Specifically, determine the effect of having VM instances in the same zone versus having instances in the same VPC network.

### Ping the external IP addresses
Ping the external IP addresses of the VM instances to determine whether you can reach the instances from the public internet.

1. Note the external IP addresses for mynet-eu-vm, managementnet-us-vm, and privatenet-us-vm from the last command.

2. To SSH into mynet-us-vm, run the following command:
```bash
gcloud compute ssh mynet-us-vm --zone=us-central1-c
```

3. To test connectivity to mynet-eu-vm's external IP, run the following command, replacing mynet-eu-vm's external IP:
```bash
ping -c 3 <Enter mynet-eu-vm's external IP here>
```
This should work!

4. To test connectivity to managementnet-us-vm's external IP, run the following command, replacing managementnet-us-vm's external IP:
```bash
ping -c 3 <Enter managementnet-us-vm's external IP here>
```
This should work!

5. To test connectivity to privatenet-us-vm's external IP, run the following command, replacing privatenet-us-vm's external IP:
```bash
ping -c 3 <Enter privatenet-us-vm's external IP here>
```
This should work!

> You can ping the external IP address of all VM instances, even though they are in either a different zone or VPC network. This confirms that public access to those instances is only controlled by the ICMP firewall rules that you established earlier.

6. Exit mynet-us-vm.
```bash
exit
```

### Ping the internal IP addresses
Ping the internal IP addresses of the VM instances to determine whether you can reach the instances from within a VPC network.

Which instances should you be able to ping from mynet-us-vm using internal IP addresses?

- privatenet-us-vm
- mynet-eu-vm
- managementnet-us-vm

1. To view the addresses of the VM instances, run the following command:
```bash
gcloud compute instances list --sort-by=ZONE
```
Note the internal IP addresses for mynet-eu-vm, managementnet-us-vm, and privatenet-us-vm.

2. Return to the SSH terminal for mynet-us-vm.
```bash
gcloud compute ssh mynet-us-vm --zone=us-central1-c
```

3. To test connectivity to mynet-eu-vm's internal IP, run the following command, replacing mynet-eu-vm's internal IP:
```bash
ping -c 3 <Enter mynet-eu-vm's internal IP here>
```

> You can ping the internal IP address of mynet-eu-vm because it is on the same VPC network as the source of the ping (mynet-us-vm), even though both VM instances are in separate zones, regions, and continents!

4. To test connectivity to managementnet-us-vm's internal IP, run the following command, replacing managementnet-us-vm's internal IP:
```bash
ping -c 3 <Enter managementnet-us-vm's internal IP here>
```

> This should not work, as indicated by a 100% packet loss!

5. To test connectivity to privatenet-us-vm's internal IP, run the following command, replacing privatenet-us-vm's internal IP:
```bash
ping -c 3 <Enter privatenet-us-vm's internal IP here>
```

> This should not work either, as indicated by a 100% packet loss! You cannot ping the internal IP address of managementnet-us-vm and privatenet-us-vm because they are in separate VPC networks from the source of the ping (mynet-us-vm), even though they are all in the same zone, us-central1-c.

## Task 5. Review
In this lab, you explored the default network and determined that you cannot create VM instances without a VPC network. Thus, you created a new auto mode VPC network with subnets, routes, firewall rules, and two VM instances and tested the connectivity for the VM instances. Because auto mode networks aren't recommended for production, you converted the auto mode network to a custom mode network. Next, you created two more custom mode VPC networks with firewall rules and VM instances using the Cloud Console and the gcloud command line. Then you tested the connectivity across VPC networks, which worked when pinging external IP addresses but not when pinging internal IP addresses.

VPC networks are by default isolated private networking domains. Therefore, no internal IP address communication is allowed between networks, unless you set up mechanisms such as VPC peering or VPN.