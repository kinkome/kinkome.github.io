# Applications

1. [[Motivation]]: Why do we need an infrastructure for kinko applications?
2. [[General principles]]
3. [[Implementation overview]]
4. [[Layout of a kinko application]]
5. [[The kinko Manifest file]]
6. [[Services accessible to kinko applications]]

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


> Back to [[Kinko Applications]].

The following text describes basic ideas behind parts of the implementation of the kinko core system

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


## Layout of a Kinko application

TBD


## The kinko Manifest file

TBD

## Services accessible to kinko applications

TBD


