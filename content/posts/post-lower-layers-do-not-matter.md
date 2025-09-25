+++
date = '2025-09-24T17:39:25-04:00'
draft = false
title = 'Lower Layers Do Not Matter Until They Do'
+++

One way to separate good engineers from the merely competent
is to throw them a problem where buggy interactions at lower
levels of the stack break the process with confounding,
seemingly unrelated failures, often in the guise of backtraces
spanning scores of lines.

A "favorite" example of this from my experience popped up when the
build of [our team's Eclipse-based
IDE](https://docs.streambase.com/latest/index.jsp?topic=%2Fcom.streambase.sb.ide.help%2Fdata%2Fhtml%2Fgettingstarted%2Fgs05-studiointro.html)
started failing in our pipelines once our company switched to the SaaS
version of JFrog. I was glad to be free from maintaining our team's
Artifactory instance, but I didn't expect such
a spectacular error
emanating deep from the innards of the Eclipse Tycho artifact resolver
code once we made the switch. For background, [Tycho](https://github.com/eclipse-tycho/tycho)
is the Eclipse
project's suite of Maven (Java build) plugins for putting together
Eclipse artifacts in a headless or command-line environment.

Perhaps it's not so bad that misery loves company. By
not-exactly-coincidence, [this
issue](https://github.com/eclipse-tycho/tycho/issues/4459) had
already been filed
with a candidate fix. I spotted this issue as being the same issue was our own
with the telltale string of `X-Artifactory-repositoryKey` in the error
message -- I probably just searched for this string inside Tycho's
GitHub issues, although any large search engine should have worked as
well.

The exact use cases are different operations within Tycho, either
mirroring in the cited issue, or during resolution when assembling our
application, all the fault of the same underlying code.

The underlying cause is that cache is layed out based on the URI being
cached, and that URI can become ridiculously long, with the path
inside the cache this having a name much longer than that which even
relatively robust file systems in Linux can handle. Now, normally, the
URI is _not_ ridiculously long [^1], but in the case of JFrog
Artifactory, what's happening is that the caching code is trying to
use the URI actually transferred after any redirects as the cache key,
which seems to make perfect sense at first. However, JFrog offloads
the serving of larger artifacts (but not all!) to a cloud service, so
the URI consists of some formidable base pathname and some kind of
encoding of temporary credentials, when tend to be formatted as long
token-ish strings.

[My colleague](https://github.com/kysmith-csg) (hi Kyle!) got involved
with the issue, and then [I
kibbitzed](https://github.com/eclipse-tycho/tycho/issues/4459#issuecomment-2515026899)
about the futility of using a URI with short-lived information as a cache
key. An interesting (well, interesting to us) discussion ensued about
whether using the original or effective URI for the cache key was in
fact the right thing.

It turns out that [the designers of HTTP knew what they were
doing](https://github.com/eclipse-tycho/tycho/issues/4459#issuecomment-2556924492).
The URI in the 302 (Found) response that JFrog and likely other servers that
delegate to cloud storage issue was perfectly justified in being
cached using the effective URI. The HTTP specification gives you
permission to treat the reference URI as the Found URI, and
caching is just one such (important) use.

At this point, Kyle deftly worked with the Tycho team to get this fix
back into the 4.x series, which we desperately needed. In fact, the
5.0 series in the main branch didn't get a release until late August
2025.

[^1]: But then there's Windows. The essential part of many Eclipse components can easily exceed 120 characters or more in length, what with reverse DNS names and timestamps usually incorporated, and so when copying these components around for installation especially, if the destination path is already about 60 or 70 characters, you can easily bump against old Windows pathname limits when copying files to such destinations. Even in 2025, there are still a shocking number of tools that do not truly support long paths in Windows.

