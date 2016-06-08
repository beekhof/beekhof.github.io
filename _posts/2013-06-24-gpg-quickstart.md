---
layout: post
title: "GPG Quickstart"
date: 2013-06-24 14:40
comments: true
tags: 
- tips
---

It seemed timely that I should refresh both my GPG knowledge and my keys.
I am summarizing my method (and sources) below in the event that they may prove useful to others:

## Preparation

The following
[settings](https://we.riseup.net/riseuplabs+paow/openpgp-best-practices#update-your-gpg-defaults)
ensure that any keys you create in the future are strong ones by
2013's standards.  Paste the following into `~/.gnupg/gpg.conf`:

    # when multiple digests are supported by all recipients, choose the strongest one:
    personal-digest-preferences SHA512 SHA384 SHA256 SHA224
    # preferences chosen for new keys should prioritize stronger algorithms: 
    default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 BZIP2 ZLIB ZIP Uncompressed
    # when making an OpenPGP certification, use a stronger digest than the default SHA1:
    cert-digest-algo SHA512

The next batch of settings are optional but aim to improve the
output of `gpg` commands in various ways - particularly against
[spoofing](http://www.asheesh.org/note/debian/short-key-ids-are-bad-news).
Again, paste them into `~/.gnupg/gpg.conf`:

    # when outputting certificates, view user IDs distinctly from keys:
    fixed-list-mode
    # long keyids are more collision-resistant than short keyids (it's trivial to make a key with any desired short keyid)
    keyid-format 0xlong
    # If you use a graphical environment (and even if you don't) you should be using an agent:
    # (similar arguments as  https://www.debian-administration.org/users/dkg/weblog/64)
    use-agent
    # You should always know at a glance which User IDs gpg thinks are legitimately bound to the keys in your keyring:
    verify-options show-uid-validity
    list-options show-uid-validity
    # include an unambiguous indicator of which key made a signature:
    # (see http://thread.gmane.org/gmane.mail.notmuch.general/3721/focus=7234)
    sig-notation issuer-fpr@notations.openpgp.fifthhorseman.net=%g

## Create a New Key  

There are [several checks](https://we.riseup.net/riseuplabs+paow/openpgp-best-practices#openpgp-key-checks) 
for deciding if your old key(s) are any good.  However, if you created
a key more than a couple of years ago, then realistically you probably
need a new one.

I followed instructions from [Ana Guerrero's post](http://ekaia.org/blog/2009/05/10/creating-new-gpgkey/),
which were the basis of the current [debian guide](http://keyring.debian.org/creating-key.html),
but selected the 2013 default key type:

1. run `gpg --gen-key`
1. Select `(1) RSA and RSA (default)`
1. Select a keysize greater than 2048
1. Set a key expiration of 2-5 years. [[rationale]](https://we.riseup.net/riseuplabs+paow/openpgp-best-practices#set-an-expiration-date-if-you-do-not-have-one) 
1. Do *NOT* specify a comment for User ID. [[rationale]](https://www.debian-administration.org/users/dkg/weblog/97)

## Add Additional UIDs and Setting a Default

At this point my keyring `gpg --list-keys` looked like this:

    pub   4096R/0x726724204C644D83 2013-06-24
    uid                 [ultimate] Andrew Beekhof <andrew@beekhof.net>
    sub   4096R/0xC88100891A418A6B 2013-06-24 [expires: 2015-06-24]

Like most people, I have more than one email address and I will want
to use GPG with them too. So now is the time to add them to the key.
You'll want the `gpg --edit-key` command for this.  Ana has a good
exmaple of [adding UIDs](http://ekaia.org/blog/2009/05/10/creating-new-gpgkey)
and setting a preferred one.  Just search her instructions for `Add
other UID`.

## Separate Subkeys for Encryption and Signing

The [general consensus](http://stackoverflow.com/questions/5133246/what-is-the-purpose-of-using-separate-key-pairs-for-signing-and-encryption)
is that separate keys should be used for signing versus encryption.

> tl;dr - you want to be able to encrypt things without signing them
> as "signing" may have unintended legal implications.  There is also
> the possibility that signed messages can be used in an attack
> against encrypted data.

By default `gpg` will create a subkey for encryption, but I followed
[Debian's subkey guide](http://wiki.debian.org/subkeys) for creating
one for signing too (instead of using the private master key).

> Doing this allows you to make your `private master key` even safer
> by removing it from your day-to-day keychain.

The idea is to make a copy first and keep it in an even more secure
location, so that if a subkey (or the machine its on) gets
compromised, your master key remains safe and you are always in a
position to revoke subkeys and create new ones.

## Sign the New Key with the Old One

If you have an old key, you should sign the new one with it.  This
tells everyone who trusted the old key that the new one is legitimate
and can therefor also be trusted.

Here I went back to [Ana's instructions](http://ekaia.org/blog/2009/05/10/creating-new-gpgkey/).
Basically:

    gpg --default-key OLDKEY --sign-key NEWKEY

or, in my case:

    gpg --default-key 0xEC3584EFD449E59A --sign-key 0x726724204C644D83

## Send it to a Key Server

Tell the world so they can verfiy your signature and send you encrypted messages:

    gpg --send-key 0x726724204C644D83

## Revoking Old UIDs

If you're like me, your old key might have some addresses which you
have left behind.  You can't remove addresses from your keys, but you
can tell the world to stop using them.

To do this for my old key, I followed instructions on the 
[gnupg mailing list](http://lists.gnupg.org/pipermail/gnupg-users/2009-May/036425.html)

Everything still looks the same when you
[search](http://pgp.zdv.uni-mainz.de:11371/pks/lookup?op=index&search=beekhof&fingerprint=on)
for my old key:

    pub  1024D/D449E59A 2007-07-20 Andrew Beekhof <beekhof@mac.com>
                                   Andrew Beekhof <abeekhof@suse.de>
                                   Andrew Beekhof <beekhof@gmail.com>
                                   Andrew Beekhof <andrew@beekhof.net>
                                   Andrew Beekhof <abeekhof@novell.com>
    	 Fingerprint=E5F5 BEFC 781F 3637 774F  C1F8 EC35 84EF D449 E59A 

But if you click through to the 
[key details](http://pgp.zdv.uni-mainz.de:11371/pks/lookup?op=vindex&fingerprint=on&search=0xEC3584EFD449E59A),
you'll see the addresses associated with my time at Novell/SuSE now
show `revok` in red.

    pub  1024D/D449E59A 2007-07-20            
    	 Fingerprint=E5F5 BEFC 781F 3637 774F  C1F8 EC35 84EF D449 E59A 
    
    uid Andrew Beekhof <beekhof@mac.com>
    sig  sig3  D449E59A 2007-07-20 __________ __________ [selfsig]
    
    uid Andrew Beekhof <abeekhof@suse.de>
    sig  sig3  D449E59A 2007-07-20 __________ __________ [selfsig]
    sig revok  D449E59A 2013-06-24 __________ __________ [selfsig]
    ...

This is how other people's copy of `gpg` knows not to use this key for
that address anymore.  And also why its important to refresh your keys
periodically.

## Revoking Old Keys

Realistically though, you probably don't want people using old and
potentially compromised (or compromise-able) keys to send you
sensitive messages.  The best thing to do is
[revoke the entire key](http://www.hackdiary.com/2004/01/18/revoking-a-gpg-key/).

Since keys can't be removed once you've uploaded them, you're actually
updating the existing entry.  To do this you need the original private
key - so keep it safe!

Some people advise you to pre-generate the revocation key - personally
that seems like just one more thing to keep track of.

Orphaned keys that can't be revoked still appear valid to anyone
wanting to send you a secure message - a good reason to set an expiry
date as a failsafe!

This is what one of my old revoked key looks like:

    pub  1024D/DABA170E 2004-10-11 *** KEY REVOKED *** [not verified]
                                   Andrew Beekhof (SuSE VPN Access) <andrew@beekhof.net>
    	 Fingerprint=9A53 9DBB CF73 AB8F B57B  730A 3279 4AE9 DABA 170E 


## Final Result

My [new key](http://pgp.zdv.uni-mainz.de:11371/pks/lookup?op=vindex&fingerprint=on&search=0x726724204C644D83):

    pub  4096R/4C644D83 2013-06-24 Andrew Beekhof <andrew@beekhof.net>
                                   Andrew Beekhof <beekhof@mac.com>
                                   Andrew Beekhof <abeekhof@redhat.com>
    	 Fingerprint=C503 7BA2 D013 6342 44C0  122C 7267 2420 4C64 4D83 

## Closing word

I am by no means an expert at this, I would be very grateful to hear
about any mistakes I may have made above.
