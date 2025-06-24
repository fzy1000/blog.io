---
title: AWS Learning 一
date: 2025-06-24 23:58:31
category: AWS
tags: AWS
---

<h2>AWS</h2>

<h3>Amazon CloudFront<h3/>
Amazon CloudFront delivers your content through a worldwide network of edge locations. When a user requests content that is being served with CloudFront, the request is routed to the location that provides the lowest latency. So that content is delivered with the best possible performance. CloudFront speeds up the distribution of your content by routing each user request through the AWS backbone network to the edge location that can best serve your content.

<h3>特点<h3/>
1. 轻量级：资源消耗小
2. 开源
3. 支持弹性伸缩
4. 支持负载均衡：IPVS



<h2>Container Services<h2/>

the three main categories of compute are virtual machines (VMs), containers, and serverless.



<h3>Container<h3/>

A container is a standardized unit that packages your code and its dependencies. This package is designed to run reliably on any platform, because the container creates its own independent environment. With containers, workloads can be carried from one place to another, such as from development to production or from on-premises environments to the cloud.

Difference between VMs and containers

<img src="/blog.io/img/VM和Container.jpeg">

Compared to virtual machines (VMs), containers share the same operating system and kernel as the host that they are deployed on.

<h3>Orchestrating containers<h3/>

In AWS, containers can run on EC2 instances. For example, you might have a large instance and run a few containers on that instance. Although running one instance is uncomplicated to manage, it lacks high availability and scalability. Most companies and organizations run many containers on many EC2 instances across several Availability Zones.

If you're trying to manage your compute at a large scale, you should consider the following:

•
How to place your containers on your instances

•
What happens if your container fails

•
What happens if your instance fails

•
How to monitor deployments of your containers

This coordination is handled by a container orchestration service. AWS offers two container orchestration services: Amazon Elastic Container Service (Amazon ECS) and Amazon Elastic Kubernetes Service (Amazon EKS).

<h3>Managing containers with Amazon ECS<h3/>
Amazon ECS is an end-to-end container orchestration service that helps you spin up new containers. With Amazon ECS, your containers are defined in a task definition that you use to run an individual task or a task within a service. You have the option to run your tasks and services on a serverless infrastructure that's managed by another AWS service called AWS Fargate. Alternatively, for more control over your infrastructure, you can run your tasks and services on a cluster of EC2 instances that you manage.


<img src="/blog.io/img/ECS.png">

<h3>Using Kubernetes with Amazon EKS<h3/>

Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services. By bringing software development and operations together by design, Kubernetes created a rapidly growing ecosystem that is very popular and well established in the market.

If you already use Kubernetes, you can use Amazon EKS to orchestrate the workloads in the AWS Cloud. Amazon EKS is a managed service that you can use to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane or nodes. Amazon EKS is conceptually similar to Amazon ECS, but with the following differences:

* In Amazon ECS, the machine that runs the containers is an EC2 instance that has an ECS agent installed and configured to run and manage your containers. This instance is called a container instance. In Amazon EKS, the machine that runs the containers is called a worker node or Kubernetes node.

* An ECS container is called a task. An EKS container is called a pod.

* Amazon ECS runs on AWS native technology. Amazon EKS runs on Kubernetes.

If you have containers running on Kubernetes and want an advanced orchestration solution that can provide simplicity, high availability, and fine-grained control over your infrastructure, Amazon EKS could be the tool for you.



<h3>Go serverless <h3/>
* There are no servers to provision or manage.
* It scales with usage.
* You never pay for idle resources.
* Availability and fault tolerance are built in.


<h3>AWS Fargate<h3/>
Fargate abstracts the EC2 instance so that you're not required to manage the underlying compute infrastructure. However, with Fargate, you can use all the same Amazon ECS concepts, APIs, and AWS integrations. It natively integrates with IAM and Amazon Virtual Private Cloud (Amazon VPC). With native integration with Amazon VPC, you can launch Fargate containers inside your network and control connectivity to your applications.

