
# Update 2017-01-02

So I'm new to python but I needed something and this met the requirements mostly.
I've made some modifications to this so that it now stores the feed list in seen.db
along with the associated emails they should go to.

So bob.com/feed can go to bob@gmail.com and sam.com/feed can go to bob@email.com,sam@email.com

As per the above I'm new to python and as such this probably isn't the best code ever, but it was a learning experience.
#ToDo
Create some sort of argument that allows you to manage feeds and emails for those feeds
tidy up the documentation
probably tidyup the code a bit more once I understand this whole tuple thing.

##########

grab_rss is a simple RSS to email gateway; it downloads feeds (RSS,
Atom, or anything else Universal Feed Parser understands), formats
them into plaintext emails, and sends them to you. The idea would be
to run it once an hour or once a day or whatever in a cron job.

* Why write this?

I had been using Google Reader. Mutt is a better information organizer
than Reader, and since I run my own mail server there are fewer
privacy concerns (I expect at some point Google will begin analyzing
what people look at in their Readers, what gets the most clicks, how
long you spend looking at particular items, etc so as to enhance the
advertising experience - assuming they don't do this already of
course). I initially was going to replace it with a standalone GUI RSS
reader, but the ones I evaluated for Linux were buggy, incomplete,
very slow or a combination of the three. So I ended up writing this
brand new buggy, incomplete and very slow program instead. Your own
ugly babies are always the cutest ones.

* Why not just use rss2email?

Using pickle for the state is obnoxious - hard to read, edit, or
save/merge using version control. I strongly prefer plain text for
everything that's even remotely important, because it lets me use
existing high quality diff/merge tools built into DVCSes to save my
data, and quality editors to modify them as necessary.

In short: die, binary data, die in a fire.

(And then I realized it was far better to save the seen items state in
a sqlite db than a plain text file in case an exception is thrown and
we revert state. Hypocrisy rocks!)

* What's there?

All the basic functionality works: it reads RSS feeds, sends you
emails about them, remembers which ones it has already told you about.
I'm using it as my only RSS source, and I'm happy with it.

* What's missing?

No provision for HTML mail. Everything is converted to text/plain,
stripping out everything except links. I don't like HTML mail.

If you want to read anything that's not written in mostly ASCII you're
out of luck; all input text is forced to 7-bit ASCII.

Probably other things that haven't even occured to me. Suggestions
welcome, patches better.

* What do I need?

Currently you need Python 2.6 (2.4/2.5 might work, haven't tried) plus
the following dependencies:

Required:
  feedparser      - http://www.feedparser.org
  stripogram      - http://www.zope.org/Members/chrisw/StripOGram

Optional:
  dateutil.parser - http://labix.org/python-dateutil
  multiprocessing - http://code.google.com/p/python-multiprocessing/

The multiprocessing module is included in Python 2.6

Patches to reduce dependencies while improving or maintaining
functionality happily accepted.

All the dependencies are included in Gentoo:

  emerge dev-python/python-dateutil \
         dev-python/feedparser \
         dev-python/stripogram

and in Debian:

  apt-get install zope-stripogram \
                  python-dateutil \
                  python-feedparser

and probably in most other reasonably sane Linux distros.

* How do I use it?

grab_rss uses 3 files, which are located in either $GRAB_RSS_DIR or ~/.grab_rss:

  - grab_rss.conf: A configuration file (see more about that below)

  - feeds.txt: A list of feed locations, one per line. Like so:

"""
http://globalguerrillas.typepad.com/globalguerrillas/atom.xml
http://randombit.net/bitbashing/index.atom
http://taint.org/feed
"""

  - seen.db: A sqlite database listing already seen/sent posts
    (you shouldn't need to ever look at this)

* What do I put in grab_rss.conf?

The only required item is what email address you want the output sent to:

"""
[GrabRSS]
to = user@example.com
"""

Using the full set of options:

"""
[GrabRSS]
to = user@example.com
from = grab-rss@example.com
smtp_host = mail.example.net
socket_timeout = 10
pool_size = 4
user_agent = Lynx/2.8.7
"""

The pool_size specifies how many processes to use for downloading
feeds (using the multiprocessing module, which means you have to have
Python 2.6 or have installed it specially for this to work).

Running several downloads in parallel can substantially speed up how
fast grab_rss runs (for my 60-something feeds, from just under a
minute with 1 process to under 10 seconds with 8 procs; further
increases in pool size didn't decrease runtimes). The optimal size
will depend a lot on your local hardware and network as well as how
many feeds you are trying to get (and all of their hardware and
networks). The default pool size is 0, which means don't use the
multiprocessing module at all. Play with the --pool-size option if you
want to experiment.

The default socket timeout is 30 seconds, which is probably fine
unless you have a very wonky network or are running only a single
process (in which case a single down server can hang your entire run
by the full socket timeout - with multiple processes useful work will
still be happening while one process waits around for the timeout).
Be warned this timeout applies to both pulling down the feeds and to
the SMTP timeout, though I haven't encountered any problems with that.

The SMTP host defaults to localhost which is probably the right thing
to do at least a third of the time.

No provisions for SMTP authentication currently, might be useful but I
don't need it so I haven't written it.

* How do I filter this?

You can filter by the sender (set in grab_rss.conf) and/or the
existence of the header X-GrabRSS-Feed. The value of X-GrabRSS-Feed is
set to the URL of the feed that this post was from, if you want to
filter posts to different mboxes based on source.

In procmail speak:

:0:
* ^X-GrabRSS-Feed:
RSSFeeds

* Can I reuse this?

Sure. License is stock GPLv2. If you need it under some other license
for some reason, contact me.
