---
title: "Gnus and Protonmail Bridge Incompatibility"
date: "2024-02-16T09:21:00+02:00"
tags:
  - Emacs
---
If you're [ProtonMail Bridge](https://github.com/ProtonMail/proton-bridge) and
[Gnus](https://gnus.org) user, you've probably seen that Gnus is acting very
strange. That's because Gluon (the embedded IMAP server inside ProtonMail
Bridge) is responding to `FETCH` requests in random order. Example
communication with Gluon might look like this:

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

Apparently, this is not against **RFC 3501** protocol but Gnus doesn't like it
anyway. The most straightforward way to fix this incompatibility is to patch
Gluon and disable parallelism. There's very handy switch
(`isParallelismDisabledCtx`) inside the `internal/state/mailbox_fetch.go`
file:

```
if !contexts.IsParallelismDisabledCtx(ctx) && (len(snapMessages) > minCountForParallelism || (len(snapMessages) > 1 && needsLiteral)) {
	// If multiple fetch request are happening in parallel, reduce the number of goroutines in proportion to that
	// to avoid overloading the user's machine.
	parallelism = runtime.NumCPU() / int(activeFetchRequests)

	// make sure that if division hits 0, we run single threaded rather than use MAXGOPROCS
	if parallelism < 1 {
		parallelism = 1
	}
} else {
	parallelism = 1
}
```

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

I hope that this micro change will save you trouble.
