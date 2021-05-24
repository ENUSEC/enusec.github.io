---
layout: post
title:  "Soda PDF Reader SSL Pinning Bypass"
description: Bypassing the SSL Pinning implementation used in the software Soda PDF.
date:   2021-05-23 18:00:00 +0000
categories: Vulnerability Crypto Research Disclosure Bypass
---
In this post I will be talking about a trivial SSL Pinning Bypass I found against LuLu Software Soda PDF Reader and outline the steps I took and how you may follow the same
methodology to bypass SSL Pinning against other Desktop and Thick Client applications.

## SSL Pinning
SSL Pinning/SSL Certificate Pinning is a method for preventing Man in the Middle attacks against an applications secure network communications. By taking record of a hosts public key the application can
verify that it is talking securely to the host, since it verifies the public certificate provided by the host with the one the application already knows about.
This pins the certificate and if an attempt is made to intercept this communication by spoofing the host, the application will detect this due to there being a mismatch between the
certificate the application knows about and the certificate being provided by the spoofed host.

SLL Pinning is used in a large amount of applications, desktop applications, thick client applications and mobile applications are the most common however it is not limited to
just these instances.

You can find out more information about SSL Pinning [here](https://www.thesslstore.com/blog/an-introduction-to-pinning/).

## Why Bypass SLL Pinning?
Often whilst pen testing a mobile application is is important to intercept the communication between the application and server it is using. However, because of the nature of these
communications they are often encrypted and pinned to prevent an attacker easily performing MitM attacks. As a pen tester this can make it difficult and you would have to find a
way arround these protections. Once bypassed the pen tester has a much more transparent view of the application and can perform a more in depth test.

The techniques used can be mirrored into a different context and in this case I wanted to look at the Soda PDF reader and any issues that might be present in the application.
By bypassing the SSL pinning mechanism I would be able to have a more transparent view of the application and also test input to the application that comes from a server response
and therefore may lack the same sanitization as user supplied input. It may also be possible to extend the applications free trial by spoofing the
server responses and tricking the application into thinking that the `7 Day Free Trial` is still going ahead even though the time has expired.

The possibilities are endless.

## Firing up the application
The first stage was to simply fire up the application and gain an idea of the program functionality. Running Soda PDF I was greeted with a user registration form allowing you to either log into the application or register an account. This immediately highlights that the application may be speaking to a backend for authentication purposes.

![Sodapdf Authentication](/assets/res/posts/sodaPDF/sodapdf_auth.PNG)

Once registered and logged in I could access the application.

![Sodapdf after Authentication](/assets/res/posts/sodaPDF/sodapdf_authenticated.PNG)

At this point it would be possible to explore the different functionality of the application and test for particular vulnerabiltites when the user can supply input through a number of different methods. Initialy, however, having seen that the application may be speaking to a backend I wanted to see if it was possible to view, intercept and modify the communication between the desktop application and the backend.

## Proc Mon
The next stage in my proccess was to explore what the application was doing behind the scenes. This involves seeing what proccesses, executables and services that are running, the permisions they have and if anything interesting sticks out. In order to do this I would first have to create some filters so that the only information I got in Procmon was that of the application itself and associated binaries.

Using task manager I was able to discover two running executables and three services. The executables and running privileges can be seen bellow.

![Table of executables](/assets/res/posts/sodaPDF/executables.PNG)

Despite being very limited this information provides some interesting insight, specifically that the activation service (presumably monitoring the time left on the free trial) is running as system. This is interesting because if there happens to be a weakness in the executable there may be scope for elavation of privileges (EOP).

However, now I knew the executables that were running I could add these as filters to Procmon and therefore only see actions that were associated with these two executables.

![Procmon Filters](/assets/res/posts/sodaPDF/proc_mon_filters.PNG)

Upon restarting the application and looking at the events in Procmon, as you can imagine, thousands of events were created.

![Procmon Events](/assets/res/posts/sodaPDF/events_snippet.PNG)

Since I was wanting to veiw the network traffic I first stripped the Procmon results to only show network traffic.

![Procmon Network Events](/assets/res/posts/sodaPDF/network_only.PNG)
 
This revealed that the application was definitely talking to a backend and the network traffic was encrypted (HTTPS). Closer inspection shows that the requests from both the activation service and the soda executable were being made to the IP address ```104.16.180.79```.

This IP address is actually a Cloudflare IP address therefore indicating that whatever domain the application is connecting too sits behind cloudflare. This also makes it difficult to run a reverse DNS against the IP to determine the domain/domains the application is connecting too.

## Wireshark
A solution to find the domains the application was connecting too was actually relatively simple. At some point the application must be doing a DNS lookup if it was making a HTTPS request. Luckily it appeared that this application was also not using the system cache therefore was making these DNS requests everytime the application would run.

To find out the domains I simply used the following filter in Wiresark which would make obvious what TCP connections were being made to the cloudflare IP and the preliminary DNS request made.
```
(ip.dst == 104.16.180.79 && tcp) || dns
```

Some of the output can be seen bellow.

![Wireshark Capture](/assets/res/posts/sodaPDF/network_capture.PNG)

Here it is obvious that the desktop app is communicating using TLS however from the wireshark capture we are now able to identify three domains that the application is try to connect to.

* oauth.sodapdf.com
* api-updateservice.sodapdf.com
* stats.sodapdf.com

It is important we know the domains the app is connecting to so that we can perform invisible proxying of the application.

## Invisible Proxying
At this point we can try and intercept the traffic.

For the the sake of this blog post I was pressuming the application, specifically the activation service that the research was focusing on is a non-proxy-aware app therefore in order to proxy the application I am going to be using Burp Suites invisible proxy.

Invisible proxying is a simple concept made extremely easy when using Burp Suite. By adding the domains that get resolved by the application to the systems hosts file we can set them to resolve to a proxy running on the local system. This proxy can then forward/mirror any requests to the intended target and intercept the response. Therefore allowing a non-proxy-aware client to have its network traffic proxied. The below diagram explains this proccess in 4 easy steps.

![Invisible Proxy](/assets/res/posts/sodaPDF/invisible_proxying.png)

Setting up the proxy settings in Burp is very straight forward.
1. Navigate to Proxy -> Options -> Proxy Listeners -> Add
2. Here we want to bind to port 443 since we are intercepting encrypted traffic.
3. Be sure to listen on the loopback only
4. Navigate to Request handling
5. Add the IP address to forward the request onto, in this case 104.16.180.79
6. Select the option `Support invisble proxying`
6. Click OK 

![Burp Suite Invisible Proxy Settings](/assets/res/posts/sodaPDF/burp_invisble_proxy.PNG)

It really is that simple to set up an invisble proxy for non-proxy-aware applications. The same technique can be used for proxying non-proxy-aware mobile applications and also Thick clients. Further information about Burp Suites invisible proxy can be found [Here](https://portswigger.net/support/using-burp-suites-invisible-proxy-settings-to-test-a-non-proxy-aware-thick-client-application).

Therefore I added the subdomains collected earlier to my hosts file and configured Burp correctly.

![Hosts file](/assets/res/posts/sodaPDF/hosts_file.PNG)

## Initial Testing
Now the proxy is set up it was time to test if it was simply a case of intercepting the encrypted communications as the application was not checking the certifcate provided. If it didn't have a strict policy then it may be possible to intercept the encrypted communications without needing any extra steps.

Now running the application if the interception is succesful we would either see the requests in Burps history tab or we would recieve an error.

![Burp Certificate Errors](/assets/res/posts/sodaPDF/burp_error.PNG)

Unfortunately, in this test we recieved certficate errors in Burp revealing that the client was in fact pinning its certifcate. This was followed by an error in the application itself showing that it was not able to connect to its backend.

![Sodapdf Error](/assets/res/posts/sodaPDF/sodapdf_error.PNG)

At this point some applications may be using the local machines certifcate store. Therefore opening `MMC.msc` I added the Burp certificate to the certifcate store and ran the application again. Unfortunately this time it also gave a similar error indicating that the application was responsible for pinning all of its own certificates and was not trusting those on the local system.

## Bypassing the SSL Pinning
At this point it was time to revist Procmon and see if there was anything interesting that may be of assistance and offer a method for bypassing the certificate pinning.

Fortunately I had previously seen a file that I thought might be extremely useful.
![Certificate Bundle File](/assets/res/posts/sodaPDF/crt_file.PNG)

Both the soda.exe and activation-services.exe were making multiple reads to the file `C:\Program Files\Soda PDF Desktop 12\curl-ca-bundle.crt`.

This file was a bundle of certificate authorities root certifcates. I thought that potentially the application was using this to verify the authenticity of certifcates and encryption being used in its network communications. Not only did the contents of the file suggest this but the name of the file beggining with `curl-*` indicated that the application may have been using curl to make the requests and using this to verify the certifcate chain created when making a request to \*.sodapdf.com.

A snippet of the bundle file can be seen below.

```
##
## Bundle of CA Root Certificates
##
## Certificate data from Mozilla as of: Wed Jan 23 04:12:09 2019 GMT
##
[... SNIP ...]
## Conversion done with mk-ca-bundle.pl version 1.27.
## SHA256: 18372117493b5b7ec006c31d966143fc95a9464a2b5f8d5188e23c5557b2292d
##


GlobalSign Root CA
==================
-----BEGIN CERTIFICATE-----
MIIDdTCCAl2gAwIBAgILBAAAAAABFUtaw5QwDQYJKoZIhvcNAQEFBQAwVzELMAkGA1UEBhMCQkUx
GTAXBgNVBAoTEEdsb2JhbFNpZ24gbnYtc2ExEDAOBgNVBAsTB1Jvb3QgQ0ExGzAZBgNVBAMTEkds
[... SNIP ...]
j/8N7yy5Y0b2qvzfvGn9LhJIZJrglfCm7ymPAbEVtQwdpf5pLGkkeB6zpxxxYu7KyJesF12KwvhH
hm4qxFYxldBniYUr+WymXUadDKqC5JlR3XC321Y9YeRq4VzW9v493kHMB65jUr9TU/Qr6cf9tveC
X4XSQRjbgbMEHMUfpIBvFSDJ3gyICh3WZlXi/EjJKSZp4A==
-----END CERTIFICATE-----
...
```

Therefore in order to bypass the SSL Pinning implementation all I needed to do was add the Burp CA certificate into this file and I would be able to trick the application into trusting the certificate chain.

Converting the .der file produced by visiting `http://burp` in your browser (configured pointing at the Burp proxy) I was able to simply copy and paste the certificate onto the end of the file `curl-ca-bundle.crt`.

```
Burp Root CA
==================
-----BEGIN CERTIFICATE-----
MIIDqDCCApCgAwIBAgIFALXNBogwDQYJKoZIhvcNAQELBQAwgYoxFDASBgNVBAYT
C1BvcnRTd2lnZ2VyMRQwEgYDVQQIEwtQb3J0U3dpZ2dlcjEUMBIGA1UEBxMLUG9y
dFN3aWdnZXIxFDASBgNVBAoTC1BvcnRTd2lnZ2VyMRcwFQYDVQQLEw5Qb3J0U3dp
Z2dlciBDQTEXMBUGA1UEAxMOUG9ydFN3aWdnZXIgQ0EwHhcNMTQwMjAyMDA1NTEy
WhcNMzEwMjAyMDA1NTEyWjCBijEUMBIGA1UEBhMLUG9ydFN3aWdnZXIxFDASBgNV
BAgTC1BvcnRTd2lnZ2VyMRQwEgYDVQQHEwtQb3J0U3dpZ2dlcjEUMBIGA1UEChML
UG9ydFN3aWdnZXIxFzAVBgNVBAsTDlBvcnRTd2lnZ2VyIENBMRcwFQYDVQQDEw5Q
b3J0U3dpZ2dlciBDQTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKdA
BQoPZSLOaIHIW5yKEel1ZhCLHPyHcVum+iCneTOtn8OKlfxBRxlD+bnKrgkRpPIF
hvoe61NoK3ASTRylonsu4rPX4spP7D4UQpjVe60EO6IjDJHLESYwepDA0ShX7SK1
iit+M7FA22UkxHIr99QK0FBuBGvR2cs7KzqHdnfJXezSFa32sWbkCGYGIb0XQQgB
lQq8yBw0AwsEH65QTPPhsC9kM4AiJwECexZPNl0iawAE8ZIpzaoPdfIziyShLqk/
N9M193rWqsJacAD9enqU2dRJSvGJDnLm64Q8rO0lZ7Rxy+JAeYdMEWFtXNdKVK/x
WWgd2NEEaYnjvRUCNmcCAwEAAaMTMBEwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG
9w0BAQsFAAOCAQEARXGSZJeMawW3aV5OUeszxUy0dcXBHNg18kxktvPABAiVJxFx
VCYoncUtju/QSyXhLTOy88l2DNSeQ7QMZ9bfZsD6OubTVMhf/QydTH+9HgR4gHWt
5Bms9oacJVy3P4NPRJTIgXhzlxv2gMp+q6ohAEfVbN0RZ1xf6Ye4SPE30N4XGlgZ
XuKdD4MVJKQFhrFjwU3shiSBPp6BiAa7SNQzsQxu2K+VIhBiv6xKBcE4JnnRB5wI
+tax5hr6eWwWKom6tyHEhAkGGeU4Y64aQPWPhC9emOZXTVA7o8ZP1hAyy++G84UO
qPuEjYujihVZGE5y139YVPWI32DnULmBH0iaFA==
-----END CERTIFICATE-----
```

## Testing the Bypass
Finally, now the Burp CA certificate is added to the certificate bundle that the application is using to verify the certificate chain I should be able to intercept its communications.

Running the application and checking the Burp history I was able to view the commuication between the desktop application and its backend.

![Success](/assets/res/posts/sodaPDF/burp_proxy_success.PNG)

I could now intercept, modify and manipulate the requests made by the application proving I had successfully bypassed the SSL Pinning implementation.

![More Success](/assets/res/posts/sodaPDF/burp_success_request_response.PNG)

## Responsible Disclosure
I had no issues disclosing this issue and the proccess was quick and responsive, however, after multiple times asking for permision to post this blog post and being told that someone would get back to me I have heard nothing and since the 90 day disclosure period is passed I see no reason not to share this educational content.

### Timeline
- First Contact - 03/02/2021
- Responsible Disclosure of Vulnerability details - 07/02/2021
- Vulnerability Fixed - 16/02/2021
- Public Disclosure - 23/05/2021

## Questions
If you have any questions feel free to get in contact with me!

Thanks for reading.

`Author: @su__charlie `
