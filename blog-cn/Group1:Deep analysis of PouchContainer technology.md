# Deep analysis of PouchContainer's rich container technology

PouchContainer is an efficient, open-source, lightweight Enterprise Rich Container Engine Technology developed by the Alibaba group. This technology presents the characteristics of strong isolation, high portability, and low resource consumption for a container. With the help of PouchContainer, corporations can rapidly deploy their business into containers and improve the utilization of physical resources in their data centers under hyperscale at the same time.

PouchContainer originates from the thoughts of Alibaba's engineers to the problems they had in their business scenarios. These engineers have dedicated enormous time and efforts in delivering internet applications, which could be proved by the characteristics of the PouchContainer listed above. Thanks to the mass scale of Alibaba, this technology embraced its first opportunity to be fully tested on its ability to support business at Nov 11th after it comes to the world.  After declared as an open source project, this new technology is positioned to serve for the public and to help enterprises quickly realize the containerization of their business.

![image1.png | center | 180x60](https://cdn.yuque.com/lark/0/2018/png/65333/1526059437042-3ae3f269-7795-44b2-a618-e6af68dc9f21.png "")

When it first comes to container technology, Alibaba had an amazing amount of backlogs inside at that time. How to quickly containerize these backlogs is the key problem of this technology when it was first promoted inside the company. Nowadays, the open source container technology is becoming more and more popular. Many companies may still be worrying about how to deal with the same problem.  In the cloud native field, many of the advanced concepts promoted by the CNCF Foundation are based on business containerization. If enterprises fall behind this trend, they will be overwhelmed by the follow-up problems brought by container management, let alone enjoying the benefits of open source projects like Service Mesh.

Through seven years of practical experience, Alibaba's container technology PouchContainer uses facts to deliver this information to the industry – rich containers are the technology of choice for rapid containerization of backlogs.

## What is Rich Container

A rich container is a container model used by enterprises to package applications and implement business containerization. This model assists with IT technicians so that they could package business applications with little effort. Business applications packaged with rich container technology can serve for two purposes:

* Rapid delivery of business with container images
* Container environment can still be compatible with the original operation system

From the technical point of view, the rich container provides an effective path to help the business to package operation and maintenance kits, system services, etc. required by more services in addition to the business application itself in a single container image; at the same time, compared to the simpler single process container, the rich container at the level of the process organization structure, also has a huge change: the container at runtime will automatically run daemons such as systemd. In this way, applications in rich container mode have the ability to run exactly the same as on a physical machine without changing any business code or operation code. That is to say, this is a more general application-oriented model.

In other words, rich containers are not intrusive at the development and operation level while protecting the efficiency of business delivery, and thus have the ability to help IT staff focus more on business innovation.

## Applicable Scene

The application of rich containers is extremely wide. It can be said that almost all of the company's backlogs can adopt rich containers as the first choice for containerization. Before the popularity of container technology, for nearly two decades, enterprise IT services were running in bare metal or virtual machines. The stable operation of the business owes operation and maintenance work a lot of credits, which can be subdivided into "infrastructure operation and maintenance" and "business operation and maintenance". All applications run on physical resources while all stably running services rely on monitoring systems, log services and other operation and maintenance systems. We have reasons to believe that in the process of business containerization, enterprises must not ignore the operation and maintenance system, otherwise the consequences can be imagined.

Therefore, in the process of containerization of the business, it is necessary to consider the scenario where the new system should be compatible with the old operation and maintenance system, and this is within the scope of use of the PouchContainer rich container technology.

## Technical Details of Rich Containers

We know that the business can be compatible with the original operation and maintenance system via rich containers. But how on earth is the rich container technology realized? The figure below clearly describes the internals of the rich container technology.

![image.png | center | 747x368](https://cdn.yuque.com/lark/0/2018/png/65333/1526059474453-02e322a1-f33f-4de2-a238-26e0501b3106.png "")

Rich container technology is 100% compatible with the community's OCI image, and  file system of the image is used as the container's rootfs when the container starts. In the operating mode, at the functional level, in addition to the internal running process, the hook method (prestart hook and poststop hook) to start and stop the container is also provided.

### Internal Processes of Rich Containers

If we look at PouchContainer's rich container technology from the perspective of internally running processes, we can classify the internal running processes into four categories:

* Init process where pid = 1
* CMD of images
* System service processes inside a container
* User-defined operation and maintenance components

#### Init Process Where pid = 1

The most obvious difference between rich container technology and traditional container technology is when container runs an init process inside, the traditional container (such as docker container) uses the CMD specified in the container image as the process of pid=1 in the container. Rich container mode can be run from three init processes:

* systemd
* sbin/init
* dumb-init

As we all know, the traditional container as a stand-alone operating environment, the internal process management has certain drawbacks: for example, the zombie process cannot be recycled, causing the container to consume too many processes and consume extra memory; for example, the system service process inside the container cannot be managed friendly, resulting in the lack of basic capabilities in some business applications, such as cron system services, syslogd system services, etc.; for example, the main reason to not supporting the normal operation of some system applications is that some system applications need to call systemd to install RPM packages......

There is no doubt that the rich container init process in the operation and maintenance mode can solve the above problems, bringing a better experience to the application. The init process is designed to be able to wait for the demise process, which is easy to solve the zombie process that was born during the operation of the business process in the above figure. At the same time, managing system service is one of the tasks it is designed for. The design of init process will provide most of the ablilities that are useful for the users, and lay a solid foundation for the operation and maintenance system.

#### CMD of the Container's Image

The CMD of the container image is the business that we want to run inside the container in the traditional sense. For example, when a user encapsulates a Golang business system into an image, it will definitely specify the startup command of the business system as CMD in the Dockerfile, so as to ensure that the CMD command will run the service from the future when the container is started with the image.

Of course, the CMD of the container image represents the business application and is the core component of the entire rich container. All the operation and maintenance adaptations are to ensure more stable operation of the business application.

#### Service Process in the Container

Server programming has evolved for decades, and many business system development models are based on the Linux operating system on bare metal or the Linux environment in a virtualized environment. Over time, many business application development paradigms interact with system service processes very frequently. For example, an application written in the Java programming language is likely to configure log management via log4j, or via log4j.properties configuration redirects application logs to syslogd in the running environment. If there is no syslogd running in the application runtime environment, it is very likely to affect the startup and operation of the business; also, business applications need to be managed by crond Periodic tasks. If there is no crond system daemon in the application runtime environment, it is impossible for the business application to configure periodic tasks through crontab. For example, the sshd system service system inside the container can quickly help the operation and maintenance engineers to make rapid progress, locate and solve problems, and so on.

PouchContainer's rich container model, having taken industry's large number of applications and system service delivery applications into consideration, results in the design of init process inside the rich container armed with the ability to manage multiple system service processes.

#### User-defined Operation and Maintenance Components

The existence of system services can assist the normal operation of the business, but in many cases, this is not enough. The enterprise itself should be responsible for the infrastructure, the operation and maintenance components that come with the applications, and at the same time plays a role in ensuring the smooth-running of the business. For example, the operation and maintenance team needs to be unified to configure the monitoring component for the business application; this team must manage the application log inside the container through a customized log agent;  they also need  to customize their own basic operation and maintenance tool so that The application operating environment can be required to comply with internal audit requirements.

Because of the init processes inside the rich container, the user-defined operation and maintenance components can run as normal and stable as usual, providing operation and maintenance capabilities.

### Starting and Stopping Hook for the Rich Container

Eventually, the task process running inside the rich container can ensure that the application runtime is stable and normal. However, for the operation and maintenance team, the content  should be maintained is often much morer than that with single process running. In general, the operational responsibilities also need to cover the environmental preparation work before the run-time, and the after-work after the end of running. For applications, this is the prestart hook and poststop hook we usually refer to.

PouchContainer's rich container mode allows users to easily specify the application's start and stop execution hooks: prestart hook and poststop hook. The operation and maintenance team specifies the prestart hook, which can help the application to perform some initialization operations in the container that meet the operation and maintenance requirements before running, such as: initializing the network routing table, obtaining the application execution permission, and downloading the certificate required for the runtime. The same team specifies the poststop hook, which can help the application to perform unified after-work management after the end of the operation or abnormal exit. For example, the intermediate data is cleaned up for the pure environment at the next startup; if it is abnormally exited, the error can be reported immediately which is able to meet operational requirements.

We can find that the start and stop hooks inside the rich container have raised the ability of the container to be maintained to another level, which greatly increased the flexibility and management ability of the operation and maintenance team.

## Conclusion

Benefiting from the mass scale of Alibaba's business, PouchContainer has helped many large Internet companies to containerize their online businesses. There is no doubt that rich container technology is the most practical technology that is not invasive for application development and application operation and maintenance. The open source PouchContainer hopes that the technology can benefit the industry, helping a large number of enterprises to win their own time in the containerization of the backlogs, quickly embrace the cloud native technology, and make great strides toward digital transformation.







