# introduction

I have been using Linux for around 8 years by now, I believe. But Linux's S3 (sleep) mode always worked somewhat flacky. Every once in awhile, it would just fail to sleep and I'd have to hold the power button to shut it down. I never understood why but I did discover a pattern: memory usage. When I started using Fedora, they used something called `zram` (compressed RAM) as swap memory. I had 16GB total memory and it was configured to have 50% of the 16GB as swap memory. Meaning I had 8GB of normal memory and 8GB of zram. Not sure if this mattered but I noticed that, whenever it entered S3 when the memory usage was at 50% or higher, it would fail to enter S3. If that was not the case, the chance was at least much higher.

# solution that worked for me

So I wrote a really simple program that would just allocate as much memory as possible and let Linux kernel kill it with OOM Killer. If my understanding was correct, this would clear all the swap memory. I had been using this for around 2 weeks now and it still hadn't failed to enter S3 since! Maybe I was just lucky this time or maybe it's because the weather is colder nowadays. But hey, if this doesn't actually solve the problem, at least I really liked seeing the memory usage dropped from 40% to like 23% after using it, lol.

The code looked like this:

```
#include <stdlib.h>
int main(void) {
    int *p = NULL:

    for(;;) {
        p = malloc(1024 * 1024 * sizeof(*p));
        memset(p, 0, 1024 * 1024 * 1024 * sizeof(*p));
    }
}
```

Then I just typed `make a` (if the source file is called `a.c`). I needed to use `memset` because, I believe, Linux kernel wouldn't actually give me the memory when I called `malloc`. It would only give me the memory after I actually used it. When compiling, I believe that `-O0` flag was also necessary otherwise `gcc` might just make it an infinite loop.
