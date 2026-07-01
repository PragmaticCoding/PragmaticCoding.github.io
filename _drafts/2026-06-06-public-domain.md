---
title:  "You Need a Public Domain"
date:   2026-06-10 12:00:00 -0500
categories: homelab
logo: /assets/logos/JavaFXLogo.png
permalink: /homelab/public-domain

RPI0: /assets/homelab/RPi0.jpg
Diagram: /assets/homelab/SnapCastDiagram.png
SHIM: /assets/homelab/audio-dac-shim.webp


excerpt: "If you are self-hosting, then you should probably have a public domain registered.  In this article, we'll look at why you should do this, and the things that it helps."
---

# Introduction

As someone who has already gone through the process of registering and managing a domain for my public website, specifically "pragmaticcoding.ca", it was a fairly trivial step to register one for personal use.  I already had a GoDaddy account from years and years ago, so it was just a matter of finding a domain name and charging it to my account.  

But why would you want to do this?  How can it help with self-hosting?  Let's take a look...

# What is a Public Domain?

A "public domain" (my term, since I couldn't find a better one anywhere on the web) is any domain that is registered into the system of domains that are managed by the Internet Corporation for Assigned Names and Numbers (ICANN).     

Everyone is familiar with the "top level" domains.  These are things like `.com` and `.net` and `.uk` or `.ca`.  There are also some subdomains of these that you see quite often.  You might have seen British domains that are part of `.co.uk`, because in the UK, all of the commercial sites have been forced into `.co.uk` domain.  In Canada you might see a `.on.ca` or a `.bc.ca` which are domains for Ontario and British Columbia.  Back in the early days of the internet, to get a `.ca` domain, you needed to prove that you did business across several provinces.  Otherwise they told you to register with your provincial domain.  They don't do that any more.

If your domain ends with any of the ICANN top level domains, then it's what I would call a "public domain".  

And you should have one.

## Registering a Domain

Each of these higher level domains maintain a registrar.  If you want to get a subdomain of one of these domains, you must put in a request with the registrar.  In practice, nobody deals directly with the registrars.  Instead you deal with a company that acts as an agent (or maybe a delegate of a registrar), and you pay through them.  These are companies like GoDaddy, and NameCheap.  In Canada there is one called "Register.ca", which isn't the `.ca` registrar, but just another agent.  You can register domains in `.net`, `.org`, `.com` as well as `.ca` through "Register.ca".  

Who you pick is up to you.  It literally doesn't matter which one you pick, but it would probably be best if you pick one that's likely to be around for a few years.  This is because the future management of your domain is going to be connected to your account at that agent, and tranferring it might be problematic if they go out of business with little warning.  Most of these companies also provide a range of other services like email and web hosting.  Prices for the same domain may vary between agents, too.

Personally, I use GoDaddy.  This is mostly because I was involved in a commercial operation many years ago that required me to set up an account with them.  I still have the account, and it was easy just to use it.  I'm not sure that registrations with GoDaddy are the cheapest, but they'll probably be around for a while.  

Depending on the domain name, and the top level domain that you register with, you can expect to pay from $15 to $30 a year for a domain.

## Domain Name Services

The important thing to remember about this is that all of that registration stuff is just about the *names*.  But these names are important because they provide the structure to allow everyone on the Internet to find the actual addresses of services they are looking for.  This facility is provided by "Domain Name Servers", often just called DNS servers.  

The one thing that every public domain must have is a DNS server that can service public requests for information about the domain.  For the most part, the registrar of the parent domain that holds your domain will have some level of DNS services for your domain, although that may just redirect to the agent through which you registered the domain.

Generally speaking, information about your domain is stored is something called a "Zone File".  A "zone" is usually just a domain.  The kind of information held in a zone file would be the addresses of the DNS servers that it uses, the addresses of the email servers that it uses and address of web servers, or application servers that it controls.

This is where the public part becomes very important.  Anything that you put into your zone file can be seen by **anyone** in the internet.  This is important for things like email handling, because you want email servers anywhere in the world to know where to connect to deliver email to your domain.

This public visibility is why you need a public domain.

# Email Handling

Years and years and years ago, my wife and I got our first email addresses from our ISP, which was Bell Canada, and they had a service called "Sympatico".  This meant that we had `name@sympatico.ca` addresses.  Over time, the "Sympatico" service disappeared, but those email addresses continued to work as part of the Bell service.  However, we had migrated to Gmail 20 years ago, so those `sympatico.ca` addresses were largely unused and forgotten over time.

If you live in Canada, then you probably know that Canada is especially poorly served with both cell phone and internet services.  There's a tiny handful of providers, they're ridiculously expensive, and the service is low quality.  

About a year ago, I finally became exhausted with the cost of my internet and TV service from Bell when the monthly bill increased by yet another $5 a month.  Also, Bell is the fibre provider for most of Canada, and in my neighbourhood, which is part of a large metropolitan suburban area, they haven't bothered to install fibre to the home.  This means the fastest speed possible was a paultry 25Mbps.  

As a result, I cancelled my fibre connection, held my nose, and signed up with the only alternative which was cable internet.

I didn't really think to much about it, but it was clear to me that as soon as I cancelled the service with Bell, those `sympatico.ca` address were going to cease to function.  

I'm suprised at how quickly the fallout happened.  Just a couple of days, in fact.

My wife asked me, "Did anything strange happen with email, I'm not getting email updates from Expedia?".

It seems that her Expedia account was so old that it still used her `sympatico.ca` address.  Oops! You cannot change the email address on an account without having access to the old account because they send it a confirmation email that you have to acknowledge.  We had some accumulated reward points that we didn't want to lose, and Expedia's Customer Service told us that she'd have to create a new account and then they could transfer the points over when it was set up.  So that's what we did.  

I'm sure that lots of you have email addresses provided from your ISP's.  I'm sure that you also have lots of accounts with various websites that depend on those email addresses.  

In a way, you're locked in with those ISP's unless you're willing to go through the pain and suffering of updating all of your accounts on every website that you use.  And remember, you have to do this **before** you disconnect the ISP address.  This could be a real bother if you're moving and don't have the option to keep your old ISP.

You could one of the other services, like gmail.com or hotmail.com, but then you're tied in to those.  What if, like me, you're fed up with Gmail?  They were cool back in the early, "Do no evil", days.  But those days are long gone.  

Having a public domain solves that problem.  It also means that for as long as you own that domain, you'll never have to change your email address ever again.  You can have that email address for life.

It also has one other key benefit:  There's no doubt that having your own domain-based email address is much more appealing than having whatever email address you could find on a public service.  Think on the simplicity of `george@thesmiths.org` over `georgesmith87302@hotmail.com`.

There are two approaches you can take to achieve this.  But first, one idea you should not pursue...

## Don't Manage Your Own eMail Server

For decades, I was involved with managing the email server for the company I worked for.  In the beginning, it was really the only option, because hosted email services from providers like Google and Microsoft simply didn't exist - especially in the corporate space.  We installed a Lotus Notes server back in the 1990's, and it was comparitively easy.  Just set up the DNS to point to it and configure the Lotus Notes server and turn it on.  

Over the years, it became more and more complicated to manage this server.  The big problem was spam - more specifically, the measures that the rest of the world were using to combat spam.  This meant that we had to be very careful to make sure that we didn't get blacklisted in the spam filters, and I can tell you this was a shock the first time that it happened.  

In some ways we were lucky that we had been running our own server for years, because we had established ourselves as a legitimate source which avoided a lot of hassles.  

Technically, it's not difficult to set up and run an email server in your homelab today.  It's not like you'll have to navigate the archane complexity of `sendmail` or anything like it.  On the other hand, all the reports I've seen say that you'll spend way too much time chasing around delivery problems caused by anit-spam measures implemented by your email recipient's services.

Finally, the other big downside to hosting your own mail server in your homelab is that, by definition, it has to be directly accessible from the Internet at large.  This means that you absolutely have to open up a hole in your firewall to allow SMTP connections to come in from anywhere in the Internet.  Not through a VPN, straight in.  With care, this is something that you *can* do, but for me this is something that I'd *rather not* do.

Just let somebody else handle the headache of running an email server.

## Mail Redirect

This is a good alternative if you aren't ready to ditch your Gmail account (or even your ISP's email) but you just want to be able to use addresses from your own domain.  You can set it up for free, and it works fairly reliably.  

