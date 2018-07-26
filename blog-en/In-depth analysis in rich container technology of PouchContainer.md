# In-depth analysis in rich container technology of PouchContainer

PouchContainer is an open-source project published by Alibaba Group as a highly-efficient and lightweight enterprise rich container engine technology of which the features include but not limited to strong isolation, high portability and low resource occupancy.It can help enterprises realize the rapid containerization of inventory business and improve the utilization rate of physical resources of data centers on a large scale.

PouchContainer comes from the internal framework of alibaba.At the very beginning, PouchContainer cost the engineers much energy to design its framework  to solve the question of how to guarantee a stable running environment for internet applications. And now, the technical features of PouchContainer such as  strong isolation and rich container are the best provement to their efforts. PouchContainer's support to alibaba's business has been tested on an unprecedented scale during the “Double 11” shopping festival of ali. After open source, this container has become a technology that benefits the public and positions itself as "helping enterprises realize rapid containerization of inventory business". 

<div data-type="alignment" data-value="center" style="text-align:center">
  <div data-type="p">
    <div id="4vggns" data-type="image" data-display="block" data-align="center" data-src="https://cdn.yuque.com/lark/0/2018/png/65333/1526059437042-3ae3f269-7795-44b2-a618-e6af68dc9f21.png" data-width="222">
      <img src="https://cdn.yuque.com/lark/0/2018/png/65333/1526059437042-3ae3f269-7795-44b2-a618-e6af68dc9f21.png" width="222" />
    </div>
  </div>
</div>

Ali had a surprisingly large scale of inventory business when container technology first came into the sight of engineers. How to quickly containerize the inventory  business through technology was the main problem when applying ali container technology in ali. Today, open source container technology is gaining popularity and we believe that there are certain enterprises troubled by the difficulty of containerizing inventory business when using such technology. In the cloud native field, most of the advanced concepts advocated by CNCF foundation are based on business containerization. If the enterprise does not choose the right entrance to the containerization when applying the cloud native, it is even more out of the question to consider the subsequent container layout, Service Mesh and other  open source technology dividends of the industry.
 
After the seven-year practical experience, PouchContainer--alibaba container technology, has used facts to convey such message to the whole industry that rich container is the preferred technology for rapid containerization of an enterprise's inventory business.

## 1. What is rich container

Rich container is a container pattern adopted in the enterprises during the procedure of packaging business application and realizing business containerization. This pattern can help IT technicians package business applications with little effort. Business applications packaged with rich container technology can achieve the following two purposes:

* container mirroring enables rapid business delivery
* container environment is compatible with the original operation and maintenance system of the enterprise

From a technical perspective, a rich container provides an efficient path to help package more operational and maintenance packages, system services and other components required by the business in a single container image in addition to the business application itself. At the same time, compared with simpler single-process containers, the rich container also has a huge change in the organization structure of the process: the systemd and other stewardship processes are automatically run inside the container when it is running. In this way, applications in rich container mode have the ability to run exactly as they would on a physical machine without changing any business code or operational code. Arguably, this is a more generic "application-oriented" model.
 
In other words, while ensuring the efficiency of business delivery, a rich container has no invasiveness to the application at the development and operational level, thus enabling IT technicians to focus more on business innovation.

## 2. Applicable scenarios

Rich containers can be used in a wide range of scenarios. It can be said that almost all the inventory business of an enterprise can adopt rich containers as the first choice for containerization. Before the popularity of container technology, corporate IT services ran in bare metal or virtual machines for nearly two decades. The stable operation of enterprise business is largely attributed to operation work. If subdivided, it includes "infrastructure operation" and "business operation". All applications run on physical resources and all business stability depends on operational maintenance systems such as monitoring systems and logging services. There is reason to believe, then, that in the process of business containerization, enterprises must not ignore the operation system, or the consequences are too serious to imagine.

Therefore, in the process of containerization of inventory business, it is necessary to consider scenarios that are compatible with the original operation  system of enterprises, which are within the scope of application of PouchContainer's rich container technology.

## 3. The implementation of rich container

Since the rich container can make business compatiable with original operation system. So what kind of technology is used to implement rich container technology? The figure below clearly describes the internals.

