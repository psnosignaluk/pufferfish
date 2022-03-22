---
title: "Microservices"
date: 2022-03-21T22:53:17Z
draft: false
---

I've seen a lot of writing on microservices of late, where the issue has turned to one of concern.
The writing is largely because teams want to pursue microservices not because they make sense,
but out of the perspective that this is somehow best practise. In short, I agree with the
writers. Microservices are something you implement or iterate to over time as your platform dicates,
not because someone told you that microservices are a good idea.

A microservices cluster is put together because it makes sense to understand the context of the
platform in a series of services that mirror a complete whole. That is, ultimately, what a platform is,
correct? If you're building a platform, and if your platform is complex enough, then microservices
are what you want to be doing. By splitting each service in such a cluster into a unique service, you're
providing a roadmap of each service to your engineers to add to as the complexity of the platform itself
grows. This is how we handle code at [Token][1]. We run an Open Banking platform, in which all transactions
are a type of payment or compliance request, and each one results in either a payment being made or a consent
request being approved. By having the backend code split into microservices (with a few heavy dependencies) we can
gradually grow this service as the needs of the business dictate.

If you're not running this kind of workload, then chances are good that you're just barking up the wrong damn tree
when it comes to architecting your service. I'd seriously be asking myself what's easiest? How can I get this
app stack off of the ground without having to fuss about it? In a lot of cases, [Kubernetes][2] is going to be your
mate. That's got nothing to do with container orchestration, CNI, the freedom to pick a container runtime that
isn't Docker (because we don't shim amiright). It all has to do with logging, monitoring and application
tracing.

All of the cloud providers have their baked in tooling to help you monitor your stack, but its on top of [k8s][2]
that it gets flexible. Prometheus + Grafana are both an utter pain in the ass and an absolute godsend when it comes
to shipping usable metrics to monitor a platform. Depending on your application type, [Loki][3] might be the logger that
you're looking for, given that it indexes timestamps and labels and doesn't index the content of the actual logs. If not,
cast eyes on [elastic.co][4], which might just be your solution. There are a ton of third party services for logging and
metrics as well, such as [DataDog][5] and [New Relic][6], to name but two. For application tracing, you have the likes of
[OpenTelemetry][7] from the FOSS camp or something like [Instana][8] from the "we paid for this, it had better bloody work"
camp. Regardless of what you're doing in your tech life at the moment, the simple facts of log it, get metrics from it
and sample the source code should be first and foremost in your mind. [Kubernetes][2] is going to break. Your application
is going to give up and ghost you. You're going to need to see traces of applications where they tell you something useful,
such as why it took over 30 seconds for a database to spit out a result.

Ubnfortunately for all of us, running this on top of [k8s][2] is where we're all currently at.

[1]: https://token.io
[2]: https://kubernetes.io/
[3]: https://grafana.com/docs/loki/latest/fundamentals/overview/
[4]: https://www.elastic.co/
[5]: https://www.datadoghq.com/
[6]: https://newrelic.com/
[7]: https://opentelemetry.io/
[8]: https://www.instana.com/