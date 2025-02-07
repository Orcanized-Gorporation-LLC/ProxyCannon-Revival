# ProxyCannon-Revival
This repo is a fork of a single script by Shellntel in their repo "scripts".  Separated here for ease of code management and to be able to collect updates etc from the community.

This project looks to have been abandoned by the original author and is no longer functional on OS's such as Ubuntu 20.04.  For this reason it has been recreated here so that it is useful again.

### Why bother

A customer requirement kicked off a search for tools that created a network of proxies for IPS evasion.  They genuinely wanted me to spend time engineering a way to avoid their "fail2ban" setup...

I looked at several existing projects and most seemed massively complex, for example, require you to have a working terraform install and so on.  I wanted a point and shoot solution: the original Proxy Cannon project seemed to be it.

But the original was buggy and didn't work...  I fell into the trap of "just doing a quick bit of bugfixing" which turned into a couple of days engineering time.  Yes, I probably could have got one of the other projects online, but no, I didn't realise that until I was far too invested in ProxyCannon-Revival.

## Command-line syntax

````
-id, default='ami-d05e75b8', Amazon AMI image ID
-t, default='t2.nano', Amazon AMI image type
--region, default='us-east-1', Select the region
-r, Enable Rotating AMI hosts
-b, Enable multi-path cache busting
-m, Disable link state monitor
-v, Enable verbose logging
--name, Set the name of the instance in the cluster
-i, default='detect', Interface to use, default will result in detecting the default gateway and using that
-l, Enable logging of WAN IP's traffic is routed through

num_of_instances, The number of instances you'd like to launch.
````

## Install

### Debian-based systems

```apt install python3-boto python3-netifaces sshuttle ```

### For those who prefer Pip

```pip3 install -r requirements.txt```


## Original announcements

The below is the text from the original blog posts.

------------ 

### September 26, 2015

This is an update to an older post that can be found here.  Since createProxy's initial release, we've received some great feedback and, as a result, we made some improvements.   There were several shortcomings with the previous version, all of which rested on the use of ProxyChains.  ProxyChains is old, outdated, and failed to support  protocols such as  UDP or ICMP or Java apps (i.e. Burp).  So it was re-written, code can be found: https://github.com/Shellntel/scripts/blob/master/proxyCannon.py

The original version used SSH's ability to create SOCKS proxies; however, as it turns out SSH also supports the ability to create both layer 2 and layer 3 tunnels.  These are different than SOCKS connection, in that a fully-fledged network tunnel is established between the two endpoints.  Find more details on how to set this up here.

The new proxyCannon script takes advantage of this feature by building VPN tunnels to each EC2 instance and round robining locally generated traffic between each one.  Since we're using the local systems routing table all session information will be retained (i.e. TCP streams are not split between systems).  Furthermore, now we can push any network based traffic across it: TCP, UDP, ICMP and more. 

Below are some pictures of proxyCannon in use:

An example of the script starting up 3 ami-d05e75b8 (ubuntu) instances on t2.micro hardware in the us-east-1 region.

Next, in a new tab we verify that our connections are up

Here we can see that 3 SSH connections are successfully established to each public IP of the ec2 instances started

And here we can see that 3 new interfaces were created, one for each SSH tunnel established

With everything stood up, we run a simple test.  In one tab we run a simple ping with a count of 3.  

We can see that each ICMP packet is sent through a different EC2 node.  Also note the time stamps on the packet each node is used randomly!

That it!  Browsers, pentesting tools, whatever, should all work seamlessly.  To close down the proxy, just go back to your first tab and hit enter.

Any questions, comment or bugs, please feel free to submit them to github.  Thanks!


### May 24, 2016

ProxyCannon, which can be found here, has undergone some revisions since our initial release and as a result, there's some new features we'd like to introduce.
Cleaner User Interface.  

We've cleaned up the number of arguments required to run the app from 6 to 3.  Now you only need to specify the AMI KEY, AMI ID, and the number instances you'd like start. You can still specify images size, type, etc, we just set the most cost effective options as default.
New Options

So if you're familiar with the older version of createProxy, you'll notice we've added new options.  

    The -v option provides verbose debug output, specifically used for troubleshooting issues.
    The -i option was added to let the user specify which interface should be used for establishing outbound connections.  Useful if your client is on a mac, or has multiple nic's.
    The -l option will log all assigned IP's to a file for record keeping 
    The -r switch rotate IP's

Rotate IP?!

So even by sending your Nessus scans across 10 Amazon EC2 hosts an IPS can still detect and shun one of your exit nodes. To deal with this, we've added the -r feature which will automatically rotate the public IP address of every exit node. Let say for example you fire up proxy cannon with 10 nodes and add the -r switch.  The first thing the script will do is build the egress nodes, tunnels, iptables and routes like normal.  Next the script will choose one node (at random) and alter the local route table to make that it appear less appealing than the other 9.  As a result, all new outbound TCP/UDP/ICMP packets will be traverse the remaining 9 nodes evenly all the while allowing any existing session data on the chosen node time to finish receiving its data. The script will then monitor the network state table for the chosen node. Once it identifies that the node is idle (i.e. no half open TCP/UDP/ICMP packets), proxyCannon tells Amazon to re-assign the nodes WAN IP. Next ProxyCannon will reprovision the host and add it back into the general pool of 10 nodes, and move onto the next.  On average, regardless of the number of exit nodes provisioned, proxyCannon will change its exit nodes IP's about 30 times in one hour without dropping / breaking a single session!
Rotate Nodes (-r) in action. &nbsp;You'll note a few AWS API timeouts. Proxy Cannon will now attempt to recover its state automatically without immediately giving up.Rotate Nodes (-r) in action. &nbsp;You'll note a few AWS API timeouts. Proxy Cannon will now attempt to recover its state automatically without immediately giving up.

Rotate Nodes (-r) in action.  You'll note a few AWS API timeouts. Proxy Cannon will now attempt to recover its state automatically without immediately giving up.
Save iptables state

ProxyCannon makes use of iptables to do its natting. In the past, if you ran it, your previous iptables state (if any) was lost. With this update, your previous iptable state will be saved before running, and restored when finished. 
Better Signal handling

We've added better (not perfect) signal handling, so that if/when an issue arises, ProxyCannon will attempt to cleanup up gracefully, restoring your system to its previous state.  
WAN IP Logging

If you ever want to know what WAN IP's your traffic took you can use the -l switch to record a log to the /tmp directory. 
Conclusion

Take a look and try it out.  If you run into any problems please submit an issue to github.