![image.png | center | 747x368](https://cdn.yuque.com/lark/0/2018/png/65333/1526059474453-02e322a1-f33f-4de2-a238-26e0501b3106.png)

Rich container is 100 percent compatible with the community's OCI image, and the image's file system will be used as the container's rootfs when the container starts. In the operating mode, the functional level, in addition to the internal running process, also includes the hook method (prestart hook and poststop hook) when the container starts and stops.

### 3.1 Internal running processes of rich container
If we treat PouchContainer's rich container technology from the perspective of internally running processes, we can classify the internal running processes into four categories:

* the init process (pid=1)
* the 'CMD' of the container mirror
* the internal service process of container
* user defined custom operation components

#### 3.1.1 The init process

Unlike traditional container (docker etc.) those who choose the specific CMD of container image as the process whose pid=1, there is a init process(pid=1) running in rich container, that is the most obvious difference between rich container and traditional container.PouchContainer's rich container mode could be choose from three types below:

* systemd
* sbin/init
* dumb-init

As we all know, as a stand-alone running environment,the traditional container's internal process management has some disadvantages. For example, the zombie process cannot be recycled, causing the container to consume too many processes and consume extra memory; The system service process inside the container cannot be managed friendly, which lead to the lack of fundamental capabilities such as cron system services, syslogd system services in some business applications; Failed to provide the essential environment for some system applications: some system applications need to call systemd to install RPM packages......

All the issues listed above can be solved by rich container's init process with no doubt, and meanwhile performing a better experience.The init process is capable to waiting for perished processes, so that zombie process can no longer exists.Another basic feature of the init process is system service management which provides almost 50% fundamental traditional operation capabilities for users.It is a solid foundation for the whole operation architecture.

#### 3.1.2 The CMD of the container image

The CMD of a container image is what we want to run in the container.For example, a user will set the start command of the business system as the CMD in Dockerfile when containerizing a Golang business system, so that the startup of the business system can be gurateened by executing this CMD command when the image is loaded by container in future.

It's obverse that the CMD command of container image is the key part of rich container, all the adaption of operation is moving around to make our business applications run more stable.

#### 3.1.3 The internal service process of container system

Server side programming has evolved for decades, most of server side programming modes are based on Linux running above physical machine or virtual environment.Over time, many business application development paradigms interact with system service processes very frequently.For example, Java applications are likely to use log4j to configure log management; It is also possible to redirect application logs to syslogd in the running environment through log4j.properties configuration. If there is no syslogd in the application environment, it will affect the startup of the service; The business application needs to use crond to manage the periodic tasks required by the business, If there is no crond in the application running environment, the periodic task configuration of the business application will be invalid. The internal sshd of the container can help the operation engineers to quickly Enter the application runtime, locate and solve problems...

PouchContainer's rich container model takes the frequent interactions between applications and system services into account, and the init process inside the rich container has the ability to natively manage multiple system service processes.

#### 3.1.4 User defined custom operation components
Although system services can assist the normal operation of the business, it is still not enough in many cases. The enterprise itself needs to equip the infrastructure and applications with operation components to escort the business.For example, the operation team not only needs to uniformly configure the monitoring component for the business application but also need to manage the application log inside the container through a customized log agent; besides, they also need to customize the basic operation tool to make the application running environment conform internal auditing standards.

Because of the init process inside the rich container, the user-defined operation components can run healthily and stably, providing continuous operation capabilities.

### 3.2 Hooks to start/stop rich container
The task process running inside the rich container can ensure that the application runtime is stable and normal, but the operation team is not only responsible for the stable running of a single runtime, but also for the environment initialization before the runtime and clean-ups after the runtime.For applications, it is the prestart hook and poststop hook that we usually refer to.

PouchContainer's rich container mode is very convenient for users to configure the prestart hook and poststart hook of applications. Prestart hook can help the application to perform some customerize initialization operations before running, such as: initializing the network routing table, obtaining the application execution permission, and downloading the certificate required for the runtime. Poststop hook can help the application to perform unified clean-up tasks when the task finished exited with exception. For example, the intermediate data is cleaned up to provide a pure environment at the next startup; Errors can be reported immediately when application crashes.

It shows that the start hook and stop hook of rich container makes the operation  capability of the container further increased, which makes it more flexible for operation team to manage applications 

## 4.Conclusion

After a lot of tests in Alibaba's internal business systems, PouchContainer has helped the oversized Internet companies to containerize all online businesses. There is no doubt that rich container technology is the most practical technology that is not invasive for application development and operation. The open source PouchContainer hopes that the technology can benefit the industry, helping a large number of enterprises to win their own time in the containerization of the old stacked business, quickly embrace the cloud native technology, and make great strides toward digital transformation.