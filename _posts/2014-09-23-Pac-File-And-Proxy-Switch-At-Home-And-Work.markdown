---
layout: post
title: PAC File and Proxy Auto Switch for Firefox IE11 and Chrome
category: idev
tags: PAC proxy auto switch IE11 local
year: 2014
month: 09
day: 23
published: true
customid: 20140923_proxy_pac
summary: Something about PAC files and related configurations on different browsers, have solutions for local pac file fails on IE11.
---
I'm using my working laptop after work at home currently before my PC DIY finished and new RMBP reveled. At work we need to access internet via internal proxy servers and after work just connect laptop directly to the wireless router to access web. It's really boring to turn proxy off at home and enable it when I go to work. So I try to use PAC(Proxy Auto Config) file to choose proxy server or connect directly to the web according to my LAN IP address.

From [wikipedia](http://en.wikipedia.org/wiki/Proxy_auto-config) when can know that what PAC file is:

>A PAC file contains a JavaScript function `FindProxyForURL(url, host)`. This function returns a string with one or more access method specifications. These specifications cause the user agent to use a particular proxy server or to connect directly.

So I just need to customize my `FindProxyForURL` to check my local IP address. If my laptop's local IP address is normal home used wireless router host IP such as `192.168.1.1` or `192.168.0.1`, simplly just return `DIRECT` for direct network access. Otherwise, return my company's proxy server. We can use the following function `myIPAddress` to get local IP and `isInNet` function to determine the subnet.

>The `myIpAddress` function has often been reported to give incorrect or unusable results, e.g. 127.0.0.1, the IP address of the localhost. It may help to remove on the system's host file (e.g. /etc/hosts on Linux) any lines referring to the machine host-name, while the line 127.0.0.1 localhost can, and should, stay.
The myIpAddress function assumes that the device's has a single IPv4 address. The results are undefined if the device has more than one IPv4 address or has IPv6 addresses.

>The `isInNet` function evaluates the IP address of a hostname, and if within a specified subnet returns true. If a hostname is passed the function will resolve the hostname to an IP address.

Ok, now I got what I need to. Here is the overall struct of my PAC file
```
function FindProxyForURL(url, host)
{
    // get local IP address, client IP address
    var ip = myIpAddress();

    // Normal router intranet ip, home usage, return direct connect, add
    // any other local IP address here
    if (shExpMatch(ip, "192.168.*") || shExpMatch(ip, "172.16.*"))
    {
        return "DIRECT";
    }

    // else its company intranet, internal and localhost, direct connect
    if(shExpMatch(url, "*.internal") 
            || shExpMatch(host, "127.0.*")
            || isPlainHostName(host))
    {
        return "DIRECT";    
    } 

    // company rules, company network, direct connect
    if(dnsDomainIs(host, "intranet")
            || dnsDomainIs(host, ".company.com")
            || dnsDomainIs(host, ".company.net"))
    {
        return "DIRECT";   
    }

    // others are internet, we need proxy to access them, multiple proxy
    // server are splitted by semicolon, usally One is enough
    else 
    {
        return "PROXY proxy1.company.com:8080; " + 
            "PROXY proxy2.company.com:8080; " + 
            "PROXY proxy3.company.com:8080";
    }
}
```

Now, just add this file to browser network settings. It works well on firefox,chrome(via [Proxy SwitchySharp](https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm?hl=en)) and IE9 on my colleague’s laptop. But It fails to get any thing on my laptop with IE11 installed. By searching the internet I found that IE11 disabled local PAC file for security reasons(WTF security reasons??) .

>In Internet Explorer 11, the WinINET team has disabled WinINET’s support for file:// based scripts to promote interoperability across network stacks. Corporations are advised to instead host their proxy configuration scripts on a HTTP or HTTPS server. 

And Microsoft also provided a temporary workaround by enable it via registry key `HKLM\SOFTWARE\Policies\Microsoft\Windows\CurrentVersion\Internet Settings\` with `EnableLegacyAutoProxyFeatures` set to 1.

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\CurrentVersion\Internet Settings]
"EnableLegacyAutoProxyFeatures"=dword:00000001
```

After Importing the registry, It still fails to get proxy server or directly connection(WTF workaround). Again by search via our great Google, There was another [workaround](https://connect.microsoft.com/IE/feedback/details/793556/local-proxy-which-is-set-in-the-proxy-auto-config-of-ie11-is-not-processed-correctly-by-ie11) provied by M$

>Please try this:
The location for local pac file for IE and the windows platform is:
file://c:/windows/system32/drivers/etc/proxy    
next to your hosts file and note there is no extension on the proxy file.

Fortunately, now the configuration works fine on IE11. But still some Apps cannot get access to the Internet when using `Use system/IE proxy settings` options, Tecent QQ for example.

At last, WTF security issues and workaround!