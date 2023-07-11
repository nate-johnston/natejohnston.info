+++
title= "How I Do Email"
date= 2021-06-02T17:14:18-04:00
description = "How I Do Email"
categories= ["How I Do Things"]
+++

This post is a true account of my journey from a standard, vanilla GMail setup
towards a wholly different method of organizing my email that I quite enjoy, and
that so far serves me well.  I shall start by answering the first question
anyone would ask: since GMail is ubiquitous, why did I leave it?

<!--more-->

### Why I Switched from GMail

First off, I wanted to dead-head my email archive.  I had hundreds of thousands
of emails, mostly unread.  I have been using filters and labels for a long time,
sectioning off my email in the thought that maybe I would need it someday.  But
the truth is, I never needed it.  I had to confront that truth, that I was an
email hoarder, in order to rethink my approach.

Second, I use Google Apps to maintain some mail aliases that go to groups of
people, so that I can give a single email address that will go to both myself
and my wife, or to the two of us plus my daughter.  This has been fantastically
useful to make sure that we are both in the know about all of our utilities and
accounts, the happenings in our daughter's world, and many other arenas.  This
was all possible because I had a domain registered with the [free tier of G
Suite](https://support.google.com/a/answer/2855120?hl=en#zippy=), but I started
to experience issues that indicated the free tier was not being maintained to
the same level as the ad-supported public version - rejection notices, spam
blacklisting, and emails being dropped.  After some digging around and a few
private conversations, I was convinced this was due to the progressive decline
in support for the Google Apps free tier.  This kind of atrophy is natural when
a paid tier comes along and the free tier is discontinued.  I wanted a place
where I would be supported, and I was willing to pay a tiny amount for the
privilege.  

Third, I am tired of being the product.  I am tired of mentioning "Hawaiian
shirts" in front of a Google Home and the next day seeing advertisements for
Hawaiian shirts everywhere.  I'm not one of those nuts who rails against the
coming Panopticon-fueled autocracy as all these technologies are used to oppress
us.  I just want there to be fewer advertising-related glitches in the matrix.
There's not much I can do about that - I saw the power of demographic data and
rostering when I worked for America Online in the Popup project.  But what I can
do is to take what small steps I can to self-select out of the Panopticon.

### Design Considerations

Here are the things that I required from my future email setup.  First the
easy/uncontroversial:

- Reliable service, no blacklists
- Support for folders or tags (folders preferred)
- Support for Bring-Your-Own-Domain MX handling
- Support for mailing lists in my domain(s)
- Handle impersonating my GMail account

The more difficult:

- Effortless inbox zero
- Zero-value crap email will not affect storage quotas
- I never look at subject-specific mail folders, so filter email by the level of attention I need to pay to it instead of a subject matter categorization.

Bonus points if email is hosted in the UK or EU.

Finally one tactical decision I made was that I decided not to migrate my
existing inbox contents to the new setup.  I would "dead-head" my old email
account but leave the contents in there so that if I was ever possessed of the
need to check out that delivery pizza advertisement from 2007 I could easily do
so.  So begins the fresh start.

### Migration to Pobox

The email provider [Pobox](http://www.pobox.com) has a long record of extremely
high reliability and good customer service.  A number of people I respect online
had mentioned their own positive experiences with it, and it was [VM Brasseur's
endorsement](https://twitter.com/vmbrasseur/status/629119632989798401) of their
AllMail feature that pushed me over the edge in favor of Pobox.  I started with
the free trial, changed my MX records, and started to enjoy the new mailbox.  It
helped that Pobox, which was acquired by Fastmail, works beautifully with the
Fastmail IOS app.  

### Email Organization

I started off with five, numbered folders, and then quickly added a sixth
unnumbered one.  All mail would be filtered into these folders:

- "1 - Critical": Mail from my wife, my daughter, family members and friends.
- "2 - Interesting Now": Bills to be paid, my homeowners association, and online orders created or packages enroute.
- "3 - Routine": Things I expect to get in a day, like notices from my home security system as it detects events, notices from my daughter's school, updates from important services like HelloFresh, and the one or two newsletters I honestly do make time to read every morning.
- "4 - Interesting Later": The place for "here's the neat stuff that happened in the last 24 hours" content from various social media networks, newsletters I may or may not read, LEGO advertisements... things I might actually want to go back and find (although not forever).
- "5 - Never Interesting": The default delivery point for mail that doesn't meet the other filters.

From here on, I will refer to these just by number, as the "level 1" mailbox
through to "level 5" mailbox.

Pobox has a feature that is very interesting: for specific folders you can
specify a time to live for emails, so any messages sent before a certain date
are automatically expunged.  The level 5 mailbox auto-deletes email that is 90
days old, and the level 4 mailbox deletes mail that is one year old.  This means
that most of the email I get is destined for deletion without any effort on my
part.

{{< figure src="../../assets/images/pobox-4.png" title="Example of auto-deletion policy on mailbox 4" >}}

The sixth mailbox I added was my place to keep mail that was new and that the
filtering system needed to be taught.  Adding a new address is a manual process
at this point, because nowadays it happens rarely.

- "sort me please"

This way when I am on the go and I see something that bucks the pattern, I can
file it there and add a rule later.

### Email Sorting

One of the downsides of Pobox is that you can have a relatively small number of
filter rules, and there is no way to increase that number.  This puts the kibosh
on any kind of builtin sophisticated email filtering engine such as what many
power users may be used to with GMail.  I know I certainly ended up with an
almost unmanageable number of filters in GMail, more than 200.

{{< figure src="../../assets/images/pobox-5.png" title="Why bother reading it?  It will be gone in 3 months." >}}

To the rescue: [imapfilter](https://github.com/lefcha/imapfilter).  This
excellent project runs from my personal computer every five minutes on a timer.
Each time it runs, it reads a configuration file written in the Lua programming
language that defines a series of matching expressions.  Mail that matches an
expression is then moved into a specific mailbox.  For example, in the below
snippet, mail from the various Virginia state agencies that communicate about
COVID-19 vaccinations and other news is filtered to the level 1 mailbox.

```
box1 = '1 - CRITICAL'
results =
    account1.INBOX:contain_from('vams.cdc.gov') +
    account1.INBOX:contain_from('vdh.virginia.gov') +
    account1.INBOX:contain_from('vdh@public.govdelivery.com')
results:move_messages(account1[box1])
```

After everything I want to process has been processed, I push the rest to the
level 5 mailbox:

```
box5 = '5 - Never Interesting'
results = account1.INBOX:select_all()
results:move_messages(account1[box5])
```

With this setup I see notifications on my phone as email comes in and is
delivered to my inbox.  But quickly those notifications disappear, as imapfilter
filters all that mail to exactly where I want without me having to lift a
finger.  As a bonus, the filters can be as complex as I want.

### Advanced Topic: Automated imapfilter config

OK, so I have this neato method for filtering my mail.  But it's pretty much
just a database of rules mapped to mailbox numbers, right?  There has to be an
easier way to manage this.

Well, at least if you define "easy" the way I do.

I already have a local [Consul](https://www.consul.io/) cluster that I mostly
use as a distributed key-value store for configuration.  I use
[consul-template](https://learn.hashicorp.com/tutorials/consul/consul-template)
to manage my configurations.  I find this an easier way than using something
like Puppet or Ansible where I have to manage a directed acyclic graph of
actions.  So I figured I would leverage this.

You might ask: why not use etcd for this?  Well, I think of this data
hierarchically, and between the non-hierarchical structure of data in etcd v3 as
well as the general incompatibility of etcd v2 and v3, I decided to go with
something that went with the grain of my conceptual model more readily.

I populated my consul key value store with the email addresses that I wanted to
be filled.  I have a test and a production part of the hierarchy, but here is
how I push entries for the Virginia alerts I mentioned before:

```
% consul kv put test/services/imapfilter/1/vams.cdc.gov contain_from
% consul kv put test/services/imapfilter/1/vdh.virginia.gov contain_from
% consul kv put test/services/imapfilter/1/vdh@public.govdelivery.com contain_from
```

You can see that I have an imapfilter folder, and under it are the numbered
mailboxes 1-4 (no need for 5 as it is the default).  Then under that is the
regex string as key, and the value is the conditional, like "contain_from" or
"contain_to".  

This is written to the configuration file by consul-template.  The template file
iterates through all the numbered folders and constructs the rules:

```
{{ range $key, $pairs := tree "prod/services/imapfilter/" | byKey }}
results = {{range $pair := $pairs }}
    account1.INBOX:{{ .Value }}('{{ .Key }}') + {{ end }}
    account1.INBOX:contain_from('nobody@cw.net')
results:move_messages(account1[box{{$key}}])
{{ end }}
results = account1.INBOX:select_all()
results:move_messages(account1[box5])
```

I add that last address, 'nobody@cw.net', as a token so I am not left with a
trailing plus sign.  Yes, I know I am a lazy boy.  But I know I will never get
any email from that address because I used to run Cable & Wireless USA's main
mailserver and I know that address is the equivalent of sending something to
/dev/null.  

I am sure I can adjust this to more complex conditions like multiple-criteria
rules when I find a need for it.

*UPDATE*: For a retrospective on how well this works on my personal email, check
out the [part 2](https://natejohnston.info/2022/pobox2/).