One caveat to this approach is that not all of the mail services that you might currently be using support the outbound redirection scheme described below.  If this is the case with your mail service provider, you'll have to go with the second solution that I describe.

### Incoming Mail

In order to have mail sent to `somebody@yourdomain.org` you'll need to use an "Email Forwading" service.  Your best bet here is to simply do a search for "email forwarding service" and find one that appeals to you.  For what it's worth, I'm using `improvMX.com` and it seems to work fine.  It's free, too - at least for forwarding just two addresses.

You'll need to set up an account with one of these services and then use whatever interface they give you to set up "aliases" for the email addresses that you want to forward.  The alias is the address that you want have people send to, and the forwarded address is the actual address in you email service that will receive the email.  For instance you might set up `somebody@yourdomain.org` to forward to `georgesmith9983203@gmail.com`.  

The last step is to set up your domain to enable the email forwarding and to specify the forwarding service as the mail handler for your domain.  This involves adding entries to the zone file for you domain in your public DNS server.  

Let's look at the how to direct email to your forwarding service...

This is done via `MX` entries in your zone file.  `MX` stands for "Mail Exchange" and each `MX` entry lists the URL of a mail server for your domain.  You can also specify a priority for each mail server.

Somewhere in the documentation or setup pages for the forwarding service, you'll find information about how to set up your domain.  This will include the URL's of the mail servers that you should use.  For each one, you create an MX entry in your zone file.  The service I'm using specifies `mx1.improvmx.com` and `mx2.improve.com` with the first one having a priority of `10` and the second one with a priority of `20`.