AWS Fargate is a purpose-built serverless compute engine for containers. AWS Fargate scales and manages the infrastructure, so developers can work on what they do best, application development. It achieves this by allocating the right amount of compute. This eliminates the need to choose and manage EC2 instances, cluster capacity, and scaling. Fargate supports both Amazon ECS and Amazon EKS architecture and provides workload isolation and improved security by design.

<h3>AWS Lambda<h3/>
With Lambda, you can run code without provisioning or managing servers. You can run code for virtually any type of application or backend service. This includes data processing, real-time stream processing, machine learning, WebSockets, IoT backends, mobile backends, and web applications like your employee directory application!

Lambda runs your code on a high availability compute infrastructure and requires no administration from the user. You upload your source code in one of the languages that Lambda supports, and Lambda takes care of everything required to run and scale your code with high availability. There are no servers to manage. You get continuous scaling with subsecond metering and consistent performance.



<h3>CIDR notation<h3/>

192.168.1.30 is a single IP address. If you want to express IP addresses between the range of 192.168.1.0 and 192.168.1.255, how can you do that?

One way is to use CIDR notation. CIDR notation is a compressed way of representing a range of IP addresses. Specifying a range determines how many IP addresses are available to you.


<h3>Amazon VPC<h3/>

A virtual private cloud (VPC) is an isolated network that you create in the AWS Cloud, similar to a traditional network in a data center. When you create an Amazon VPC, you must choose three main factors:

+ Name of the VPC

+ Region where the VPC will live – A VPC spans all the Availability Zones within the selected Region.

+ IP range for the VPC in CIDR notation – This determines the size of your network. Each VPC can have up to five CIDRs: one primary and four secondaries for IPv4. Each of these ranges can be between /28 (in CIDR notation) and /16 in size.

Using this information, AWS will provision a network and IP addresses for that network.

<h3>Internet gateway<h3/>
To activate internet connectivity for your VPC, you must create an internet gateway. Think of the gateway as similar to a modem. Just as a modem connects your computer to the internet, the internet gateway connects your VPC to the internet. Unlike your modem at home, which sometimes goes down or offline, an internet gateway is highly available and scalable. After you create an internet gateway, you attach it to your VPC.

<h3>Virtual private gateway<h3/>
A virtual private gateway connects your VPC to another private network. When you create and attach a virtual private gateway to a VPC, the gateway acts as anchor on the AWS side of the connection. On the other side of the connection, you will need to connect a customer gateway to the other private network. A customer gateway device is a physical device or software application on your side of the connection. When you have both gateways, you can then establish an encrypted virtual private network (VPN) connection between the two sides.



<h3>AWS Direct Connect<h3/>

To establish a secure physical connection between your on-premises data center and your Amazon VPC, you can use AWS Direct Connect. With AWS Direct Connect, your internal network is linked to an AWS Direct Connect location over a standard Ethernet fiber-optic cable. This connection allows you to create virtual interfaces directly to public AWS services or to your VPC.


<img src="/blog.io/img/直连.png">

<h3>Main route table<h3/>

When you create a VPC, AWS creates a route table called the main route table. A route table contains a set of rules, called routes, that are used to determine where network traffic is directed. AWS assumes that when you create a new VPC with subnets, you want traffic to flow between them. Therefore, the default configuration of the main route table is to allow traffic between all subnets in the local network. The following rules apply to the main route table:

+ You cannot delete the main route table.
+ You cannot set a gateway route table as the main route table.
+ You can replace the main route table with a custom subnet route table.
+ You can add, remove, and modify routes in the main route table.
+ You can explicitly associate a subnet with the main route table, even if it's already implicitly associated.

<h3>Custom route tables<h3/>

The main route table is used implicitly by subnets that do not have an explicit route table association. However, you might want to provide different routes on a per-subnet basis for traffic to access resources outside of the VPC. For example, your application might consist of a frontend and a database. You can create separate subnets for the resources and provide different routes for each of them.

