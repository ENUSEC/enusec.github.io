---
layout: post
title:  "Quickly Reversing The SolarWinds Backdoor"
description: A Quick look into how you might approach reverse engineering the SolarWinds Backdoor.
date:   2020-12-28 20:27:00 +0000
categories: SolarWinds RE Threats APT
---
This is a brief writeup on how to get your hands dirty with reversing the SolarWinds backdoor, some of the tooling required, and where to find it within the backdoored DLL. 

The SolarWinds compromise was a sophisticated supply-chain attack, in which it is highly likely the attackers had access to the build-process of SolarWinds Orion - the company's flagship product.

The sample we'll be looking at has a SHA-256 hash of `019085a76ba7126fff22770d71bd901c325fc68ac55aa743327984e89f4b0134`.

1. Choose your analysis environment, we recommend using a VM, such as FireEye's Threat Intelligence "ThreatPursuit-VM" which can be found [here](https://github.com/fireeye/ThreatPursuit-VM).
2. We have provided the malware for analysis here, download the backdoored SolarWinds DLL from here:  [solarwinds_implant.zip](/assets/res/posts/solarwinds/solarwinds_implant.zip). The password is `infected`. You may need to disable Windows Defender, or any other anti-malware solution, if enabled in your analysis environment.
3. Next, as we're dealing with a .NET binary, we'll use dnSpy (or any other .NET reflector) to decompile the bytecode back to source. Grab the latest release [here](https://github.com/dnSpy/dnSpy/releases/download/v6.1.8/dnSpy-net-win64.zip), and extract to a location of your choice.
4. Start dnSpy, and open the malware - File -> Open. Next, you'll see it opened in the left pane. We can observe several classes within the SolarWinds code, this is expected as the attackers embedded the backdoor code within legitimate SolarWinds-written code - they are likely to of have had presence  in SolarWind's entire build process
![nvekw.png](/assets/res/posts/solarwinds/nvekw.png)
5. Now, we'll locate the backdoor code. Go to the `SolarWinds.Orion.Core.BusinessLayer` namespace from the listing.
6. Next, you'll see a bunch of classes. Select the `OrionImprovementBusinessLayer` class from the listing.
7. Sick, we've successfully identified the malicous code. 
![l8rbghqemf.png](/assets/res/posts/solarwinds/l8rbghqemf.png)
8. Have fun reversing, we highly recommended taking a look at [begin.re](begin.re), and Malware Unicorn's [RE101](https://malwareunicorn.org/workshops/re101.html#0) if you want to learn more. 

# Detection
FireEye released a whole host of YARA rules (and more), to detect this specific threat. This can be found [here](https://github.com/fireeye/sunburst_countermeasures/blob/main/all-yara.yar#L6-L23).

## Boring Stuff
The information contained in this website is for general information purposes only. The information is provided by ENUSEC and while we endeavour to keep the information up to date and correct, we make no representations or warranties of any kind, express or implied, about the completeness, accuracy, reliability, suitability or availability with respect to the website or the information, products, services, or related graphics contained on the website for any purpose. Any reliance you place on such information is therefore strictly at your own risk.

In no event will we be liable for any loss or damage including without limitation, indirect or consequential loss or damage, or any loss or damage whatsoever arising from loss of data or profits arising out of, or in connection with, the use of this website.

Through this website you are able to link to other websites which are not under the control of ENUSEC. We have no control over the nature, content and availability of those sites. The inclusion of any links does not necessarily imply a recommendation or endorse the views expressed within them.

Every effort is made to keep the website up and running smoothly. However, ENUSEC, takes no responsibility for, and will not be liable for, the website being temporarily unavailable due to technical issues beyond our control.

`Author: @LloydLabs`
