## Weekend Proxmox Server Project



### Intro

I've recently been looking for a project to complete on the weekend. Virtualising my router always sounded like a bit of fun, so I decided to give it a go!

This is a project that a friend of mine had also completed. This, combined with influx of videos found on the Server the Home Youtube Channel (LINK) as well as the FORBIDDEN ROUTER video series found on level1techs drew my attention enough that I wanted to have a go myself!

Knowing myself, I thought I should document how I built out the initial setup, and since this is a personal project I thought I should try half-assing a blog for documentation.

### My Approach to the Solution

I had a few key criteria I was looking for when picking software and hardware. These were:

1. Pick distros/ software I am already familiar with when possible.
2. The Router has to be small, quiet, cheap and fast  enough. 
3. The solution has to be simple enough that other people can understand it.
4. Have the solution be self contained, but easily recoverable if a software component dies.

With this criteria in mind, this was what I ended up deciding upon for the project.

### Hardware

An Intel Celeron N5105 4 Core Firewall unit from Topton computer on Aliexpress. The Router came bare-bones, but current specs are:

| Category   | Spec                                                         |      |
| ---------- | ------------------------------------------------------------ | ---- |
| CPU        | Intel Celeron N5105                                          |      |
| RAM        | 16GB of RAM (Rated at 3200MHz, running at 2667MHz)           |      |
| Storage    | 240GB SSD (M.2 GEN 3 SSD)                                    |      |
| Networking | 4x Intel i225 2.5Gbit NIC                                    |      |
| Misc.      | 1x Serial Port, 2x USB 2.0, 2x USB 3.0 Displayport,HDMI, MicroSD slot, SIM tray. |      |



![PXL_20221118_090039950](https://github.com/carbonos1/carbonos1.github.io/blob/main/_posts/2022-11-22-graphics/PXL_20221118_090039950.jpg)



![PXL_20221119_050615043](https://github.com/carbonos1/carbonos1.github.io/blob/main/_posts/2022-11-22-graphics/PXL_20221119_050615043.jpg)

### Software:

When deciding on the software for my router, I decided on the following for each Category

| Category          | Solution           | Reason                                                       |
| ----------------- | ------------------ | ------------------------------------------------------------ |
| Type 2 Hypervisor | Proxmox            | I have used Proxmox in the past and have experimented with PCI passthrough. I am also very familiar with Debian, which allows me to use the Debian based proxmox easily. |
| Router Distro     | PFsense            | I am familiar with freeBSD through university, as well as uning TrueNAS core on my NAS. I was happy to use a freebsd based OS for my router as a result. I chose freebsd for long term stability and support from Netgate |
| Container Host OS | Debian             | I am familiar with Debian. it is also very lightweight in container instances, using less than 50MB of RAM. |
| DNS               | Pi-Hole, Lan-cache | Pi-Hole is used as a caching server, as well as to block ads. Lan-cache is used to cache files to be downloaded often, like windows updates and steam games. I was contemplating running unbound for a dot level DNS server, but thought this would be a good start anyways. |
| Upstream DNS      | 1.1.1.1            |                                                              |



The Logical Diagram of the solution looked like as follows:

![Quinton Logical Diagram.drawio](https://github.com/carbonos1/carbonos1.github.io/blob/main/_posts/2022-11-22-graphics/Quinton%20Logical%20Diagram.drawio.png)

This diagram is an accurate representation of the router as of 20/11/22.  Additional work will need to be done to allow it to interact with the TPG internet service I am currently using.

Each Device / VM listed here will auto-boot when proxmox comes online.




```tsql
 SELECT *
 FROM sys.tables
 WHERE [name] = 'SomeTable'
```



### DNS

The firewall uses two different DNS applications to serve DNS, Pi-hole and LANCache.

Pihole is a highly capable ad blocking DNS solution, able to block ads across the entire network. Pi-Hole has evolved to be capable of far more, but the only functions we're using  in this example is local entries and its ad blocking capabilities. Pi-Hole was installed directly to an LXC container.

LAN-Cache automatically stores files in instances where they could be frequently downloaded, eg Steam, Windows update, etc. This can reduce unneeded load from the network. This was installed on an LXC container as a docker container, and configured to only use 90GB of space, down from the 2TB default. LAN-Cache fetches its DNS upstream from Pi-Hole, and thus custom entries on pi-hole will appear on Lan-Cache (as well as ad blocking!)



### Testing

As you can imagine, for Australian internet standards, this router is extremely overkill. However, from a LAN perspective it is interesting to see how it performs. I was primarily interested in seeing how the virtual bridges performed. Since I don't have any 2.5gb capable devices as well, I was only able to test up to 1gb/s worth of throughput of the physical NIC.

#### iPerf Tests:

**<u>Proxmox Host to PFsense host connection:</u>**

![312708613_600343885177782_7901942438424020443_n](https://github.com/carbonos1/carbonos1.github.io/blob/main/_posts/2022-11-22-graphics/Proxmox-pfsense.png)



**<u>Pihole Container to Proxmox Host</u>**

![313898247_629872635557373_2093165718693338476_n](https://github.com/carbonos1/carbonos1.github.io/blob/main/_posts/2022-11-22-graphics/pihole-to-proxmox.png)

**<u>Laptop (Thinkpad T480 ) to PFsense via Proxmox</u>**

![laptop-to-pfsense-via-proxmox](https://github.com/carbonos1/carbonos1.github.io/blob/main/_posts/2022-11-22-graphics/laptop-to-pfsense-via-proxmox.png)

**<u>Laptop (Thinkpad T480) to PFsense via ETH1</u>**

![laptop-to-pfsense-direct](https://github.com/carbonos1/carbonos1.github.io/blob/main/_posts/2022-11-22-graphics/laptop-to-pfsense-direct.png)



As can be seen, the router is more than capable of generating large amounts of traffic, as well as having the routing / switching capacity to switch data at speeds exceeding those of the physical ports. I find it interesting that the PF-Sense to proxmox connection is only 3Gbit/s, but this may be bridge overhead from the Virtual Machine, or the fact the bridge is attached to a 2.5Gbit interface. Ping on all interfaces is less than 1ms, and ping to to upstream router when configuring was the same, indicating little to no latency is introduced by the box itself.

Overall, this fun weekender project created a highly capable router more than happy to route at 2.5Gbit speeds, which will replace my edgerouter quite nicely.

