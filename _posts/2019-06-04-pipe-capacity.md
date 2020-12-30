---
layout: post
title: "A Short Adventure Through Linux Pipes"
date: 2019-06-04 10:11:14 -0500
---

I was recently working on a Node application and ran into a really strange bug that turned out to be related to Linux pipes. This post describes that bug, and the solution I found for it.

For those of you that want the TLDR, just skip to the end and read the solution. 

This may not be new info for some people, but it was for me, so I thought it was worth sharing.


## The Bug

I was working on a load testing Node application that spawned a long-running child process called [k6](https://github.com/loadimpact/k6), which generated the load.

I configured k6 to send around 200k requests, but after sending around 300 of them, k6 would gradually slow down and stop sending requests altogether. I ran the exact same k6 command in the terminal, and saw no such issue.

*(Do you already have an idea of what the issue might be?)*

## Debugging

One of the first things I did when I noticed this difference in behavior was to attach a listener to the stdout of the k6 subprocess. In Node, that's done like this:

```js
child.stdout.on('data', (data) => {
    console.log(data.toString());
});
```

Strangely, all I saw was the default k6 splash screen, which is printed to stdout every time the program starts.

<img class="img-fluid" src="/images/pipe-capacity/k6-splash.png">

This was surprising, because I was calling `console.log` several times throughout my k6 script, but none of that output was showing up in stdout when k6 was run as a Node subprocess.

Like any programmer, I decided to look at stderr, hoping to find some clues. Little did I know that I would actually find the solution.

## The Solution

Using the same syntax, and simply replacing `stdout` with `stderr`, I logged k6's stderr to Node's stdout. 

```js
child.stderr.on('data', (data) => {
    console.log(data.toString());
});
```

Lo and behold, all of the `console.log` calls in the k6 subprocess showed up through stderr! This turned out to be a [bug](https://github.com/loadimpact/k6/issues/782) (or feature? 🤔) in k6, where calls to `console.log` go to stderr.

Another strange thing happened. k6 no longer got stuck! I commented and uncommented the above block several times and reran the program each time, just to check my sanity, and confirmed that adding this snippet "unblocked" the process.

After some more research, I finally figured out that the bug I'd seen had to do with [Linux pipe capacity](http://man7.org/linux/man-pages/man7/pipe.7.html). Unbeknownst to me and to several people I was working on this software with, Linux pipes (which is what stdout and stderr are), have a maximum capacity! According to the pipe man page, 

> A pipe has a limited capacity.  <b>If the pipe is full, then a write(2) will block or fail</b>, depending on whether the O_NONBLOCK flag is set (see below).  Different implementations have different limits for the pipe capacity. Applications should not rely on a particular capacity: an application should be designed so that a reading process consumes data as soon as it is available, so that a writing process does not remain blocked.

The subprocess' stderr pipe was simply getting filled up, blocking further writes, which in turn prevented the entire subprocess from making any progress.

I was curious what the actual pipe capacity was on my computer (a MacBook Pro running Mojave 10.14.4), so I used [this script](https://unix.stackexchange.com/a/11954/295710) from the Unix StackExchange, which outputted this:

```
write size:          1; bytes successfully before error: 65536
write size:          2; bytes successfully before error: 65536
write size:          4; bytes successfully before error: 65536
write size:          8; bytes successfully before error: 65536
write size:         16; bytes successfully before error: 65536
write size:         32; bytes successfully before error: 65536
write size:         64; bytes successfully before error: 65536
write size:        128; bytes successfully before error: 65536
write size:        256; bytes successfully before error: 65536
write size:        512; bytes successfully before error: 65536
write size:       1024; bytes successfully before error: 65536
write size:       2048; bytes successfully before error: 65536
write size:       4096; bytes successfully before error: 65536
write size:       8192; bytes successfully before error: 65536
write size:      16384; bytes successfully before error: 65536
write size:      32768; bytes successfully before error: 65536
write size:      65536; bytes successfully before error: 65536
write size:     131072; bytes successfully before error: 0
write size:     262144; bytes successfully before error: 0
```

It seems that the pipe capacity, at least for stdout is 65536 bytes.


The following is a short Node program you can try running that shows how filling a pipe to maximum capacity blocks your process from finishing. You can uncomment the `child.stdout.on` block to get the process to exit properly. All this program does is spawn a subprocess that echoes a really long string (100,000 'a's) to stdout.

Note that in some systems, according to the [man page](http://man7.org/linux/man-pages/man7/pipe.7.html), the write will fail, rather than block.

```js
const spawn = require("child_process").spawn;

function longString() {
    let str = "";
    for (let i = 0; i < 100000; i++) {
        str += "a";
    }
    return str;
}

async function echo(callback) {
    try {
        let child = spawn("echo",
            [
                longString()
            ]
        );
        child.on('exit', (code) => {
            console.log(`Exit code: ${code}`);
            callback(null);
        });
        child.on('error', (err) => {
            throw err;
        });
        // child.stdout.on('data', (data) => {
        //     console.log(data.toString());
        // });
    } catch (err) {
        callback(err);
    }
}

echo(function (err) {
    console.log("Done echoing");
    if (err !== null) {
        console.log(err);
    }
});
```

This bug was really tricky to figure out. I hope you learned something new from this, and if you did, share this with someone you know! Maybe you'll save them a few hours of debugging 😊.