How you enter the addresses and priorities specified by the forwarding service you select will depend on the UI provided by your DNS service.  

Finally, you'll need to set up something in your DNS zone that tells the forwarding service that they are authorized to perform the forwarding.  At least, this is what ImprovMX.com wants, and I expect that others will too.  For ImprovMX.com, they want a `TXT` entry with a specific string.

A `TXT` entry in your zone file is just text string that anybody in the Internet can read.  The important thing is that nobody else can *write* entries in your zone file except you.  So if the `TXT` entry that they are looking for is in your zone file, then *you* had to have put it there.  For my ImproveMX.com account, the string is "v=spf1 include:spf.improvmx.com ~all".  Presumably, this isn't just authorization, but some sort of configuration as well.

### Outgoing Mail

Not all email service providers support alias accounts for outgoing emails.  Gmail does, and I strongly suspect that all of the other big, big providers do as well.  

In order to do this, however, you are going to need an SMTP relay service.  I've been using Smtp2Go after having issues with Brevo, but there are quite a few free services available.  Once again, do an Internet search for "smtp relay service" and you'll quickly get lists of services that you can use.  All of these free services have limits on how many emails you can send in a time period, typically daily or monthly.  Smtp2Go, is pretty restrictive at 1,000/month, but even that is way more than I'll ever need.

Whatever service that you pick, it's almost certainly going to require that you create an account.  Then, of course, it will need you to prove that you own the domain from which you are going to be sending emails.  That's going to involve creating some specific entries in your DNS zone file.  Smtp2Go uses 3 `CNAME` entries, and they give you the exact information with buttons to copy the data so that you can paste it into the GUI for your DNS server.  

The last prepratory step is to create a user/password that can send emails through the SMTP service.  These are the credentials that you'll be putting into Gmail.  

