# 1.3. I/O Latencies

## Exercise 1.3.1

**Latencies for remote I/O**

> [!todo] **Exercise 1.3.1**
> 
> `time` and `strace` the `curl` command to retrieve the CERN main page **home.web.cern.ch**. Use the strace option to display the real time as the first column `-ttt` and restrict the syscall output to network related calls `-e network`.
> 
> Bash
> 
> ```
> time strace -ttt -e network curl home.web.cern.ch
> ```
> 
> **a)** Identify the network latency and explain the execution time of the command.
> 
> **b)** Can you understand the HTTP protocol for the command? What is the syntax used to get a response for an URL?

> [!success] **Solution**
> 
> Look at the time passed between two **strace** output lines. There is a ~70ms delay for the `connect` and another ~80ms delay for the `sendto`/`recvfrom` call to retrieve the contents. The absolute numbers depend on your network connectivity.
> 
> Bash
> 
> ```
> 1539170160.031625 connect(3, {sa_family=AF_INET, sin_port=htons(80), sin_addr=inet_addr("188.184.37.205")}, 16) = -1 EINPROGRESS
> 1539170160.107213 getsockopt(3, SOL_SOCKET, SO_ERROR, [0], [4]) = 0
> ...
> 1539170160.107329 sendto(3, "GET / HTTP/1.1\r\nHost: home.web.c"..., 80, MSG_NOSIGNAL, NULL, 0) = 80
> 1539170160.187182 recvfrom(3, "HTTP/1.1 302 Found\r\nDate: Wed, 1"..., 102400, 0, NULL, NULL) = 470
> ```
> 
> You can actually see the HTTP protocol in the output. The client sends a request like `"GET / HTTP/1.1\r\nHost: home.web...."` To see the full request, you can add `-s 1024` as an **strace** option.

---

## Exercise 1.3.2

> [!todo] **Exercise 1.3.2**
> 
> You have **straced** an analysis application (e.g. ROOT) using files on your local harddisk. You see that it issues synchronous read calls with the following statistics:
> 
> Plaintext
> 
> ```
> % time     seconds  usecs/call     calls    errors syscall
> ------ ----------- ----------- --------- --------- ----------------
> 19.30    0.039031           1     20000            read
> 14.61    0.000143         143         1         1  connect
> ```
> 
> The realtime of the application running with local data is **10 seconds** and it is CPU bound.
> 
> Assuming the application uses the same I/O model for remote file I/O, which runtime do you expect if the network RTT is **20ms** instead of local disk?

