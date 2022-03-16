---
title: "Cloudflare Pages"
date: 2022-03-16T17:12:57Z
draft: true
---

Configured Cloudflare Pages for the talking face of this service. It's a relatively simple thing to get going.
In my case, running a Hugo-based SSG, its a case of selecting Hugo from the options menu and setting the `Build Command`
and `Build Output directory`. Note that in my case, I had to change the `hugo` build directive to `hugo -D` in order
to get this to work properly for me. Have a looksee at the configuration I've used for this:

![Build Screenshot](/build-screenshot.png)

Once you have this set up, it's a quick matter of running the builder to deploy the code to Cloudflare.

```
10:35:42.910	Initializing build environment. This may take up to a few minutes to complete
10:35:44.898	Success: Finished initializing build environment
10:35:44.898	Cloning repository...
10:35:47.096	Success: Finished cloning repository files
10:35:47.385	Installing dependencies
10:35:47.396	Python version set to 2.7
10:35:51.273	v12.18.0 is already installed.
10:35:52.666	Now using node v12.18.0 (npm v6.14.4)
10:35:52.998	Started restoring cached build plugins
10:35:53.014	Finished restoring cached build plugins
10:35:53.710	Attempting ruby version 2.7.1, read from environment
10:35:59.538	Using ruby version 2.7.1
10:35:59.899	Using PHP version 5.6
10:36:00.069	5.2 is already installed.
10:36:00.098	Using Swift version 5.2
10:36:00.099	Installing Hugo 0.54.0
10:36:00.765	Hugo Static Site Generator v0.54.0-B1A82C61A/extended linux/amd64 BuildDate: 2019-02-01T10:04:38Z
10:36:00.770	Started restoring cached go cache
10:36:00.791	Finished restoring cached go cache
10:36:00.948	go version go1.14.4 linux/amd64
10:36:00.968	go version go1.14.4 linux/amd64
10:36:00.973	Installing missing commands
10:36:00.973	Verify run directory
10:36:00.973	Executing user command: hugo -D
10:36:01.074	Building sites … 
10:36:01.074	                   | EN  
10:36:01.075	+------------------+----+
10:36:01.075	  Pages            | 10  
10:36:01.075	  Paginator pages  |  0  
10:36:01.075	  Non-page files   |  0  
10:36:01.075	  Static files     |  5  
10:36:01.075	  Processed images |  0  
10:36:01.075	  Aliases          |  4  
10:36:01.075	  Sitemaps         |  1  
10:36:01.075	  Cleaned          |  0  
10:36:01.075	
10:36:01.075	Total in 19 ms
10:36:01.085	Finished
10:36:01.085	Note: No functions dir at /functions found. Skipping.
10:36:01.086	Validating asset output directory
10:36:01.730	Deploying your site to Cloudflare's global network...
10:36:04.824	Success: Your site was deployed!
```

Now this is where stuff gets interesting. For approximately $0 per month, my code
lives in a server that's close to home for me (London, UK), but also pretty damn close to
other people across the globe.

Here's a `curl -I https://pufferfish.cc` lookup from my machine against the site:

```
curl -I https://pufferfish.cc
HTTP/2 200
date: Wed, 16 Mar 2022 16:46:49 GMT
content-type: text/html; charset=utf-8
access-control-allow-origin: *
cache-control: public, max-age=0, must-revalidate
referrer-policy: strict-origin-when-cross-origin
x-content-type-options: nosniff
expect-ct: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
report-to: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v3?s=cDIpBFyVWaP%2Bz4w%2Fx7hFrh1GSV6082ZUB50YFatLDIxvCfb1qsJUwEK94%2BUaTo6SucyLI7pxNAd2jfdH%2FTo1%2FkYZNRhQIVUWaQQeMSN55uGqoERMORxGThV7kVW%2FLmOq"}],"group":"cf-nel","max_age":604800}
nel: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
vary: Accept-Encoding
cf-cache-status: DYNAMIC
server: cloudflare
cf-ray: 6ecef796ce8f71d5-LHR
```

Here's a lookup against using a NordVPN AMS server:

```
curl -I https://pufferfish.cc
HTTP/2 200
date: Wed, 16 Mar 2022 18:00:33 GMT
content-type: text/html; charset=utf-8
access-control-allow-origin: *
cache-control: public, max-age=0, must-revalidate
referrer-policy: strict-origin-when-cross-origin
x-content-type-options: nosniff
expect-ct: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
report-to: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v3?s=Ppc1Xq%2BLol98w16kPYhlKcSJqfrzvCgIktiFTXygjTjukU2Ten5ZG7kMzNxPq%2B%2FlkyvQo5DQrMdVVHltQcCk5ZoGegMbn1IKNGPwnYaCK651yPmpahhsAgCmqcvTUtqp"}],"group":"cf-nel","max_age":604800}
nel: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
vary: Accept-Encoding
cf-cache-status: DYNAMIC
server: cloudflare
cf-ray: 6ecf639b4e351e6d-AMS
alt-svc: h3=":443"; ma=86400, h3-29=":443"; ma=86400
```

All of this for free and for nothing. That is awesome.