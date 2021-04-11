# trouble-shooting

If any of these issues appear to be happening in the `cloud` environment, contact BinaryMist with details.

# SUT requests taking too long

## Description

When running purpleteam, the time of the web requests to your SUT seem to be taking longer than they should be.

## Cause

This can be due to scripts in your SUT that are taking a long time to load or not loading at all. We saw this initially with NodeGoat in regards to SUT pages attempting to fetch the livereload script. NodeGoat was expecting the livereload script to be hosted locally which it wasn't, subsequently the page load wouldn't finish loading.

## Solution

Check that Zap doesn't have any "Timed out while reading a new HTTP request" messages. If it does:

1. Debug the relevant app-scanner cucumber step where the Selenium webdriver instance makes it's requests to your SUT
2. Then VNC into the selenium container, open the browser tools and check that all resources are in-fact loading in a timely manner

We fixed this by removing the dependency on this script (livereload.js) in the NodeGoat `production` environment.


# URL not found in the Scan Tree

## Description

App test failing (specifically the Zap active scan) with the following error message displayed in the CLI app log:  
`URL Not Found in the Scan Tree`.  
This error is also visible in the app-scanner log and originates from Zap. Zap also logs the following message as a `WARN` event:  
`Bad request to API endpoint [/JSON/ascan/action/scan/]`  
`URL Not Found in the Scan Tree`

## Cause

This can be due to one or more missing `attackFields` in the Build User config (Job) for a given `route` that you have specified. These `attackFields` are not only used by Selenium to proxy the specific `route`'s request through Zap, but also used to inform Zap of the `postData` when a request is made to [ascanActionScan](https://www.zaproxy.org/docs/api/?java#ascanactionscan).

## Solution

Check that your Build User config (Job) contains all of the `attackFields` that your SUT requires to make a successful request.

# "Terminfo parse error" in Terminal

## Descriptioin

Running the purpleteam CLI may produce a `Warning` message: `Terminfo parse error`.

## Cause

This is due to the `TERM` environment variable being incompatible for the CLI dependency blessed.

## Solution

Try setting the `TERM` environment variable to something other than your system default before running the purpleteam CLI. Blessed-contrib provided [some details](https://github.com/yaronn/blessed-contrib/#troubleshooting).

We have had good results with `TERM=xterm`.