Now you're ready to set up the outgoing alias on Gmail.  Go into the web interface for you Gmail account at `https://mail.google.com` and log in.  Then click the gear icon, which should bring up a settings sidebar, which should have an link called "See all settings".  Click on that link, and you'll get a settings screen that has about 10 tabs at the top.  Click on the "Accounts and Import" tab, and you'll then see a section that says "Send mail as:".  Then you can click on the "Add another email address" link, which should bring up a pop-up window that allows you to enter the details of the new address.

Give this address an name and then enter the address that you're using from your own domain, for instance `somebody@mydomain.org`.  Make sure the "Treat as an alias" box is checked off.  

In the next step you enter the URL and port of the SMTP server, the user id and the password.  Then save it and you should be good to go.  Use "Compose" to send a test email to yourself.  

## Mail Service that Uses Your Domain

My migration path plan is to move my email handling from Gmail and over to a Canadian company called NorthMail (northmail.ca).  They are new, and I've contacted them a few times with some compatibility issues that I encountered, and they've always indicated that what I wanted was in their future plans.  

The first thing I asked them about was compatibility with FairEmail, which is the email app I use on my phone.  They now support it.  However, they didn't support the mail forwarding scheme described above for outgoing mail, although they indicated that they had requests for it.

I checked back last month, and they now have a "For Business" product which allows you to set up accounts within your own domain.  It will work just as well for families with their own public domains as for a business, and the price per mailbox is about the same as for an individual mailbox.  This is the direction that I'm planning to go, although it won't be transparent to my wife as she's currently using the Gmail app for email on her phone.  I'm going to flip her over to FairEmail first.  This means it will have to be a process.

Setting this up is going to be easy for me.  Just purchase the account with NorthMail and tell them what my domain name and email addresses will be, and then configure my public DNS service such that my MX entries point to the NorthMail servers instead of the ImprovMX servers.  

I'm sure that you can find similar services from thousands of other providers around the world.

# SSL Certificates

Another story from the deep past...

Back around 1997 the company that I worked for needed to register a domain and get an SSL certificate for our web server.  Back in those days you dealt directly with the registrar, and we had to prove that we did business across the country before we could register directly into the `.ca` domain.  We also dealt directly with VeriSign to get the certificate.

You could not get a certicate from VeriSign unless you could prove that you owned the website and the business associated with it.  I remember having to go get a copy of our articles of incorporation to provide this proof.  In truth, I cannot remember 100% if we needed them for the domain or for the certificate, but I feel like it was more likely to have been the certificate.  It *was* 30 years ago, though.

I do remember that the whole process was an ordeal that literally took weeks to complete.  

Nowadays, it's not even remotely difficult.  But you should remember that an SSL certificate does 2 things:

1. Proves that web server that you are connecting to is actually the one in your address bar.
1. Provides the keys to allow your browser to use encrypted communication with the web server.

Both of these are important, but it's the first one which provides the technical hurdles to getting a certificate.  

First off, certificates are issued by only a few **trusted** certificate authorities.  Your browser is going to have a list of certificate authorities that it trusts.  If it encounters a website with a certificate that is not signed one of these trusted authorities, it will display ugly warning messages.  This includes the "self-signed" certificates that some of your homelab servers may use by default.  

This means that you need to get your certificates from one of these trusted authorities.  Today, we have a system called "ACME", which stands for "Automatic Certificate Management Environment", that you can use to automate the process of getting a certificate from a trusted authority.  

Just like back in the 1990's, the key element that you have to prove to the certificate authority is that you own and control the web server and the domain for which you are trying to acquire a certificate.  There are two ways to do this:

1. Make a specific customization to the web server that the certificate authority can connect to and detect.
1. Make a specific customization to the zone file for the domain that the certificate authority can detect.

Item (1) is problematic.  Many of the servers for which you'll want to issue certificates are not going to be publicly accessible, and therefore cannot be connected to by the certificate authority.

Item (2) is trivial if you have a public domain.  

And remember, certificates expire.  And proper certificates expire every few months.  So this is something that you want to set up and then forget about.  You're not going to want to set up some whole in your firewall so that certificates can be re-issued at any random time.

