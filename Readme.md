# 1️⃣🐝🏎️ The One Billion Row Challenge

.NET implementation of https://github.com/gunnarmorling/1brc

Runs in 4.8 seconds on 6 cores i5-12500/64GB RAM/Firecuda 530 (busy machine with 30+GB RAM used and YouTube music playing)

Running the top OpenJDK version (`calculate_average_ddimtirov.sh`) locally gives 5.75 secs vs 11.88 reported in the Java repo, so the ratio is x2.07 and my result would scale to **9.9 secs**.

## Results

**First attempt**

Mmap + paralell using Span API and some unsafe tricks to avoid Utf8 parsing until the very end.

```
Processed in 00:00:10.6978618
Processed in 00:00:10.8473143
Processed in 00:00:10.9107262
Processed in 00:00:10.9733218
Processed in 00:00:10.5854176
```

**Some micro optimizations**

```
Processed in 00:00:09.7093471
```

Float parsing is ~57%, dictionary lookup is ~24%. Optimizing further is about those two things. We may use `csFastFloat` library and a specialized dictionary such as `DictionarySlim`. However the goal is to avoid dependencies even if they are pure .NET.

It's near-perfectly parallelizable though. On 8 cores it should be 33% faster than on 6 that I have. With 32GB RAM the file should be cached by an OS after the first read. The first read may be very slow in the cloud VM, but then the cache should eliminate the difference between drive speeds.


**Use naive double parsing**

If we can assume that the float values are well formed then the speed almost doubles.

```
Processed in 00:00:05.5519479
```

**Optimized double parsing with fallback**

No assumptions are required if we fallback to the full .NET parsing implementation on any irregularity.

```
Processed in 00:00:05.2944041
Processed in 00:00:05.3489315
```

**Cache powers of 10, inline summary.init**

```
Processed in 00:00:04.7363095
Processed in 00:00:04.8472097
Processed in 00:00:04.8235814
Processed in 00:00:04.7163938
```


**Microoptimize float parsing, but keep it general purpose**

```
Processed in 00:00:04.4547973
Processed in 00:00:04.5303938
Processed in 00:00:04.5125394
```

