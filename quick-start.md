# quick start

There are two options with getting your System(s) under Test:

* purpleteam `cloud`. All back-end services are set-up and ready to go
* purpleteam `local`. You set-up everything yourself

Both `cloud` and `local` use the same code base.

## purpleteam `cloud` (BinaryMist purpleteam)

The quickest way to get up and running with having your Web application and/or API under test is to take the purpleteam `cloud` path. At a high level, these steps look like the following:

1. Obtain a purpleteam-labs `cloud` account. We set-up and manage everything in the cloud for you. If you decide to take the `local` path instead, this will be your responsibility
2. Get the [purpleteam CLI](https://github.com/purpleteam-labs/purpleteam) on your system and [configure it](https://github.com/purpleteam-labs/purpleteam#configure)
3. Create a _Build User_ config (_Job_). There are some examples [here](https://github.com/purpleteam-labs/purpleteam/tree/main/testResources/jobs)
4. [Start testing](https://github.com/purpleteam-labs/purpleteam#run)

Purpleteam CLI can be run manually, driven from your CI, or other builds, to continuously inform you of security regressions in the Web applications that you are developing. This way you can easily find and fix your defects as they are being introduced.

Follow all directions under [`cloud`](cloud.md).

## purpleteam `local` (OWASP purpleteam)

If you choose to go the purpleteam `local` path, a non-trivial set-up is required to get up and running.

Follow all directions under [`local`](local.md).
