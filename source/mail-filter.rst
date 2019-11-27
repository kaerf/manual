.. _mailfilters:

###############
Filtering mails
###############

We filter incoming mails with `Rspamd <https://rspamd.com>`_ which uses `multiple <https://rspamd.com/comparison.html>`_ filtering and statistical methods to generate a spam score, including (but not limited to) SPF, DMARC and DNS blacklists. Mails with a score greater than 15 get rejected. 

Configure spam filter
=====================

Use ``uberspace mail spamfilter`` to configure the filter for your account:

.. code-block:: console

  [eliza@dolittle ~]$ uberspace mail spamfilter status
  Mail filter enabled.
  [eliza@dolittle ~]$ uberspace mail spamfilter disable
  Mail filter disabled.
  [eliza@dolittle ~]$ uberspace mail spamfilter enable
  Mail filter enabled.

Header
======

We add the score to the mail header: ``X-Rspamd-Bar``, ``X-Rspamd-Report`` and ``X-Rspamd-Score``.

Maildrop as mail filter
=======================

Create your maildrop configuration file
---------------------------------------

Create a new directory to contain your maildrop config file. The config file could also be directly in your home folder, but you might want to create several configuration files for several accounts, and keep log files in the same folder.

.. code-block:: console

  [eliza@dolittle ~]$ mkdir .mailfilter
  [eliza@dolittle ~]$ touch .mailfilter/mailfilter-default

Edit your configuration file
----------------------------

Full `documentation on maildrop <https://www.courier-mta.org/maildropfilter.html>` can be found on the website of Courier MTA.

For starters, your config file can look something like this (scroll down for the complete example):

.. code-block:: bash

  # set logging to a log file in the .mailfilter directory
  logfile "$HOME/.mailfilter/mailfilter-default.log"
  # default mail directory, assuming you use maildir.
  MAILDIR="$HOME/Maildir"

  # ...

  # last line in your config file is the to command that ultimately drops mail into your Inbox
  to "$MAILDIR"
  
Junk mail filtering
-------------------

After having enabled the uberspace rspamd spamfilter, you will want to make use of the headers it sets.

.. code-block:: bash
  # set a variable to contain your maximum spam score to 5
  MAXSPAMSCORE=5.0
  # default name for your Junk folder should be "Junk", right under your Inbox.
  FOLDER_Junk="$MAILDIR/.Junk"

  # ...

  # test the currently treated email for a X-Rspamd-Score header using a regular expression
  if ( /^X-Rspamd-Score: ([-+]?[0-9]*\.?[0-9]+)/ )
    to "$FOLDER_Junk"

You may want to try some different numbers for your maximum spam score. For some, setting it to 3.5 might work. Try some different numbers until you find the rate of false positives to false negatives suits you best. But be sure to put the number as a floating-point number (so that's "3.0" rather than "3").

.. code-block:: bash
  MAXSPAMSCORE=3.5

Filtering by Subjects
---------------------

Similarly, a regular expression can be used to search in the email subject or other headers.

.. code-block:: bash
  
  # move subjects "final notice" or "you have more friends on facebook than you think" to Junk
  # to be certain also subjects with one or more spaces in the beginning are caught, use "\s+"
  if ( /^Subject:\s+(final notice|you have more friends on facebook than you think)/ )
    to "$FOLDER_Junk"

Create mail folders automatically
---------------------------------

Maildrop can also be used very generically, so you might want to check that the Junk folder already exists before you start dropping mails into.

.. code-block:: bash

  # maildrop can even execute shell commands like in bash scripts, using the apostrope:
  # check if the folder exists, or create the Junk folder using the maildirmake command
  `test -d "$FOLDER_Junk" || /usr/bin/maildirmake "$FOLDER_JUNK"`

Full example config
===================

.. code-block:: bash
  
  # set logging to a log file in the .mailfilter directory
  logfile "$HOME/.mailfilter/mailfilter-default.log"
  # default mail directory, assuming you use maildir.
  MAILDIR="$HOME/Maildir"

  # set a variable to contain your maximum spam score to 5
  MAXSPAMSCORE=5.0
  # default name for your Junk folder should be "Junk", right under your Inbox.
  FOLDER_Junk="$MAILDIR/.Junk"

  # maildrop can even execute shell commands like in bash scripts, using the apostrope:
  # check if the folder exists, or create the Junk folder using the maildirmake command
  `test -d "$FOLDER_Junk" || /usr/bin/maildirmake "$FOLDER_JUNK"`

  # test the currently treated email for a X-Rspamd-Score header using a regular expression
  if ( /^X-Rspamd-Score:\s+([-+]?[0-9]*\.?[0-9]+)/ )
    to "$FOLDER_Junk"
  # move subjects "final notice" or "you have more friends on facebook than you think" to Junk
  # to be certain also subjects with one or more spaces in the beginning are caught, use "\s+"
  if ( /^Subject:\s+(final notice|you have more friends on facebook than you think)/ )
    to "$FOLDER_Junk"

  # last line in your config file is the to command that ultimately drops mail into your Inbox
  to "$MAILDIR"
  

Activate your mailfilter
========================

Your ``.qmail-default`` file directs all incoming email to maildrop and gives `~/.mailfilter/mailfilter-default` as your configuration file

.. code-block:: console

  |/usr/bin/maildrop ~/.mailfilter/mailfilter

