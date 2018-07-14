---
layout: post
title: Impulse Donations
---

Two facts about my life:
- Cambridge is covered with street canvassers. [[^1]] I never give these people money, because roughly speaking, outsourcing investment decisions to charismatic strangers on the street seems unwise. I'm also not convinced this is a particularly ethical way to fundraise, but that's an issue for another day. But, post conversation, I always feel guilty: there's a lot of truth to the leading "when it comes down to it, couldn't you afford to give one dollar a day?" It'd be nice to be able to channel that guilt productively.
- It takes all of about 10 seconds and 3 button presses to impulse buy something on my phone, but I don't know of an equally easy way to make a quick charitable donation. The latter is more pro-social than the former, so ideally it's at least as easy.

I don't know much about web stuff but this seemed fixable. Here's what I came up with. I have a Twilio number, and whenever I want to make a donation, I text that number `<name of charity> <dollar amount>`. For example to donate \\$5 to GiveWell and then \\$5 to Cool Earth:

![Donation text]({{ site.baseurl }}/images/donation_text.jpg "Donation text")

Once a donation text is received, a HTTP request goes out to the [PandaPay](https://www.pandapay.io) API, and a donation is immediately made.[[^2]] Pretty quickly I get a confirmation email:

![Donation result]({{ site.baseurl }}/images/donation_result.png "Donation result")

Under the hood this is all running on Twilio functions. Some basic parsing is done:

![Donation parsing]({{ site.baseurl }}/images/donation_parsing.jpg "Donation parsing")

Setting the system up took me a shockingly long time, but at the end of the day it's quite simple. Nonetheless, it's the first thing I've built on the web that works, so I am immensely pleased.

And, thus far, it doesn't just work technically: it also works psychologically. There's an outlet for the guilt, and it's been kind of fun to make little impulse donations! Perhaps my only complaint are fees: there the Twilio phone number cost, per-text costs, PandaPay's fee, and credit card fees. In total these aren't a big dollar amount but are a sadly large percentage of the small donations I'm making. There's probably also some security risk, though as far as I can tell the only (and low probability) danger is someone using my money to donate to charities I've pre-selected...which is the whole point of this :) Overall, if you want to trick yourself into making more charitable donations, I highly suggest a setup like this.

Addendum: the current charities I have are GiveWell, the Against Malaria Foundation, Cool Earth, the ACLU, and the Greater Boston Food Bank. I'm open to more suggestions!


[^1]: In fact, [the last picture here](https://www.nature.org/membership-giving/donation/monthly-giving/fundraising-in-your-neighborhood/index.htm) shows a canvasser at the T stop I frequent.

[^2]: Well, earmarked for a donor advised fund, etc., but the money is gone from me and en route to the requested charity.
