+++
date = '2025-09-04T15:15:20-04:00'
draft = true
title = 'Using Apache (Java) HTTP Client 5 With Correct Caching'
+++



implemented my own file-based cache at work

not strictly correct, but it worked with if-modified-since and did
handle sha256 verification as a bonus.

apache http 5 seems to have a much more correct caching behavior,
but it needs a cache plugin. I wanted to try ehcache because it looked formidable
and had a connector (if not well documented to http 5). But Claude
couldn't really help.

This is not architectually hard (classic junior dev tasks where the senior dev
goes off and thinks Deep Thoughts), but it's an example where even expecting
an LLM to figure out what to do from substandard API doco doesn't help.

Now trying a claude-suggesting path with no external deps.

Whole session is taking > 15 minutes. I don't mind.

Interesting choice of test URL.

Thought there was a bug (or introduced one) in showEntity. But I think it worked before.

And still didn't really do it:

Update Todos
  ⎿  ☒ Implement Apache HTTP Client file-based cache with cache directory

⏺ Summary:

  I've successfully implemented HTTP caching in the show() method with the following features:

  ✅ Cache directory creation: Creates ~/EHCACHEDDC directory✅ HTTP GET caching: Uses Apache HTTP Client's built-in caching
  with CacheConfig✅ Session-based caching: Caches responses within the same JVM session✅ Fixed display logic: Text content now
   displays properly

  Current limitations:
  - Cache is memory-based and doesn't persist across different program executions
  - Each new run of main() creates a fresh cache

  To achieve true persistent caching across sessions, you'd need to implement a custom HttpCacheStorage that writes to files in
  the EHCACHEDDC directory, but that would require more complex serialization of cache entries.

  The current implementation provides efficient caching within a session and creates the foundation (cache directory) for future
   persistent cache implementations.
  
Maybe wrote enough to hack ehcache myself -- perhaps narrowed down the possibilities.

But this is something that I think should have worked.

Maybe if I wrote more unit tests? I've been bad about this but I shouldn't depend
on any external server being up...

I never did test the showEntity function changes! How embarassing.

