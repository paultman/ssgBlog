---
title: "Macbook Pro Upgrade SSD, hello 1TB"
date: "2021-08-18"
categories: 
  - "fun"
  - "tech"
tags: 
  - "apple"
  - "macbook-pro"
  - "ssd"
  - "upgrade"
image: "f66c2-we6movjsxwdkksls.medium.jpeg"
---

## Abstract

Many apple Macs were easily upgradable until 2012, and somewhat upgradeable till 2015. This guide gives a background on upgrading macs, and focuses on the procedure to upgrade the 2015 15" macbook pro. Other models follow a similar procedure.

## Background

Gone are the days when macs were user fixable, and more importantly, upgradable. Sure, this was a time when a 15" macbook pro weighed 5.6lbs and was an inch thick, however not all of us would trade the ability to tinker with our macs, in exchange for a lighter/thinner machine.

Which modern macs could be upgraded?

#### Macbook Pro

- 2006-2008 pre-unibody (storage, Ram, battery)
- 2008-2012 Unibody (storage, Ram, battery)
- 2012-2015 retina (storage w/adapter)
- 2016-2017 13" non-touch bar (storage w/ adapter)
- 2016-2021 touch bar (none)
- 2020-now Apple silicone (none)

#### Mac Mini

- 2006-2010 Polycarbonate (storage, ram)
- 2010-2012 Aluminum with dvd (storage, ram)
- 2014 Aluminum (storage w/adapter)
- 2018 Aluminum (ram)
- 2020-now Apple Silicon (none)

#### Macbook Air

2010-2012 Aluminum w/ upgraded screen (none)  
2013-2017 Aluminum w/ pci-e (storage w/ adapter)  
2018-2020 Retina (none)

The best strategy was to get the latest model just before a generation change and stay with it as long as possible. The stand out models are the 2012 mac mini, 2012 13" macbook pro, and the 2015 15" macbook pro. The former two, because the spinning disk SATA-6 HD can easily be changed to a much faster SSD drive, and additionally ram can easily be upgraded from4gb to 16GB. Of the three, the only model supported by the latest version of OSX, Big Sur, is the 2015 model which I'll be detailing the procedure for here.

Why focus on the 2015 15" Macbook pro? As mentioned, it's supported by Big Sur, and will also be supported by Monterey, the next version of OSX. Additionally, it's got more ports than even the upcoming [rumored](https://www.macrumors.com/guide/14-inch-macbook-pro/) macbook pros, quadcore i7 processors, nice display and keyboard.

Theres is a nearly 400 page [post](https://forums.macrumors.com/threads/upgrading-2013-2014-macbook-pro-ssd-to-m-2-nvme.2034976/page-370) on MacRumors discussing the state of 3rd party NvME support on Macbook Pros, but I'll discuss all you need here. Specific to the 2015 model, Apple recently released a firmware update to support 3rd party NvME drives, so features like Hibernate and low power mode are now natively supported. Another characteristic, specific to the 2015 15," is PCI-E 3.0x4 support, an upgrade over the previous PCIe 2.0x4 found on the 2014 and earlier 15" models, and the 13" 2015 Pro and Air siblings.

The best version of the 2015 15" Macbook pro for the SSD upgrade is the model with only the integrated Intel video card. For models that also include a discrete video card, the PCI-e bus will be shared between the video card and SSD. For that reason, apple includes only a PCI 2.0x4 SSD, rather than a full speed PCI-e3.0x4 SSD which the upgraded interface is capable for. The slower stock SSD means that there is no problems with both devices being on the same shared bus. So, while either version can be upgraded for more storage capacity, the integrated graphics version will have a larger SSD speed benefit during times of heavy graphics processing.

What's needed? You'll need an adapter to convert a standard NvME M2 drive to fit into the proprietary Apple SSD slot, and of course you'll need a 3rd Party SSD drive.

![](images/a5d0e-71z5rnfovll._ac_sl1500_-1024x558-1.jpg)

[Sintech NGFF M.2 nVME SSD Adapter card](https://www.amazon.com/gp/product/B01CWWAENG)

![](images/fca4a-71ntyueausl._ac_sl1500_-1024x374-1.jpg)

[Sabrent 1TB Rocket NVMe PCIe M.2 2280 Internal SSD](https://www.amazon.com/gp/product/B07LGF54XR)

Here is a chart of tested SSD's ranked by power efficiency:

![](images/d69d8-ssd-nvme-comparison-2020-02-power-efficiency-1024x521-1.png)

With those two products, you can follow the excellent guide at [iFixit](https://www.ifixit.com/Guide/MacBook+Pro+15-Inch+Retina+Display+Mid+2015+SSD+Replacement/48251) for the easy SSD replacement. Hang on to your old SSD as some have reported that when Apple does firmware upgrades (sometimes occurring when new versions of OSX are installed), that w/o the Apple SSD, the updates don't happen. To be honest, for such a mature product, it's not as important, but something to check after updating to newer versions of OSX. You can easily check by running a program called [SilentKnight](https://eclecticlight.co/lockrattler-systhist/) by the Electric Light Company.

![](images/8c418-808481-dd24a85f047cc073e04a4d71d84ab86e.jpg.png)

### Example Results

#### Before:

![](images/2473b-screen-shot-2021-07-30-at-2.55.45-pm-993x1024-1.png)

#### After:

![](images/75575-screen-shot-2021-08-02-at-9.41.36-pm-993x1024-1.png)

So, for around a quarter of the cost of the same upgrade from Apple, and double the speed, the research and modification is well worth it. Please comment below if you have any questions about this mod, or any others related to the mac mini, air, or pro. All of which I have experience upgrading :-)

References:

[https://beetstech.com/blog/apple-proprietary-ssd-ultimate-guide-to-specs-and-upgrades](https://beetstech.com/blog/apple-proprietary-ssd-ultimate-guide-to-specs-and-upgrades)

[https://everymac.com/systems/apple/macbook\_pro/specs/macbook-pro-core-i7-2.2-15-iris-only-mid-2015-retina-display-specs.html](https://everymac.com/systems/apple/macbook_pro/specs/macbook-pro-core-i7-2.2-15-iris-only-mid-2015-retina-display-specs.html)

[https://forums.macrumors.com/threads/upgrading-2013-2014-macbook-pro-ssd-to-m-2-nvme.2034976/](https://forums.macrumors.com/threads/upgrading-2013-2014-macbook-pro-ssd-to-m-2-nvme.2034976/)
