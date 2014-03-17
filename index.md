---
layout: index
---

> Note: The following is preliminary information. While we try our best to keep it
> accurate, we cannot promise to never make mistakes. If you find some errors here,
> please drop us a note at [contact@kinko.me](mailto:contact@kinko.me).

<h1 id="about" class="page-header">Privacy starts in your home! <small><a href="#email">Next: E-mail encryption</a></small></h1>

[kinko.me](https://kinko.me) implements an en/decrypting SMTP- and IMAP-proxy on ARM-class hardware, the kinko box. Emails are synced from the users' email accounts via IMAP to the box and are stored in plaintext in a secure storage area on the box. The kinko box also includes a webmailer to be able to use email with the browser.

Connections to the kinko box are secured by TLS using a private key only known to the box itself. Furthermore, the kinko box is tunnelled to a public internet location. Consequently, users can access secure email from everywhere, using IMAP compatible email clients and/or browsers, including mobile clients.

kinko uses GnuPG for encryption, with the addition of encrypting the email subject. Further additions should allow "Post-email alternatives" (a la bitmessage) to be used with the email clients that users are using today already. Other, privacy-related additions are planned as well.

Both the kinko box software and the routing service's software are open sourced.


<h1 id="email" class="page-header">E-mail encryption <small><a href='#device'>Next: Device discovery and setup</a></small></h1>

Initially kinko is used to provide pretty secure email for everyone - regardless of the tech skill
level of its users. To provide backwards compatibility with existing solutions for both email and
email encryption kinko is built on top of legacy email protocols (SMTP,IMAP) and crypto (PGP).


<h2 id='email-legacy'></h2>
## Using IMAP and SMTP with kinko

1. Once installed, the kinko box becomes available under a unique name such as *pinkpony.kinko.me*. 
	The kinko box provides an SMTP and an IMAP server under that name; IMAP and SMTP must be 
	configured to use that instead of the original mail server settings.

1. When **an email is sent via SMTP**, the email enters the kinko mail pipeline; and, in due course, 
	will be encrypted using the PGP keys stored on the keys, and then submitted to the original mail 
	server. This process will work synchronously - that means that if the original mail server rejects 
	the email for whatever reason, kinko does not accept it either, and the mail client will show an error message.

1. When the kinko box is powered on, it constantly synchronizes the original mail IMAP server. When an
	email arrives there (say: you write to contact@kinko.me), it will be picked up by the synchronization 
	process, will be decrypted on the kinko box, potentially parsed for spam, and then stored in the IMAP 
	server on the kinko box. This is where it becomes available for the user, in its decrypted form.

<h2 id='email-webmail'></h2>
## Using webmail with kinko

kinko comes with a preconfigured webmailer. One can use the webmailer to access the emails of her
accounts on the kinko system by opening the unique URL in the browser, e.g. *https://pinkpony.kinko.me*.

(We are currently using roundcube-email, but this might change)


<h2 id='email-gpg'></h2>
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

<h2 id='email-pipeline'></h2>
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

<h1 id="device" class="page-header">Device discovery and setup <small><a href='#applications'>Next: Kinko applications</a></small></h1>

So how do we make sure the communication between a user and her kinko installation
is really private? This is where the kinko box comes into play.

<h2 id='device-setup'></h2>
## Device setup process

When a kinko installation is powering up initially, it must be configured. For this it
connects to a pipe2me server and requests a short-lived name and certificate.

It then starts the device discovery process, which allows a user to take ownership of her
kinko installation.

Note: *pipe2me* is a client/server system, which tunnels TCP connections and provides security
by automatic provisioning with certificates and domain names. For this *pipe2me* employs SSL and 
SSH connections, with some configuration code provided by The kinko team. For more information
check [https://github.com/kinkome/pipe2me](https://github.com/kinkome/pipe2me)
and [https://github.com/kinkome/pipe2me-client](https://github.com/kinkome/pipe2me-client).


<h2 id='device-discovery'></h2>
## The device discovery process

During startup an unconfigured kinko box announces itself via *zeroconf*. The kinko desktop app
(see below) watches out for such notifications and allows a user to connect to her kinko box.

Once the user connects to her kinko box, she is asked to provide a strong password to initialize
disk encryption. A strong password suggestion will be provided.

<h2 id='device-desktop'></h2>
## The kinko desktop app

The kinko desktop application combines the following functionality:

- a **locked-down browser installation:** some components of the kinko installation
  comes with a user interface which requires an HTML browser. As regular browsers
  are inherently unsafe for security-critical code we provide a locked down installation
  that, for example, prevents 3rd party browser plugins to be installed.

- a **zeroconf browser** which watches the local network to allow the user to connect
  to and configure a not-yet-configured kinko installation.

- a **user notification** component, which allows the kinko box to notify the user and 
  to interact with her (for example to ask for explicit key validation.)

<h1 id="applications" class="page-header">Applications <small><a href='#online'>Next: Online Resources</a></small></h1>

<h2 id='applications-motivation'></h2>

## Motivation

Why do we need something like applications? There are a couple of reasons to implement an application-like infrastructure. The first reason is security with respect to updates:

### Secure updates thanks to applications

Updates are hard. Not only do we have to update the software on a remote box in an easy-to-use manner. But from a security perspective there are additional challenges: Lets assume we have a perfectly working kinko solution. A team of experienced security reviewer reviewed all the source code and found no flaws. Now we want a slightly improved software: the webmail component should be accessible in a new translation. How can we distribute the new software to the users?

If we would bundle a new binary package and put it on a user's device, why should she still trust the new software version, for example not to send her PGP keys back to the kinko servers? She trusted the initial version because of the external review (and because it is open source, etc.), but then the new package has not been reviewed by anyone. It could be possible that we changed the software.

The **application infrastructure** allows to separate concerns within the entire kinko software. For example, the web mailer needs not be able to read GnuPG keys at all - it must just be able to send and receive mail via SMTP and IMAP. That means, if the basic kinko infrastructure allows to be configured so that an application cannot just read any data on the device.

### 3rd party applications

Future additions to the kinko box should allow any user to install 3rd party applications. This way a user can
extend the functionality provided by her kinko box. Still, a 3rd party application must not be able to read and write random files on the disk or use resources at will.

Instead, a user will be able to install a 3rd party application by downloading it; the download will be verified, and then **the user will be asked whether or not she wants to grant permissions as asked by the application.**

For example, a video chat application might want access to the list of email addresses of a user to help set up the chat. The video chat application will have no reason to access GnuPG keys of any email account in the system. In that case the kinko db manager will export a permission "readaccounts" (or, in full, "kinko-db-readaccounts"), which lets the video chat application access just that piece of information.



<h2 id='applications-principles'></h2>

## General principles

### What are applications?

An application implements some functionality which should work in the context of the kinko application suite. This might be an extension to the kinko dashboard, some command line utilities that change the way kinko works (spam filters, for example), or just some data to be used in other applications (for example, the kinko-cabundle application provides cabundle for the kinko system.)

The kinko core system (based on Debian) provides a number of facilities for applications. These include:

- secure storage
- automatic backup of configurations
- automatic start and stop of applications
- application events
- secure communication between applications
- user notifications (kinko-notify)
- simple to use permission system
- integration into kinko web configuration frontend
- automatic testing
- automatic resource management

### Trusted applications

All applications are signed by the developers' public keys. Mismatching applications will not be installed.
During application the user will be asked to grant requested permissions to an application (say, the permission to read the accounts database).

Each application can define its own set of permissions to further extend the permission system.

### Application Stages

Each application belongs to a stage. An application in stage `N` will only be executed when the previous stages are running. For example, the `diskencryption` application, which provides a secure storage area
on disk for all applications to use, is part of *stage0*. That means that all applications in *stage1*, etc. can rely on `diskencryption` to be available.

### Application dependencies

Applications can define **application dependencies**. That way an application can only be installed when all of its dependencies are met; for example the `keymanager` application requires `gpg` to be installed. 

### Integration into kinko web frontend

An application can provide an HTML-based configuration part, to allow users to interact with it and/or to change its configuration. These parts are integrated automatically into the kinko web server (which is responsible to render the configuration frontend.)

### Automatic resource management

Each application and all of its processes may use up only a limited amount of resources on the system. Misbehaving applications will be terminated automatically.

### Process management

Each application can be in one of these states: 

- **uninstalled/inactive:** the application is present on disk, but is not configured
- **installed:** the application is installed and configured, but not running
- **running:** the application is installed, configured, and (if possible) running

Applications are run via a process manager. The final version will use `monit` to monitor processes and resources. An development environment might use `foreman` instead.

<h2 id='applications-overview'></h2>

## Implementation Overview

### Application directory layout

Each application has to adhere to a specific directory layout. For a full explanation see [[Layout-of-a-kinko-application]]. However, the main idea is like this:

- A `./bin` path contains binaries to be used from the application
- The `./var` path may be used to hold data used during runtime. The path will point to encrypted
  storage for applications at stage 2 or later.
- The `./etc` path may be used to hold configuration data. Such data is considered user input and will be included in the configuration backup.
  
### Secure Storage

Secure storage is implemented using Linux disk encryption.

### Automatic Backup of Configurations

The administrator can download a backup of all configurations. The backup contains a list of active applications, and the configuration data of those.

### Automatic starting and stopping of applications

### Secure communication between applications

Applications can share data and interfaces in a controlled way. The kinko system uses Linux user accounts and groups to organize applications into groups, and relies on file system access rights to manage access to various parts of an application.

That means an application may provide a number of permissions (that will turn into Linux user groups). Other applications may request a defined permission. Core applications (i.e. applications that come as part of the kinko email package and are signed by the kinko team) are automatically granted these permissions; when installing 3rd party applications the user will be asked to grant those permissions.   

### Application events

Applications may define events and subscribe to events. The kinko event system is very simple; a subscribing application only gets to know that an event occurred and when it occurred for the last time. It is not possible to attach data to an event.

### User notifications (kinko-notify)

Applications can notify the user about important events that might require user interaction. The user will see notifications in her kinko desktop app.

### Integration into the kinko web configuration frontend

Applications may contain a web configuration applet. These are integrated into the kinko configuration web server. For example, the webmail application contains the web mailer, which will automatically be integrated under the path /webmail. 

### Automatic testing

To help developers building 3rd party apps an application may provide a test infrastructure. When the kinko system is configured to do so, all applications' test scripts will be run during the installation of an application.


<h2 id='applications-layout'></h2>

## Layout of a Kinko application

TBD



<h2 id='applications-manifest'></h2>

## The kinko Manifest file

TBD


<h2 id='applications-services'></h2>

## Services accessible to kinko applications

TBD


<h2 id='applications-services'></h2>

## Services accessible to kinko applications

TBD


<h1 id="online" class="page-header">Online resources! <small><a href='#top'>Top</a></small></h1>

Find us

- **on the web:** [https://kinko.me](https://kinko.me)
- **on github:** [http://github.com/kinkome](http://github.com/kinkome)
- **on facebook:** [http://facebook.com/kinko.me](http://facebook.com/kinko.me)