# Dynamic Domain Name System

You are probably aware that if you are using a normal residential internet connection, then your public IP address will change from time to time.  Anything that you try to set up that relies on using the public IP address of your connection will need to make sure that it has the current IP address in order to work.

You can use a service like "Duck DNS" to do this.  That's going to require that you create an entry in their zone file, and you'll end up with a hostname like 'fred.duckdns.org'.  Then you'll need to set up a script that checks your connection's IP address and updates the DuckDNS.org zone file with the new address.  

This works, but you're tied into yet another service that you've got to keep track of.  

You can do *exactly* the same thing with a public domain.  In fact, if you are using OPNSense, this is just another service that you can set up in your firewall and it will run on a scheduled basis and keep your zone file up to date with the current address of your IP connection.  

This is called "Dynamic DNS" (or "DDNS"), because your DNS zone is dynamically adjusted to reflect changes in the IP addresses of the hostnames.

This is really useful if you are setting up an incoming VPN, because you'll need to let the roaming clients know where to connect to...

# Integration with VPNs

This refers to VPN's that you set up to allow access into your network from outside.  You might be familiar with using a VPN as a client, where you connect into a commercial VPN in order to anonymize your traffic, enter the Internet from another country, or to prevent your ISP from snooping on your connections.  Generally speaking, at some point the clients for these services are going to need to connect to a specific gateway or group of gateways in order to join the VPN.  

The same situation exists when you create a VPN for an outside client (or peer) to access *your* network.  You'll need a publicly accessible gateway that the clients can connect to.  But you cannot give them an IP address, because your ISP could change it at any given time.  This is where DDNS comes in to play.  

## Tailscale

The other implementation of VPN's that you are likely to use is "TailScale".  The TailScale service actually maintains its own DNS service for your TailScale network that only your TailScale clients can access.  This is great, and allows you to use URL's instead of IP addresses when you're setting up clients.

However, if you are like me and you don't want to go through TailScale when you're on your own WiFi this isn't going to work all by itself.  You'll want to connect directly to the server from your WiFi, and when you leave your house you'll want to connect through TailScale.  

If you have a public DNS server and a private internal DNS server (which you probably should have), then you can set up a situation where your internal DNS server has entries that point directly to the internal addresses of your servers, and the public DNS server contains the TailScale addresses.  

When you are on your WiFi, you'll be using your internal DNS server and therefore connect directly to your services without TailScale.  When you are outside your house, you'll be using your cell provider's DNS server, which will get your addresses from your public DNS server and route you through TailScale to your severs.  This is called "Split Horizen DNS".

# API Tokens

Some of the things I described require that you are able to update your DNS zone file in an automated fashion.  This is usually accomplished through API tokens, which provide a way for applications to connect to your DNS service and make changes to your zone file.  Essentially, they are fairly long strings of gibberish which are unique to your account with the DNS service.  

Generally, you can create multiple API tokens.  This allows you to have different tokens for different purposes, and you can usually disable them, or have them expire, or delete them whenever you want.  

Not all DNS services support API tokens, so you'll have to be on the lookout for this when you pick a DNS service.  GoDaddy, for instance, only supports API tokens for what they call "premium" clients - those clients with 50 or more domains registered through them.  I think this is a change in the past few years.  

You don't have to pick your registrar agent with this in mind, though.  It's relatively trivial to pass the handling of your DNS services off to any other service provider that you want.  I chose to use deSEC.io, which is in Germany and is also free.  I also see it popping up as an option in places where you need to pick your DNS service for automated processes.  So it's a good choice for me.

In my GoDaddy account, I just go to the area for my domain, pick the "DNS" section and plug in the servers for deSEC, which are `ns1.desec.io`, and `ns2.desec.org`.  That's it.  From then on I just use my account at deSec to manage my DNS.

# Conclusion

Having your own public domain with a public DNS server is a great idea because it removes a lot of friction from a few key infrastructure tasks that you're going to need to do to run a homelab.