> [!success] **Solution**
> 
> You have to add **(20000 + 1) * RTT** to the realtime because the application will have to wait for each read request at least 20ms defined by the round-trip time.
> 
> ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAqEAAAA8CAYAAABWxWPqAAAQAElEQVR4AezdBZA0z1nH8cXdNbgFd7dCgru7hBAghbtb4RRWJFBoUUhwJ3iAwgungCK4u7trfp/LO/+ad969u5m73b3Z299VP9czPdM93d/umX7m6e7Zx970rwRKoARKoARKoARKoAQOTKBK6IGB93IlUAIlsNmUQQmUQAmUQJXQtoESKIESKIESKIESKIGDEzi4EnrwEvaCJVACJVACJVACJVACqyNQJXR1VdIMlUAJlMDOCTTBEiiBElgdgSqhq6uSZqgESqAESqAESqAEbj+B26+E3v46bAlLoARKoARKoARK4OgIVAk9uiprhkugBEpg/QSawxIogRK4jECV0MsI9XgJlEAJlEAJlEAJlMDOCVQJ3TnSJlgCJVACJVACJVACJXAZgSqhlxHq8RIogRIogfUTaA5LoASOjkCV0KOrsma4BEqgBEqgBEqgBI6fQJXQ46/DlqAESqAESqAESqAEjo5AldCjq7JmuARKoARK4OYJNAclUALXJVAl9LoEG78ESqAESqAESqAESmAxgSqhi5E1QgmUQAmUQAmUQAmUwHUJVAm9LsHGL4ESKIESKIH9E+gVSuDWEagSeuuqtAUqgRIogRIogRIogfUTqBK6/jpqDkugBEqgBEqgBErg1hGoEnrrqrQFKoESKIESKIHrE2gKJbBvAlVC90246ZfAYwg8XryPjrxEpK4ELiLwFjn4kMhjRepKoARK4NYSqBJ6a6u2Bbs6gZ3HpIB+XlL978gvRepK4CICj8hBLysfGr+KaCDcoHPvPnWu33oIhLoS2DWBKqG7Jtr0rkLgWRLpOyJvFNnmdAAPyIFfjvx55C8jnx55ksgxuAcnk/eLUET/P/7gnjwbnxv5r4jwP43/vpEniozdkvIPaf5dEsDqZ+K/UmTqlqQ5xH3BbPxw5KUia3DazTckI38d+avI90aeNzLXzY0/ML2snlx3bprOPU+8rHxcDr5V5DUja3SXtYXHSabfLvIHEe3wj+K/T4RSF+8onHvkM5PTb41MnzVDm9jHfZbL3eXmtil18mOJ6VlCfirbrxJRjnhH4JrFkyNQJfTkqnw1BX7x5IQCQfH6w2y/ceQ89+Y58DWR94xQ5p4r/nNEvi3ypJF9uCdMoiTetdzzJPYHRD4r8u+RwT1FNr4u8qiIDu5p41PEHxb/WyLjcs0tvzR1mOI+c9LA6mPjf3tkyndumq+XuOJT9H412y8bWYN79mTiByIsy88Y/xki3xz58QgrYrwL3dz4mM6tp7lpXpixOwcp1Q/N9idFniayBje3LVB6PjgZpkhTorVD7eYdE6Z9P278fbgnS6KU33g7cfLupXCamDax6/tseo1hf26bwvfLE+n9Ivi+WHzPrx+N//6RuhJYJYEqoauslpPIFKXmC1PSF42YKxlvq/PANyz5fTnqzT7e5l/zj4Xi5eO/YWQf7rWS6C4e3jpe1qCfT3pj95bZoUyz7rJ8/W32Pyyi09DZD+VaUn7xdEa4DgovRe27ku5HRaQVb8Ofy1TePzWR7h95eGQt7kHJCIvxV8X/3wjLj5eaX882jjribJ7r5safW08uNDdN584RVueny4naYrwbd3PbghdEytBXJMe/E+H+Iv+0owfGf5nIPpz0vdzuIu2nTyJeACjU2bzL7eM+u+sCo505bYqy+ZGJ85uRP4m4H34l/rtF/i3yIZHnjNSVwOoIVAldXZWcTIZYQH8kpaV8USCyudXpsChWv5Gj4/P+LPt/H6EkXKZw5LQbcU+Vq75BhDIxKIXZPXOvkP/PH9FxsoRmc/Mv+ffICKej488tv47onRIBE2yyeeYww84Q+ouchWw2c9N0OqXuZ7PxD5GrOJ04KyX/ovjm3VEqLzpnOMZqbKiaUvSPQ2B8/FjVXy3bzxY5zy2JP7eelqR5Xr6m4erR8Opa2vjctvDaKQhL/K/FHzv3vPY4vGCNj61pW1v1kvaNydQfR8ZuX/fZ+BrD9tw25aXSy7z7/52HyPF/K/KLEXXxwvHr7iHQgJsmUCX0pmug17+MAOVJp+ANf9u5hl6H4UrnPVNOMrdUR2jelnlSN6WkvlDyYjj+J+NPnaH3/0wgBZGfzTPHimGDYirfc8tP4dXRGMb9DwlMRFqUecFz03TudcXw6MckkQ+KqJ949zj5wmOox3tOmAQYomRt+5uEbyur4fnny7Hz3JL48qV+LqunJWnKl7Jqo9qqbXNZp0o4he2nczKLvzJl8yjcKyaX/xPxQhTvHvdyCdG+423MEXWfULS1S+3YCIFjNyWG4Q3tmxYzzYP8Heo+m9umvMibD62d/vYow9rP8DzxLBwOrZH5kLf6J0agSuiJVfgRFtcwsGz/k38joXxQuJ44YTo01gDDsZ+c/SeIUFJYAb4g26wX8Ra54bqLIk1O1rnKJ4vW5NDmOxMgXxZr6LCzu6GksY7aNv9S+JCPy8pv+FAHaaqC4X1pDDIo8JQdYff3L3JZmjnl2k4ZDI9TClmXlHGcKAX0cxJg2sKQz+xe6JRVvSur9McnD2mwrI7Dx9tL4s+tp7lpUgDMDzZULY+UAwqEaRNPOc7kne1hzrD2fCdo1Z57keXNIq6p9Z9Sqs25Vx8/pTB0/gvxXzfCmads0SErt/0l4rpeQJfE2Xaue8Tcc3WkDNNz1POh7jPXmtPO3QOeI54n2uuQZ5y1G+X4vTuBu2R+J8l6JXB1AlVCr86uMQ9DQKe97UoULUqITlynYPjaA9eCCAtUviSR3jTCShBvsTvvuksSMiRsTpZ8zon33DnpdSKsGRYZZPPMUsSfyrT8LJ2sjtPz7Ov4+VbZ8s8r2zRN5+5CKCPm17IKjxXRQQF9h1zEMHq8WU5ZzztxWtZt5103/rZ6mpvmSydDbxLB4+vjW3DHGkrZzO49DjsKG4XknoOTgLfJ/u9GrESfK84XL9F24rxknMdCWShEymLx3IfnipSmz47vnjU3nMU8u4ud657X/ucmJg1zWS06GuayTuMq23nXmba9695nrjW9/rA/vdYQPvZfPTusqV8Z35x06e2SeZKtW0igp08IVAmdAOnu0RJgBWF5NBQ4dBKsYt+TErEUxDu4o/SZUzZHCdVhsRgakmUVXKKUHbxgV7ggBYTiNSiihmRZQN8laR1TWa9bT6ydhpsNxUsrxd+Y12roFyP7Y2FFN+1gsIiPj023TR145QRiO1fcL5SuRDuoo1hbdPUauep40cxPZN+imngHd9g9a65qRCXeUTvKp68T/FBK4cXPC6YRorUxT/bqTplAldBTrv3bVfbvT3FYHa0E15mbS/f6CfvaiCHxePc4CoHvjbKaTsWQIJmG2//4pMSSE28njgXmA5MSy608WwiU3VvkHlMU9UIRNSRo0ce7Jvg8i1MOrc7top4MP1vg4wsG2iWF671T0m+KXHXxV6KeOen5JucSMfxNQTlL4ID/3Kusn5RlQ8W+t8kyTFHyJYzzsqLtuAen4hu8FtyZfzw9Zv9tz0vwTrih6/fKtoWCN8Ejl96ZU5YvS2q/H3nriJeceBsvw1dhLm6lBPZCoEroXrA20R0SYM2U3GA1sk3Mf/JWz0rEWmQ481VzwGpillCdm6HOr04Y62K8e5yHs089USqnYp4emYbb//yk5IEebyfOXDhD0qxSw2eohoTnln+Yb6fslKUhPp+VmD+sVp6bpji7FgvJDGfLi5+nlN+l11BW1m3Di9Oymo8oPenzt8lV419UT3PTNI/ZHEgKl0Ujvlhg/qEvRRim3pbfYwqjCLsn1cu0f/HS5+WNcvTPKdSXRt4+whJuSo1pAV4efVoowVud+9y3b92HY/mUnG0e9RfFH4cP215Oc+hcR0mlGFNyLZIiVvGb7mNRmBdEzxT1bChcu1XGcYK7vs9ca2k795z0TPPlCHmm3I/zeBXm4/jdPjICa8/u9CGx9vw2f6dHYFAmpsORHrY6NHPffJrnBYKGUkkRNdRn2Je17c0Svu0XgxK8sXrUnNFtliNpkm3HxBFXGpfJts5qHMdK3HdPgE85DVZBH5qm6Ork5pZfh0W5NA8VlyR5n9P52zHXlD83TefuUijZn5EEWWd8PN+QoTm8ypng2c5LB0WHwmlByjgiKxCLq+9SjsPH21eJf1k9zU1Tnq289mtCrH72KaGUc211nE/b6k59zrGS+nUcVr+lIp5r7UIoTb5X6cWP8jZO0yIbL48+1WQlt/mxfgjBXG7nm8PIKurHHXyeaBx32GalZL2d3pfqW71rF9Nj9t3LQxrbfM8RLwWmSQxiVEJ7sljJtAGr4g95n81tU0N5PBN9L9Rz0OIqPBwzz9YLlJe2qzCXRqUE9kKgSuhesDbRHRKwwt3bPIvEONmhc2b5ZFWh1BhOc47OxIILCo4HuQ5O+KGFsrdNKRzyYWGOoVjzIimQQ7jOTr4punPLr3NkRaWAT8tL0dFJ+3Uh15ibpnN3JYMCyuLL8kWZ8Gs05keas7ZEEcVGGSigOt4hjxQcysujNpvNoNA7TullfRyusSS+tOfU09w0XzIJ+gg6hUD9iveJCTMXUpvO5l2O4qYMlKS7DmzZMY1jsPwt8cXbktyVg3wXV9mm9yxrp7ZpnjYFXB59lsmFWFBZg7UFSrcyCz+UGPV4j1xsLL60QZFjfdVWDXHv6z5T3uu0U22b8p4ibPwwhfvLtnbmeWJuuvtjTczlr3LiBKqEnngDWEHxPTwpTrKig+KPhfWORdOvxrBIDMdYPFldzPkcwgypsaoM+zp1FhDDf0PYIX3z/XTE24ZZWQFNFTBkxlKj0x5EOEVNXpeU34p6nTuFT1zi2g/IhsUWA4claSbqmWPRlbb64p8FzvwnP4ZLBwV0iKaj1LkvVUQpLF+cRFi7dbDZPHOG+Vl6BmVBIAumsmtDPugtbEn8ufW0JE1fchiUL/mhaGonfj3L/lgM8eJkeHUcvm1bHrSlpSLetvTOC7usLRhS/8FE9hUAylU2N9qN4W0vZj5HJUy5H5yN4ZxsbpTXnNk5ll/n71P0j/KtvPzhWvu4z67bTlk6zW//hGTSFwiGZ4m2o70ZvcmhzdqZy2PlmAkszLubbGGUnl4COyGgEzav8v+SGktQvA1lwcOTAnY/ARGKpk+3sECwVlBkKDTEkBMrRU47cx60j8iWdHyWxGrhh2V/fE52D+YoDspHOZpe1Ly26RSD4RxlxsC+7bnlZ+l8SCI9NIIPVubCsYKwgGCbQ5slafp8kHjimLPnRcHH94WxXLFaSfM8Ya001cCvuQxlGp+rk6SIehaZUjE+dtH2d+fgp0UM5xpuZMGybaU3JTSHzhyLuHqQZ/5ZYP7NjT+3npLkZm6a6sN0C+1TXnG0GMdKZumMxYfqvTSwmI7Db2J7blvw4mdUwrdhLYTxcmiKgPbD6m9erPwbkqfk/1x2cMDDPf0R2WeBjHcjbng2sRBr71bNmwfqmSJD+7jPrtNO/SKZaS7uIfmbilEWz0/ha2Uub5UTJHBe8SZd3gAABB5JREFUoz1BFC3ygQlYJUxBYWGYio6JNWfIkuF487N0UB6ifl/dMDeFYzhHp27up7l1lBqfO/Lhdx+rpzAN5x3SZ3lkFWKJVMbxtS2+ELZNWIYMzw7nzym/c5XTKmu/vuO6WPncE3bmiTlnkLlp+o7jtjwK8/OYFI4hzW2+Fw1KtI5w23FhFFFWHFYy+3PElAsdr4Uk5hFSbHDWRqQ3pOGnC30CiDVozGBu/Ln15Hpz0mQFlGdD/L4OYDW3Nuv7jeN8S4+Cr01TUMd5d+wmZElbMB2CVdo3QL3APDwZds/6XmU2N9qFxT/mWlK0ccCDZZuS55ybkvOeTQ+8k6F93GfXaafmyHrRdU9uE18NoISumfkdtPVOjUCV0FOr8eMtrw5e58Cy8sgUwwM13n3Ovk7ceRQSD2b7952wcMPQKFkY7a7TDXOy8FCAxtME7jpp5o5yXVT+cTIswhQGrHRuOs3x8WF7SZpDnLX5rIosoMT20vyJIy6xvTT+tvOlIz1ie3yONkHUiXrysmV/fM6wbeW871Z6sRjCjsl3/5mzrR3y7Q/5V35zufnKjwMe9odzlvjScA33/ZJ41zlXfg91n2lH2hOxfZV8Y7tL5lfJQ+OUwF0Erq+E3pVcd0rg1hBgfSLXLZApBCx8Fh1cN63GPx0CFpSYL2nOs/ZzOiW/WklZW/34wUUW96ul3FglUAJ7I1AldG9om3AJnBEwt80H2i0c8ItOZ4H9VwKXEDCFwrxon29iwbrn9AaUQAmUwLETqBJ67DXY/B8DAXMWfYrJ3EjfHTyGPDePN0fAAq0H5fIW96xhLmiyUlcCJVACuydwhEro7iE0xRI4AAELMqzgtljoAJfrJY6YgDl/Pma/7WsCR1ysZr0ESqAE7iZQJfRuHt0rgX0SsJLcAop9XqNpHz+BdbaT4+faEpRACayMQJXQlVVIs1MCJVACJVACJVACp0CgSujltdwzSqAESqAESqAESqAEdkygSuiOgTa5EiiBEiiBXRBoGiVQAredQJXQ217DLV8JlEAJlEAJlEAJrJBAldAVVkqzVAIlUAIlUAIlUAK3nUCV0Ntewy1fCZRACZTAHAI9pwRK4MAEqoQeGHgvVwIlUAIlUAIlUAIlsNlUCW0r2GzKoARKoARKoARKoAQOTKBK6IGB93IlUAIlUAIlgEClBE6dQJXQU28BLX8JlEAJlEAJlEAJ3ACBKqE3AL2XLIESKIESKIESKIFTJ1Al9NRbQMtfAiVQAiVwGgRayhJYGYEqoSurkGanBEqgBEqgBEqgBE6BQJXQU6jllrEESqAESqAESqAEVkagSujKKqTZKYESKIESKIHbQaClKIGLCVQJvZhPj5ZACZRACZRACZRACeyBQJXQPUBtkiVQAiVQAiVQAiVQAhcTeDQAAAD//5OKlxMAAAAGSURBVAMAI+OjtZFgrW4AAAAASUVORK5CYII=)
> 
> Neglecting the disk read time, our application (which originally finished in 10 seconds) will now take **410.02 seconds** due to latency induced I/O inefficiency.

---

## Exercise 1.3.3

> [!todo] **Exercise 1.3.3**
> 
> A physics framework uses an optimization to improve remote data access.
> 
> Taking into account exercise [[1.3.2]], which kind of optimization could that be? What do you need to do to reduce latency effects in remote I/O?

> [!success] **Solution**
> 
> The simplest solution is to **reduce the number of round-trips**.
> 
> Instead of issuing **20k x 6kb** requests, an application can issue fewer, larger requests (e.g., **4 x 30MB**).
> 
> By using **large cache pre-fetch windows**, the remote I/O time is reduced to just a few additional round-trips plus the time to transport the data. This can be further improved by **asynchronous pre-fetching** (requesting data ahead of time), which masks latency bottlenecks during data analysis.
