---
title: "Home Lab Part 1"
date: 2020-01-29T16:53:07-05:00
draft: false
categories: ["openstack"]
---

This is how I built my home lab server.

<!--more-->

Working as a full time upstream developer in OpenStack means that sometimes I
need to run a full cloud environment in order to determine if a new feature
works, or debug some kind of issue that someone has reported.  This requires
more capability than a standard laptop can provide: I need a machine that is a
very basic enterprise-class server.  And it needs to be able to have plenty of
memory.  And I specifically work on the networking aspects of OpenStack
(neutron) so I will need an environment that enables cloud networking
scenarios.

This is the first in a series of articles where I detail the process I followed
to procure a server to work on OpenStack upstream development.  I wanted to get
all-new components, fully supported by manufacturer warrantees.  I wanted to
get what was necessary to be able to test a wide variety of situations.  And I
wanted to pay as little as possible for it.  And it would need to have at least
128GB of RAM so that I could load up a configuration with a cloud that had
multiple controllers and computes on virtual machines, running virtual machines
of their own.

All prices are what I paid in fall of 2019.

## Part 1: Compute

If I need a server, then the best thing to do would be to go to the people that
make a large percentage of the servers out there in datacenter environments.  I
looked at offerings from HPE, Dell, Lenovo, and several other manufacturers.
This would have the advantage of providing a machine that would already have
a well tested and integrated configuration.  While putting together a server
entirely from component parts in the way a "gamer PC" is done these days is
fun, I would not end up with something that looked like it would be at home in
a datacenter.  And I would not have a company to file a warranty case with when
the system board has issues.

I settled on the very base model of the HPE ProLiant ML110 Gen10 server.  I
have a long experience with the ProLiant line, going all the way back to AOL's
webcache.  

- Single processor 8-core Intel Xeon 3104 
- 6 RAM slots; require DDR4-2666 RDIMM (DDR4-2933 RDIMM also compatible)
- 8 GB memory preinstalled (the minimum)
- Has capability for 192 GB of memory if 32 GB sticks are used
- ILO BMC Card 
- [Link to HPE](https://buy.hpe.com/b2c/us/en/servers/proliant-ml-servers/proliant-ml100-servers/proliant-ml110-server/hpe-proliant-ml110-gen10-server/p/1010192782)

The CPU is not blazing fast (it's a 1-CPU system) but I should be able to run
OpenStack on it - OpenStack is usually a memory-constrained beast.  My
deployments and tests may just require a bit more patience.  And if it becomes
an issue, the system board has another slot so I can get another Xeon to add
on.

The ILO BMC card is an important addition because system management cards are
used in datacenter environments to bootstrap bare metal hosts for bare metal
provisioning, which is something OpenStack can do through the Ironic project.
I don't do much with Ironic but I would like the capability to work with these
scenarios in the future.  Plus you can get on the console of your server [from
an iPhone app](https://apps.apple.com/us/app/hpe-ilo-mobile/id497560256).

Also evaluated:

- Dell R230 ($669) - unable to expand past 64GB RAM https://www.dell.com/en-us/work/shop/povw/poweredge-r230
- Dell PowerEdge T340 ($789) - unable to expand past 64GB RAM https://www.dell.com/en-us/work/shop/povw/poweredge-t340
- Lenovo ThinkSystem ST50 ($697) - unable to expand past 64GB RAM https://lenovopress.com/datasheet/ds0067-lenovo-thinksystem-st50
- HPE ProLiant MicroServer Gen10 - unable to expand past 16GB

Price for the ProLiant ML110:  $781.99

## Part 2: Memory

As I mentioned, OpenStack is a memory-constrained set of applications.  I would
like to get 32-GB RAM sticks, so that only 3 slots will be in use (8, 32, 32)
and I'll have 3 more for possible future upgrades.  By "possible future
upgrades" I just want to be clear I mean "I will certaintly want to do this but
in order to get the rock bottom cheap price I am deferring some of the RAM for
later" - the ideal quantity would be 128GB.  The place with the highest quality
reputation to go for RAM is Crucial, which has $209.99 per 32GB DDR4-2666
RDIMM.

- [Link to Crucial](https://www.crucial.com/usa/en/compatible-upgrade-for/HP---Compaq/proliant-ml110-gen10)

Price for the RAM: $209.99 x 4 = $840

## Part 3: NIC card

To quote wholesale from the OpenStack documentation:

  PCI-SIG Single Root I/O Virtualization and Sharing (SR-IOV) functionality is
  available in OpenStack since the Juno release. The SR-IOV specification
  defines a standardized mechanism to virtualize PCIe devices. This mechanism
  can virtualize a single PCIe Ethernet controller to appear as multiple PCIe
  devices. Each device can be directly assigned to an instance, bypassing the
  hypervisor and virtual switch layer. As a result, users are able to achieve
  low latency and near-line wire speed.

I will need an SR-IOV capable card to handle network virtualization
programming.  I asked my local SR-IOV expert,
[Rodolfo](https://rodolfo-alonso.com/), what he would recommend.  He
recommended an Intel X540 dual interface network card as a good and inexpensive
SR-IOV-capable option.

- [Link to the Intel X540 on Amazon](https://www.amazon.com/10Gtek-X540-T2-Converged-Network-Adapter/dp/B01HMGWOU8/ref=sr_1_1_sspa?keywords=intel+x540&qid=1570107242&s=gateway&sr=8-1-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEySjY1WUNXQ1Y4UFJEJmVuY3J5cHRlZElkPUExMDE4NTIzMTUwWFZMM1NaOVBGQyZlbmNyeXB0ZWRBZElkPUEwNDM2OTQzMlY2MjM2Sk1QWjQxTyZ3aWRnZXROYW1lPXNwX2F0ZiZhY3Rpb249Y2xpY2tSZWRpcmVjdCZkb05vdExvZ0NsaWNrPXRydWU=)

Price for the X540: $199.99

## Total Price

I threw in a small hard drive for $100 to round out the picture and the price
is less than two thousand dollars:

- Server: $782
- Memory: $840
- SR-IOV card: $200
- Hard drive: $100
- Total: $1,922

{{< figure src="../../assets/images/proliant-front.jpg" title="HPE ProLiant ML110 Gen10 - Front" >}}
{{< figure src="../../assets/images/proliant-back.jpg" title="HPE ProLiant ML110 Gen10 - Back" >}}
{{< figure src="../../assets/images/proliant-frontback-diagram.jpg" title="HPE ProLiant ML110 Gen10 - Front/Back Diagram" >}}
{{< figure src="../../assets/images/proliant-board-diagram.jpg" title="HPE ProLiant ML110 Gen10 - System Board Diagram" >}}

In the next chapter I will discuss how I configured the server and deployed
OpenStack on it.
