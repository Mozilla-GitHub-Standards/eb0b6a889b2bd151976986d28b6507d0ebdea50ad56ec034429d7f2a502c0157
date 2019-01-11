# Mozilla wpt-api Documentation
## (a.k.a Using WebPageTest...Mozilla Edition)

## Introduction
## Project Goals:
* Accurately measure, track, and report on key Firefox-performance metrics, over time
* Leverage, and be an integral part of, the [WebPageTest](https://www.webpagetest.org/) ecosystem

## Metrics & Metadata:
Current Metrics:
* domContentFlushed
* timeToFirstByte
* timeToFirstNonBlankPaint
* timeToFirstMeaningfulPaint
* timeToConsistentlyInteractive
* visualComplete
* pageLoadTime
* requestsFull
* bytesInDoc

Test Metadata:
* browser_version
* browser_name

### Manually Testing Metrics in Firefox
1. Enable/modify any (missing) prefs/pref-overrides
2. Open Tools -> Web Developer -> Web Console
3. Load a URL
4. After the page has "finished" loading, type something like: ```window.performance.timing.timeToFirstInteractive``` (**DO** take note of the other implemented/available metric options available as you autocomplete.)
5. Hit return/enter
6. You should see a value similar to ```1542438605479``` (time-stamped offset, in milliseconds)

PRO-TIP: you can and *should* input ```window.performance.timing``` into the Console, to dump the entire PerformanceTiming object

**[XXX FIX ME - Upload a tantalizing screengrab, here]**

### Adding Metrics to WebPageTest with Firefox
1. Manually test the metric + pref (ahem); most metrics can be found in the following Firefox DOM WebIDL:  [```mozilla-central/dom/webidl/PerformanceTiming.webidl```](https://hg.mozilla.org/mozilla-central/file/tip/dom/webidl/PerformanceTiming.webidl)
2. If needed, add/modify Firefox's `prefs.js`, via a PR to  [```wptagent/internal/support/Firefox/profile/prefs.js```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/Firefox/profile/prefs.js)
  Example: https://github.com/WPO-Foundation/wptagent/pull/181/files#diff-69b0882d86377063fd0514c0dc978308
3. Additionally, we might need to have the metric (if not available via standard APIs) emitted in WebPageTest, in [```wptagent/internal/js/page_data.js```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/js/page_data.js#L27).
Example: https://github.com/WPO-Foundation/wptagent/pull/230

## WebPageTest Runs at Mozilla

This should be a mid-to-high level overview of pertinent Mozilla-specific goals, design considerations (challenges) and, as a brief overview, a visualization of our WPT ecosystem.

## Firefox-Pertinent *WPT* Code:
###  wptagent
* [```Dockerfile```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/Dockerfile)
* [```desktop_browser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/desktop_browser.py) (base class for **_all_** browsers)
* [```browsers.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/browsers.py) (self-described "Main entry point [sic] for controlling browsers")
* [```webpagetest.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/webpagetest.py) (the one, the only)
* [```firefox.py```](https://github.com/WPO-Foundation/wptagent/blob/master/internal/firefox.py) (hey, that's us!)
* [```traffic-shaping.py```](https://github.com/WPO-Foundation/wptagent/blob/master/internal/traffic_shaping.py) (cross-platform rate-limiting, latency, etc. support)
* [```page_data.js```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/js/page_data.js) (user timings, vendor-specific/internal metrics & configs/settings)
* [```firefox_log_parser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/firefox_log_parser.py) (where the metric-culling magic happens)
* [```visual_metrics.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/visualmetrics.py) (the guts of the visual-comparison algorithms)
* [```trace_parser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/trace_parser.py) (calculates metrics with recipes/formulas)
* [```pcap_parser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/pcap-parser.py) (wait for it...parses pcap files!)

# Infrastructure
### WebPageTest Instance (*Mozilla-internal*)
* https://wpt.stage.mozaws.net - Auth0/OAuth
* https://wpt-api.stage.mozaws.net - Basic Auth
   * You'll need to file a bug in Cloud Ops, requesting access

### Jenkins
* [```https://qa-preprod-master.fxtest.jenkins.stage.mozaws.net/job/wpt/```](https://qa-preprod-master.fxtest.jenkins.stage.mozaws.net/job/wpt/)
   * To obtain access, see these (internal-only) [Mana docs](https://mana.mozilla.org/wiki/display/TestEngineering/qa-preprod-master.fxtest.jenkins.stage.mozaws.net)

### AWS EC2 Setup
* **1** ```c5.xlarge``` __WebPageTest__ core server-instance (Linux, EC2 AMI ID: ami-024df0ababa7118ae)
* **6** x ```c5.xlarge``` __wptagent__ server instances (Linux, EC2 AMI ID: ami-a88c20d5)

## Dependencies
#### Core:
* [WebPageTest](https://github.com/WPO-Foundation/webpagetest/blob/master/README.md) (core/server)
* [wptagent](https://github.com/WPO-Foundation/wptagent/blob/master/README.md) (configs/runs browser tests, and collects and submits metrics/data to be post-processed by the above core WebPageTest server; the project formerly known as "wptdriver."
* [webpagetest-api](https://github.com/marcelduran/webpagetest-api/blob/master/README.md) (NodeJS-wrapped core API)

Mobile (Android) Testing
* TBD; scaffolding up soon, here


## W3C (Draft) Specs

* [Perf-Timing Primer](https://w3c.github.io/perf-timing-primer/)
* [Navigation Timing](https://www.w3.org/TR/navigation-timing/)
* [Navigation Timing Level 2](https://www.w3.org/TR/navigation-timing-2/) (Working Draft)
* [High-Resolution Time Level 2](https://www.w3.org/TR/hr-time-2/)
* [Performance Timeline Level 2](https://www.w3.org/TR/performance-timeline-2/)
* [User Timing](https://www.w3.org/TR/user-timing/)

## Assorted Links:
* Example Mobile-Device Lab Config: https://github.com/WPO-Foundation/webpagetest-docs/blob/2db35b31fe1c992c02650a5b401f7ed208d8fa27/user/Private%20Instances/mobile/example.md
* https://www.w3.org/2000/09/dbwg/details?group=45211&order=org&public=1
* Web-platform tests we run: https://github.com/web-platform-tests/wpt/tree/744325921ba52791bc8db1b45d2aed097577753a/navigation-timing
***

## Contributions
***psst, this could be you!***

## Credits & Thanks
* Pat Meenan - too numerous to call out; thanks for the project, the community, the tooling, and your direct support
* Rick Viscomi, Andy Davies, & Marcel Duran - thanks for /the/ published book, and the numerous blog posts, Tweets, talks, forum posts, etc., to further its adoption/ecosystem
* Peter Hedenskog - for everything (he knows!)
* [Sitespeed.io crew](https://www.sitespeed.io/) - for helping me re-envision and prototype an idea from 2010!
* Wikimedia Performance Team for their [amazing + open WebPageTest setup](https://wikitech.wikimedia.org/wiki/Performance)
* Firefox Performance Team for their support & shared focus