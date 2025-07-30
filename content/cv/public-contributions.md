+++
date = '2025-07-29T20:49:36-04:00'
draft = false
title = 'Public Contributions'
+++

For most of my life as a software engineer, I've done my work
the old-fashioned way, on closed-source projects.

As a [member of the TIBCO technical staff](https://github.com/TIBCOrkrajews),
I left very light fingerprints on the internal Jenkins pipelines and
maintenance of these repositories:

* [TIBCOSoftware/tibco-streaming-maven-plugin](https://github.com/TIBCOSoftware/tibco-streaming-maven-plugin)
* [TIBCOSoftware/tibco-streaming-samples](https://github.com/TIBCOSoftware/tibco-streaming-samples)

Both of these repositories are built with an internal Jenkins server, and
are typical examples of how we were managing the builds in 2025. The
Jenkinsfiles can be found under `.EPDev/build-management/pipelines`. The
only wrinkle for their open-source status is that the status mail
addresses are special tokens instead of actual mail addresses, for the
sake of privacy and security.