If you associate a subnet with a custom route table, the subnet will use it instead of the main route table. Each custom route table that you create will have the local route already inside it, allowing communication to flow between all resources and subnets inside the VPC. You can protect your VPC by explicitly associating each new subnet with a custom route table and leaving the main route table in its original default state.


<img src="/blog.io/img/route_table.png">

<h3>Secure subnets with network access control lists<h3/>

Think of a network access control list (network ACL) as a virtual firewall at the subnet level. A network ACL lets you control what kind of traffic is allowed to enter or leave your subnet. You can configure this by setting up rules that define what you want to filter. 

Network ACLs are considered stateless, so you need to include both the inbound and outbound ports used for the protocol. If you don't include the outbound range, your server would respond but the traffic would never leave the subnet.

<h3>Secure EC2 instances with security groups<h3/>
The next layer of security is for your EC2 instances. Here, you can create a virtual firewall called a security group. The default configuration of a security group blocks all inbound traffic and allows all outbound traffic.
By default, a security group only allows outbound traffic. To allow inbound traffic, you must create inbound rules.

security groups are stateful. That means that they will remember if a connection is originally initiated by the EC2 instance or from the outside, and temporarily allow traffic to respond without modifying the inbound rules.
If you want your EC2 instance to accept traffic from the internet, you must open up inbound ports. If you have a web server, you might need to accept HTTP and HTTPS requests to allow that type of traffic into your security group. You can create an inbound rule that will allow port 80 (HTTP) and port 443 (HTTPS)



<h2>With security groups, everything is blocked by default so you can only use allow rules. Whereas with network ACLs, everything is allowed by default but you can use both allow and deny rules. <h2/>



<h3>Storage Types<h3/>
AWS storage services are grouped into three categories: file storage, block storage, and object storage. In file storage, data is stored as files in a hierarchy. In block storage, data is stored in fixed-size blocks. And in object storage, data is stored as objects in buckets.

<img src="/blog.io/img/AWS_Storage.png">

<h3>File storage<h3/>

You might be familiar with file storage if you have interacted with file storage systems like Windows File Explorer or Finder on macOS. Files are organized in a tree-like hierarchy that consist of folders and subfolders.

Each file has metadata such as file name, file size, and the date the file was created. The file also has a path, for example, computer/Application_files/Cat_photos/cats-03.png. When you need to retrieve a file, your system can use the path to find it in the file hierarchy.

File storage is ideal when you require centralized access to files that must be easily shared and managed by multiple host computers. Typically, this storage is mounted onto multiple hosts, and requires file locking and integration with existing file system communication protocols.


<h3>Block storage<h3/>

File storage treats files as a singular unit, but block storage splits files into fixed-size chunks of data called blocks that have their own addresses. Each block is an individual piece of data storage. Because each block is addressable, blocks can be retrieved efficiently. Think of block storage as a more direct route to access the data.

When data is requested, the addresses are used by the storage system to organize the blocks in the correct order to form a complete file to present back to the requestor. Besides the address, no additional metadata is associated with each block

If you want to change one character in a file, you just change the block, or the piece of the file, that contains the character. This ease of access is why block storage solutions are fast and use less bandwidth.

<h3>Use cases for block storage<h3>

Because block storage is optimized for low-latency operations, it is a preferred storage choice for high-performance enterprise workloads and transactional, mission-critical, and I/O-intensive applications.

<h3>Object storage<h3/>

In object storage, files are stored as objects. Objects, much like files, are treated as a single, distinct unit of data when stored. However, unlike file storage, these objects are stored in a bucket using a flat structure, meaning there are no folders, directories, or complex hierarchies. Each object contains a unique identifier. This identifier, along with any additional metadata, is bundled with the data and stored.

<h3>Use cases for object storage<h3/>

With object storage, you can store almost any type of data, and there is no limit to the number of objects stored, which makes it readily scalable. Object storage is generally useful when storing large or unstructured data sets.


