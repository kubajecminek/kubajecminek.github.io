---
title: "Gnus & ProtonMail Bridge Incompatibility"
date: 2024-02-16T22:02:16+01:00
tags: ['Emacs']
---
This story is, as the title suggests, about Gnus (the Emacs newsreader,
remember?) and [ProtonMail Bridge](https://proton.me/mail/bridge). Gnus can
communicate over IMAP but is very slow in doing so. That's why people recommend
fetching the mail locally and feeding it into the Dovecot server, which runs on
the same machine as Gnus. This setup eliminates network round trips, and Gnus
suddenly becomes very fast. I tried that for a while, but it didn't feel as
comfortable. When I decided to switch my mail provider to ProtonMail, I was
really looking forward to using Bridge because it acts as a local IMAP server,
and I would no longer need Dovecot and fetchmail. When I fired up Gnus for the
first time, I quickly realized that it wasn't going to be that easy. Gnus was
acting crazy with the Bridge, and I almost gave up. I searched the entire
internet, but it seemed like no one else had the same issue. I reported it to
ProtonMail, but they told me they don't support Gnus (what a surprise). I
didn't report this issue to `bug-gnu-emacs` because I was sure nobody would
pick it up. I was so pissed off that I decided to fix it myself.

## Bug or not a bug

At this point, I wasn't even sure if it was a problem with Gnus or with the
Bridge. Gnus works fine with all the major IMAP servers, and the Bridge works
fine with all major clients. I started looking for clues in the Gnus source
code, but it's a huge package, and I didn't find anything useful. So I compiled
Gluon - the IMAP server embedded in the Bridge - and installed Dovecot once
again. Next, I created test mailboxes on both servers and started debugging the
responses. Nothing seemed out of the ordinary until...

Gluon started to respond to FETCH requests with a random message order, but
Dovecot didn't. An example communication with Gluon looked something like this:

```
Gnus: FETCH 1:3 ...
Gluon: * 3 RESPONSE ...
Gluon: * 1 RESPONSE ...
Gluon: * 2 RESPONSE ...
```

or this

```
Gnus: FETCH 1:3 ...
Gluon: * 2 RESPONSE ...
Gluon: * 3 RESPONSE ...
Gluon: * 1 RESPONSE ...
```

and it was completely random. I used edebug inside Emacs and confirmed that
this was indeed the culprit of the issue. I looked into RFC 3501 to see if this
is against the protocol, but apparently it is not (also see this [StackOverflow
discussion](https://stackoverflow.com/questions/31478228/may-i-shuffle-the-order-of-the-e-mails-in-a-response-for-a-fetch-command)).
I hoped I could just send this to Proton and let them fix their server, but
unfortunately, they didn't do anything wrong.

## Toward solving the issue

I was left with two options: (i) patch Gnus, (ii) patch Gluon. I considered
both options and even started working on the client, but I quickly realized
that it would be too much work, so I went back to Gluon. After skimming through
its source code, I had the suspicion that Gluon's weird responses had something
to do with concurrency (Gluon is written in Go - a language known for its
concurrent features).

After a while I found what I was looking for (this snippet is taken from
`internal/state/mailbox_fetch.go`).

```
if !contexts.IsParallelismDisabledCtx(ctx) && (len(snapMessages) > minCountForParallelism || (len(snapMessages) > 1 && needsLiteral)) {
   ...
   ...
}
```

So there's **isParallelismDisabledCtx** you say? :thinking: As it turned out, I
could disable the whole parallelism thingy with a simple server option.

## Patch
This is the final patch for the `proton-bridge` package.

```
From 23f9c69a1552af1f946687f55de901488a2c9a38 Mon Sep 17 00:00:00 2001
From: =?UTF-8?q?Jakub=20Je=C4=8Dm=C3=ADnek?= <kuba@kubajecminek.cz>
Date: Fri, 16 Feb 2024 16:07:21 +0100
Subject: [PATCH] Gnus Fix: disable parallelism

---
 internal/services/imapsmtpserver/imap.go | 1 +
 1 file changed, 1 insertion(+)

diff --git a/internal/services/imapsmtpserver/imap.go b/internal/services/imapsmtpserver/imap.go
index 63888b51..358173c6 100644
--- a/internal/services/imapsmtpserver/imap.go
+++ b/internal/services/imapsmtpserver/imap.go
@@ -120,6 +120,7 @@ func newIMAPServer(
 		gluon.WithReporter(reporter),
 		gluon.WithUIDValidityGenerator(uidValidityGenerator),
 		gluon.WithPanicHandler(panicHandler),
+		gluon.WithDisableParallelism(),
 	)
 	if err != nil {
 		return nil, err
-- 
2.42.0
```

This single line of code solved all my issues. I hope this will be useful to
other people.
