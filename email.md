---
layout: index
---

# E-mail encryption

Initially kinko is used to provide pretty secure email for everyone - regardless of the tech skill
level of its users. To provide backwards compatibility with existing solutions for both email and
email encryption kinko is built on top of legacy email protocols (SMTP,IMAP) and crypto (PGP).

<a name='gpg'></a>
## PGP as used by kinko

### Subject encryption

While usually the subject is not encrypted with PGP, we also move the subject into the message body,
replacing the subject with a "This email is encrypted" text.

Users of the kinko box will never see this subject, as the receiving kinko box automatically
restores the original mail message. Users of other PGP based solutions will find the original subject
embedded into the message.

### Key generation

Keys are either generated on the kinko box when installing an email address, or uploaded by the user.

### Key detection

Keys are detected in

- incoming email: email that passes through will be checked for PGP keys.
- key servers: keys can be found on key servers.

In addition keys can be uploaded by the user.

### Key validation and trust

- **Trust on first use:** Incoming keys that are assigned to an email address
  where the system does not know any key before are automatically trusted.
  **This is the default mode.** 
- **Manual verification:** Incoming keys that are assigned to an email address
  where the system does not know any key before are automatically trusted.

In addition, organizations can set up their own trust infrastructure. When configured, there are
additional ways to gain trust:

- **Trusted keyservers:** Users can configure an authoritatative key server. Keys that are stored on that
  keyserver are automatically trusted.
- **CA signatures:** Organizations can set up an organization key. All public keys signed with that key are
  automatically trusted.

### User intervention

While intended to work mostly automatic, the kinko system still allows the user to interact
with the key detection and validation process.

When configured accordingly, the kinko system will ask the user to explicitely accept a PGP key.
More about user intervention in general can be found in the Kinko UI article. 

<a name='mail_pipeline'></a>
## The kinko mail pipeline

Each email entering the system is subject to the kinko mail pipeline. The email pipeline consists
of a number of components, that are built independently. Each component is built as a Unix pipe -
accepting emails as input and calling the next component in the pipeline to hand over the email.

In contrast to a regular mail queue in a mail server the kinko mail pipeline works synchronously -
allowing to accept or reject a message instantly and reliably when it enters the system.

Initially there are two mail pipelines in the system:

- kinko-mail-receive, consisting of 
  - kinko-gpg-keydetection
	- kinko-gpg-decrypt
	- kinko-spam-spamcheck
- kinko-mail-send, consisting of
  - kinko-gpg-keydetection
	- kinko-gpg-encrypt
	- kinko-mail-forward

