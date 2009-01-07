== NAME ==

<blockquote>swaks - Swiss Army Knife SMTP, the all-purpose smtp transaction tester</blockquote>

== DESCRIPTION ==

<blockquote>
swaks' primary design goal is to be a transaction-oriented SMTP test tool, used for debugging mail servers. stress testing configs, etc.  It handles SMTP features and extensions such as TLS, authentication, and pipelining; multiple version of the SMTP protocol including SMTP, ESMTP, and LMTP; and multiple transport methods including unix-domain sockets, internet-domain sockets, and pipes to spawned processes.  Options can be specified in environment variables, configuration files, and the command line with a goal of being extremely configurable for ease of use for operators and scripters.
</blockquote>

== QUICK START ==

<blockquote>
Deliver a standard test email to user@example.com on port 25 of test-server.example.net:
: swaks --to user@example.com --server test-server.example.net
Deliver a standard test email, requiring CRAM-MD5 authentication as user me@example.com.  An "X-Test" header will be added to the email body.  The authentication password will be prompted for.
: swaks --to user@example.com --from me@example.com --auth CRAM-MD5 --auth-user me@example.com --header-X-Test "test email"
Test a virus scanner using EICAR in an attachment.  Don't show the message DATA part.:
: swaks -t user@example.com --attach - --server test-server.example.com --suppress-data </path/to/eicar.txt
Test a spam scanner using GTUBE in the body of an email, routed via the MX records for example.com:
: swaks --to user@example.com --body /path/to/gtube/file
Deliver a standard test email to user@example.com using the LMTP protocol via a UNIX domain socket file
: swaks --to user@example.com --socket /var/lda.sock --protocol LMTP
Report all the recipients in a text file that are non-verifyiable on a test server:
: for E in `cat /path/to/email/file`
: do
:: swaks --to $E --server test-server.example.com --quit-after RCPT --hide-all
:: [ $? -ne 0 ] && echo $E
: done
</blockquote>

== TERMS AND CONVENTIONS ==

<blockquote>
It can be confusing discussing the various portions of an SMTP transaction.  This document tries to be consistent in its use of terms to avoid unnecessary confusion.
</blockquote><blockquote>
Transaction
: A transaction is the opening of connection over a transport to a target and using a messaging protocol to attempt to deliver a message.
</blockquote><blockquote>
Target
: The target of a transaction is the thing that swaks connects to.  This generic term is used throughout the documentation because most other terms improperly imply something about the transport being used.
</blockquote><blockquote>
Transport
: The transport is the underlying method used to connect to the target.
</blockquote><blockquote>
Protocol
: The protocol is the application language used to communicate with the target.  This document uses SMTP to speak generically of all three supported protocols unless it states that it is speaking of the specific 'SMTP' protocol and excluding the others.
</blockquote><blockquote>
Message
: SMTP protocols exist to transfer messages, a set of bytes in an agreed-upon format that has a sender and a recipient.
</blockquote><blockquote>
Envelope
: A message's envelope contains the "true" sender and receiver of a message.  It can also be referred to as its components, envelope-sender and envelope-recipients.  It is important to note that a messages envelope does not have to match its To: and From: headers.
</blockquote><blockquote>
DATA
: The DATA portion of an SMTP transaction is the actual message that is being transported.  It consists of both the message's headers and its body.  DATA and body are sometimes use synonymously, but they are always two distinct things in this document.
</blockquote><blockquote>
Headers
: A message's headers are defined as all the lines in the message's DATA section before the first blank line.  They contain information about the email that will be displayed to the recipient such as To:, From:, Subject:, etc.  In this document headers will always be written with a capitalized first letter and a trailing colon.
</blockquote><blockquote>
Body
: A message's body is the portion of its DATA section following the first blank line.
</blockquote>

== OPTION PROCESSING ==

<blockquote>
To prevent potential confusion in this document a flag to swaks is always referred to as an "option".  If the option takes additional data, that additional data is referred to as an argument to the option.  For example, "--from fred@example.com" might be provided to swaks on the command line, with "--from" being the option and "fred@example.com" being --from's argument.
</blockquote><blockquote>
Options can be given to swaks in three ways.  They can be specified in a configuration file, in environment variables, and on the command line.  Depending on the specific option and whether or not an argument is given to it, swaks may prompt the user for the argument.
</blockquote>

<blockquote>
When swaks evaluates its options, it first looks for a configuration file (either in a default location or specified with --config).  Then it evaluates any options in environment variables.  Finally, it evaluates command line options.  At reach round of processing, any options set earlier will be overridden.  Additionally, any option can be prefixed with "no-" to cause swaks to forget that the variable had previously been set.  This capability is necessary because many options treat defined-but-no-argument differently than not-defined.
</blockquote>

<blockquote>
The exact mechanism and format for using each of the types is listed below.
</blockquote>

=== CONFIGURATION FILE ===

<blockquote>
A configuration file to set commonly-used or abnormally verbose options can be used.  By default swaks looks in order for $SWAKS_HOME/.swaksrc, $HOME/.swaksrc, and $LOGDIR/.swaksrc.  If one of those is found to exist (and --config has not been used) that file is used as the configuration file.
</blockquote><blockquote>
Additionally a configuration file in a non-default location can be specified using --config.  If this is set and not given an argument swaks will not use any configuration file, including any default file.  If --config points to a readable file, it is used as the configuration file, overriding any default that may exist.  If it points to a non-readable file and error will be shown and swaks will exit.
</blockquote><blockquote>
A set of "portable" defaults can also be created by adding a perl __DATA__ section to the end of the swaks program file.  At the end of the script add a single line containing "__DATA__", and any lines below that will be treated as the contents of a configuration file.  This allows a set of user preferences to be automatically copied from server to server in a single file.
</blockquote><blockquote>
If present and configuration files have not been explicitly turned off, the __DATA__ config is always read.  Only one other configuration file will ever be used per single invocation of swaks, even if multiple configuration files are specified.  Specifying the --config option with no argument turns off the processing of both the __DATA__ config and any actual config files.
</blockquote><blockquote>
The format of the configuration file is designed to allow future expansion.  Currently it only allows the equivalent of command line options to be specified.  These options must be given between a line of "<OPTIONS>" and a line of "</OPTIONS>".  Between these two lines, lines beginning with a hash (#) are ignored.  All other lines are assumed to be an option to swaks.  In this file the leading dash or dashes to options are optional.  Here is an example of the contents of a configuration file:
<pre>
<OPTIONS>
# always use this sender, no matter server or logged in user
--from fred@example.com
# I prefer my test emails have a pretty from header (note lack of dashes on option)
h-From: "Fred Example" <fred@example.com>
</OPTIONS>
</pre>
Note that everything after the first space is assumed to be the option's argument and is not shell processed, so any quoting is usually unneeded and will not be stripped off by the shell.
</blockquote><blockquote>
There is a deprecated option --input-file (or -l) in swaks that was a precursor of the configuration file defined here.  That option has been judged deficient and is being replaced wholesale with the idea of the configuration file defined above.  The option still exists for the time being but its use is strongly discouraged.  It is no longer documented, and it will be removed entirely in some future release.
</blockquote>

=== ENVIRONMENT VARIABLES ===

<blockquote>
Options can be supplied via environment variables.  The variables are in the form $SWAKS_OPT_name, where name is the name of the option that would be specified on the command line.  Because dashes aren't allowed in environment variable names in most unix-ish shells, no leading dashes should be used and any dashes inside the option's name should be replaced with underscores.  The following would create the same options shown in the configuration file example:

<pre>
$> SWAKS_OPT_from='fred@example.com'
$> SWAKS_OPT_h_From='"Fred Example" <fred@example.com>'
</pre>

Setting a variable to an empty value is the same as specifying it on the command line with no argument.  For instance, setting SWAKS_OPT_server="" would cause swaks to prompt the use for the server to which to connect at each invocation.
</blockquote><blockquote>
In addition to setting the equivalent of command line options, SWAKS_HOME can be set to a directory containing the default .swaksrc to be used.
</blockquote>

=== COMMAND LINE OPTIONS ===

<blockquote>
The final method of supplying options to swaks is via the command line.  The options behave in a manner consistent with most unix-ish command line programs.  Many options have both a short and long form (for instance -s and --server).  By convention short options are specified with a single dash and long options are specified with a double-dash.  This is only a convention and either prefix will work with either type.
</blockquote><blockquote>
The following demonstrates the example shown in the configuration file and environment variable sections:
<pre>
$> swaks --from fred@example.com --h-From: '"Fred Example" <fred@example.com>'
</pre>
</blockquote>

== TRANSPORTS ==

<blockquote>
swaks can connect to a destination via unix pipes ("pipes"), unix domain sockets ("unix sockets"), or internet domain sockets ("network sockets").  Connecting via network sockets is the default behavior.  Because of the singular nature of the transport used, each set of options in the following section is mutually exclusive.  Specifying more than one of --server, --pipe, or --socket will result in an error.  Mixing other options between transport types will only result in the irrelevant options being ignored.  Below is a brief description of each type of transport and the options that are specific to that transport type.
</blockquote>

=== NETWORK SOCKETS ===

<blockquote>
This transport attempts to deliver a message via TCP/IP, the standard method for delivering SMTP.  This is the default transport for swaks.  If none of --server, --pipe, or --socket are given then this transport is used and the destination server is determined from the recipient's domain (see --server below for more details).
</blockquote><blockquote>
This transport requires the IO::Socket module which is part of the standard perl distribution.  If this module is not loadable, attempting to use a this transport will result in an error and program termination.
</blockquote><blockquote>
--server [destination mail server[:port]], -s [destination mail server[:port]]
: Explicitly tell swaks to use network sockets and specify the hostname or IP address to which to connect, or prompt if no argument is given.  If this option is not given and no other transport option is given, the target mail server is determined from the appropriate DNS records for the domain of the recipient email address using the Net::DNS module.  If Net::DNS is not available swaks will attempt to connect to localhost to deliver.  See also --copy-routing.
</blockquote><blockquote>
--port [port], -p [port]
: Specify which TCP port on the target is to be used, or prompt if no argument is listed.  The argument can be a service name (as retrieved by getservbyname(3)) or a port number.  The default port is determined by the --protocol option.  See --protocol for more details.
</blockquote><blockquote>
--local-interface [IP or hostname], -li [IP or hostname]
: Use argument as the local interface for the outgoing SMTP connection, or prompt user if no argument given.  Argument can be an IP address or a hostname.  Default action is to let the operating system choose local interface.
</blockquote><blockquote>
--copy-routing [domain]
: The argument is interpreted as the domain part of an email address and it is used to find the destination server using the same logic that would be used to look up the destination server for an recipient email address.  See  --to option for more details on how the target is determined from the email domain.
</blockquote>

=== UNIX SOCKETS ===

<blockquote>
This transport method attempts to deliver messages via a unix-domain socket file.  This is useful for testing MTA/MDAs that listen on socket files (for instance, testing LMTP delivery to Cyrus).  This transport requires the IO::Socket module which is part of the standard perl distribution.  If this module is not loadable, attempting to use this transport will result in an error and program termination.
</blockquote><blockquote>
--socket [/path/to/socket/file]
: This option takes as its argument a unix-domain socket file.  If swaks is unable to open this socket it will display an error and exit.
</blockquote>

=== PIPES ===

<blockquote>
This transport attempts to spawn a process and communicate with it via pipes.  The spawned program must be prepared to behave as a mail server over STDIN/STDOUT.  Any MTA designed to operate from inet/xinet should support this.  In addition some MTAs provide testing modes that can be communicated with via STDIN/STDOUT.  This transport can be used to automate that testing.  For example, if you implemented DNSBL checking with Exim and you wanted to make sure it was working, you could run 'swaks --pipe "exim -bh 127.0.0.2"'.  In an ideal world the process you are talking to should behave exactly like an SMTP server on stdin and stdout.  Any debugging should be sent to stderr, which will be directed to your terminal.  In the real world swaks can generally handle some debug on the child's stdout, but there are no guarantees on how much it can handle.
</blockquote><blockquote>
This transport requires the IPC::Open2 module which is part of the standard perl distribution.  If this module is not loadable, attempting to use this transport will result in an error and program termination.
</blockquote><blockquote>
--pipe [/path/to/command and arguments]
: Provide a process name and arguments to the process.  swaks will attempt to spawn the process and communicate with it via pipes.  If the argument is not an executable swaks will display an error and exit.
</blockquote>

== PROTOCOL OPTIONS ==
<blockquote>
These options are related to the protocol layer.
</blockquote><blockquote>
--to [email-address[,email-address,...]], -t [email-address[,email-address,...]]
: Tells swaks to use argument(s) as the envelope-recipient for the email, or prompt for recipient if no argument provided.  If multiple recipients are provided and the recipient domain is needed to determine routing the domain of the last recipient provided is used.
: There is no default value for this option.  If no recipients are provided via any means, user will be prompted to provide one interactively.  The only exception to this is if a --quit-after value is provided which will cause the smtp transaction to be terminated before the recipient is needed.
</blockquote><blockquote>
--from [email-address], -f [email-address]
: Use argument as envelope-sender for email, or prompt user if no argument specified.  The string <> can be supplied to mean the null sender.  If user does not specify a sender address a default value is used.  The domain-part of the default sender is a best guess at the fully-qualified domain name of the local host.  The method of determining the local-part varies.  On Windows, Win32::LoginName() is used.  On unix-ish platforms, the $LOGNAME environment variable is used if it is set.  Otherwise getpwuid(3) is used.  See also --force-getpwuid.
</blockquote><blockquote>
--helo [helo-string], --ehlo [helo-string], --lhlo [helo-string], -h [helo-string]
: String to use as argument to HELO/EHLO/LHLO command, or prompt use if no argument is specified.  If this option is not used a best guess at the fully-qualified domain name of the local host is used.  If the Sys::Hostname module, which is part of the base distribution, is not available the user will be prompted for a HELO value.  Note that Sys::Hostname has been observed to not be able to find the local hostname in certain circumstances.  This has the same effect as if Sys::Hostname were unavailable.
</blockquote><blockquote>
--quit-after [stop-point], -q [stop-point]
: Point at which the transaction should be stopped.  When the requested stopping point is reached in the transaction, and provided that swaks has not errored out prior to reaching it,  swaks will send "QUIT" and attempt to close the connection cleanly.  These are the valid arguments and notes about their meaning.
<blockquote>
{| cellpadding=5 cellspacing=5
|-
| align=left valign=top | CONNECT
| Terminate the session after receiving the greeting banner from the target.  BANNER is a valid synonym.
|-
| align=left valign=top | FIRST-HELO
| In a STARTTLS (but not tls-on-connect) session, terminate the transaction after the first of two HELOs.  In a non-STARTTLS transaction, behaves the same as HELO (see below).  FIRST-EHLO and FIRST-LHLO are valid synonyms.
|-
| align=left valign=top | TLS
| Quit the transaction immediately following TLS negotiation.  Note that this happens in different places depending on whether STARTTLS or tls-on-connect are used.  This always quits after the point where TLS would have been negotiated, regardless of whether it was attempted.
|-
| align=left valign=top | HELO
| In a STARTTLS session, quit after the second HELO.  Otherwise quit after the first and only HELO.  EHLO and LHLO are valid synonyms.
|-
| align=left valign=top | AUTH
| Quit after authentication.  This always quits after the point where authentication would have been negotiated, regardless of whether it was attempted.
|-
| align=left valign=top | MAIL
| Quit after MAIL FROM: is sent.  FROM is a valid synonym.
|-
| align=left valign=top | RCPT
| Quit after RCPT TO: is sent.  TO is a valid synonym.
|}
</blockquote>
</blockquote><blockquote>
--timeout [time]
: Use argument as the SMTP transaction timeout, or prompt user if no argument given.  Argument can either be a pure digit, which will be interpretted as seconds, or can have a specifier s or m (5s = 5 seconds, 3m = 180 seconds).  As a special case, 0 means don't timeout the transactions.  Default value is 30s.
</blockquote><blockquote>
--protocol [protocol]
: Specify which protocol to use in the transaction.  Valid options are shown in the table below.  Currently the 'core' protocols are SMTP, ESMTP, and LMTP.  By using variations of these protocol types one can tersely specify default ports, whether authentication should be attempted, and the type of TLS connection that should be attempted.  The default protocol is ESMTP.  This table demonstrates the available arguments to --protocol and the options each sets as a side effect:
<blockquote>
{| cellpadding=5 cellspacing=5
! Protocol !! HELO Verb      !! Auth Options !! TLS Options !! Port
|-
| SMTP    
| HELO            
|         
|         
| smtp  / 25
|-
| SSMTP   
| EHLO->HELO      
|         
| -tlsc   
| smtps / 465
|-
| SSMTPA  
| EHLO->HELO      
| align=center | -a      
| -tlsc   
| smtps / 465
|-
| SMTPS   
| HELO            
|         
| -tlsc   
| smtps / 465
|-   
| ESMTP   
| EHLO->HELO      
|         
|         
| smtp  / 25
|-   
| ESMTPA  
| EHLO->HELO      
| align=center | -a      
|         
| smtp  / 25
|-   
| ESMTPS  
| EHLO->HELO      
|         
| -tls    
| smtp  / 25
|-   
| ESMTPSA 
| EHLO->HELO      
| align=center | -a      
| -tls    
| smtp  / 25
|-   
| LMTP    
| LHLO            
|         
|         
| lmtp  / 24
|-   
| LMTPA   
| LHLO            
| align=center | -a      
|         
| lmtp  / 24
|-   
| LMTPS   
| LHLO            
|         
| -tls    
| lmtp  / 24
|-   
| LMTPSA  
| LHLO            
| align=center | -a      
| -tls    
| lmtp  / 24
|}
</blockquote>
</blockquote><blockquote>
--pipeline
: If the remote server supports it, attempt SMTP PIPELINING (RFC 2920).  This is a younger option, if you experience problems with it please notify the author.  Potential problem areas include servers accepting DATA even though there were no valid recipients (swaks should send empty body in that case, not QUIT) and deadlocks caused by sending packets outside the tcp window size.
</blockquote><blockquote>
--force-getpwuid
: Tell swaks to use the getpwuid method of finding the default sender local-part instead of trying $LOGNAME first.
</blockquote>

== TLS / ENCRYPTION ==
<blockquote>
These are options related to encrypting the transaction.  These have been tested and confirmed to work with all three transport methods.  The Net::SSLeay module is used to perform encryption when it is requested.  If this module is not loadable swaks will either ignore the TLS request or error out, depending on whether the request was optional.  STARTTLS is defined as an extension in the ESMTP protocol and will be unavailable if --protocol is set to a variation of smtp.  Because it is not defined in the protocol itself, --tls-on-connect is available for any protocol type if the target supports it.
</blockquote><blockquote>
-tls
: Require connection to use STARTTLS.  Exit if TLS not available for any reason (not advertised, negotiations failed, etc).
--tls-optional, -tlso
: Attempt to use STARTTLS if available, continue with normal transaction if TLS was unable to be negotiated for any reason
</blockquote><blockquote>
--tls-optionsl-strict, -tlsos
: Attempt to use STARTTLS if available.  Proceed with transaction if TLS is negotiated successfully or STARTTLS not advertised.  If STARTTLS is advertised but TLS negotiations fail, treat as an error and abort transaction.
</blockquote><blockquote>
--tls-on-connect, -tlsc
: Initiate a TLS connection immediately on connection.  Following common convention, if this option is specified the default port changes from 25 to 465, though this can still be overridden with the --port option.
--tls-get-peer-cert [/path/to/file]
: Get a copy of the TLS peer's certificate.  If no argument is given, it will be displayed to STDOUT.  If an argument is given it is assumed to be a filesystem path specifying where the certificate should be written.  The saved certificate can then be examined using standard tools such as the openssl command.  If a file is specified its contents will be overwritten.
</blockquote>

== AUTHENTICATION ==
<blockquote>
If instructed to do so swaks will attempt to authenticate to the target mail server.  This section details available authentication types, requirements, options and their interactions, and other fine points in authentication usage.  Because authentication is defined as an extension in the ESMTP protocol it will be unavailable if --protocol is set to a variation of smtp.
</blockquote><blockquote>
All authentication methods require base64 encoding.  If the MIME::Base64 perl module is loadable swaks attempts to use it to perform these encodings.  If MIME::Base64 is not available swaks will use its own onboard base64 routines.  These are slower than the MIME::Base64 routines and less reviewed, though they have been tested thoroughly.  Using the MIME::Base64 module is encouraged.
</blockquote><blockquote>
If authentication is required (see options below for when it is and isn't required) and the requirements aren't met for the authentication type available, swaks displays an error and exits.  Two ways this can happen include forcing swaks to use a specific authentication type that swaks can't use due to missing requirements, or allowing swaks to use any authentication type, but the server only advertises types swaks can't support.  In the former case swaks errors out at option processing time since it knows up front it won't be able to authenticate.  In the latter case swaks will error out at the authentication stage of the smtp transaction since swaks will not be aware that it will not be able to authenticate until that point.
</blockquote><blockquote>
Following are the supported authentication types including any individual notes and requirements.
</blockquote><blockquote>
The following options affect swaks' use of authentication.  These options are all inter-related.  For instance, specifying --auth-user implies --auth and --auth-password.  Specifying --auth-optional implies --auth-user and --auth-password, etc.
</blockquote><blockquote>
--auth [auth-type[,auth-type,...]], -a [auth-type[,auth-type,...]]
: Require swaks to authenticate.  If no argument is given, any supported auth-types advertised by the server are tried until one succeeds or all fail.  If one or more auth-types are specified as an argument, each that the server also supports is tried in order until one succeeds or all fail.  This option requires swaks to authenticate, so if no common auth-types are found or no credentials succeed, swaks displays an error and exits.
: The following tables lists the valid auth-types
<blockquote>
{| border=0 cellpadding=5 cellspacing=5
| valign=top | LOGIN, PLAIN
| These basic authentication types are fully supported and tested and have no additional requirements
|-
| valign=top | CRAM-MD5
| The CRAM-MD5 authenticator requires the Digest::MD5 module.  It is fully tested and believed to work against any server that implements it.
|-
| valign=top | DIGEST-MD5
| The DIGEST-MD5 authenticator (RFC2831) requires the Authen::DigestMD5 module.  Only known to have been tested against Communigate and may therefore have some implementation deficiencies.
|- 
| valign=top | CRAM-SHA1
| The CRAM-SHA1 authenticator requires the Digest::SHA1 module.  This type has only been tested against a non-standard implementation on an Exim server and may therefore have some implementation deficiencies.
|-
| valign=top | NTLM/SPA/MSN
| These authenticators require the Authen::NTLM module.  Note that there are two modules using the Authen::NTLM namespace on CPAN.  The Mark Bush implementation (Authen/NTLM-1.03.tar.gz) is the version required by swaks.  This type has been tested against Exim, Communigate, and Exchange 2007.

In addition to the standard username and password, this authentication type can also recognize a "domain".  Rather than create a new option for this single authentication type, the domain can be passed by adding "%DOMAIN" to the end of the username.  For instance, if "-ap user@example.com%NTDOM" is passed, "user@example.com" is the username and "NTDOM" is the domain.  Note that this has never been tested with a mail server that doesn't ignore DOMAIN so this may be implemented incorrectly.
|}
</blockquote>

</blockquote><blockquote>
--auth-optional [auth-type[,auth-type,...]], -ao [auth-type[,auth-type,...]]
: This option behaves identically to --auth except that it requests authentication rather than requiring it.  If no common auth-types are found or no credentials succeed, swaks proceeds as if authentication had not been requested.
</blockquote><blockquote>
--auth-optional-strict [auth-type[,auth-type,...]], -aos [auth-type[,auth-type,...]]
: This option is a compromise between --auth and --auth-optional.  If no common auth-types are found, swaks behaves as if --auth-optional were specified and proceeds with the transaction.  If swaks can't support requested auth-type, the server doesn't advertise any common auth-types, or if no credentials succeed, swaks behaves as if --auth were used and exits with an error.
</blockquote><blockquote>
--auth-user [username], -au [username]
: Provide the username to be used for authentication, or prompt the user for it if no argument is provided.  The string <> can be supplied to mean an empty username.
</blockquote><blockquote>
--auth-password [password], -ap [password]
: Provide the password to be used for authentication, or prompt the user for it if no argument is provided.  The string <> can be supplied to mean an empty password.
</blockquote><blockquote>
--auth-map [auth-alias=auth-type[,...]], -am [auth-alias=auth-type[,...]]
: Provides a way to map alternate names onto base authentication types.  Useful for any sites that use alternate names for common types.  This functionality is actually used internally to map types SPA and MSN onto the base type NTLM.  The command line argument to simulate this would be "--auth-map SPA=NTLM,MSN=NTLM".  All of the auth-types listed above are valid targets for mapping except SPA and MSN.
</blockquote><blockquote>
--auth-plaintext, -apt
: Instead of showing AUTH strings literally (in base64), translate them to plaintext before printing on screen.
</blockquote>

== DATA OPTIONS ==
<blockquote>
These options pertain to the contents for the DATA portion of the SMTP transaction.
</blockquote><blockquote>
--data [data-portion], -d [data-portion]
: Use argument as the entire contents of DATA, or prompt user if no argument specified.  If the argument '-' is provided the data will be read from STDIN.  If any other argument is provided and it represents the name of an open-able file, the contents of the file will be used.  Any other argument will be itself for the DATA contents.
: The value can be on one single line, with \n (ascii 0x5c, 0x6e) representing where line breaks should be placed.  Leading dots will be quoted.  Closing dot is not required but is allowed.  The default value for this option is "Date: %D\nTo: %T\nFrom: %F\nSubject: test %D\nX-Mailer: swaks v$p_version jetmore.org/john/code/#swaks\n%H\n\n%B\n".
: Very basic token parsing is performed on the DATA portion.  The following table shows the recognized tokens and their replacement values:
<blockquote>
{|
|-
| %F
| Replaced with the envelope-sender.
|-
| %T
| Replaced with the envelope-recipient(s).
|-
| valign=top | %D
| Replaced with the current time in a format suitable for inclusion in the Date: header.  Note this attempts to use the standard module Time::Local for timezone calculations.  If this module is unavailable the date string will be in GMT.
|-
| %H
| Replaced with the contents of the --add-header option.  If --add-header is not specified this token is simply removed.
|-
| %B
| Replaced with the value specified by the --body option.  See --body for default.
|}
</blockquote>
</blockquote><blockquote>
--body [body-specification]
: Specify the body of the email.  The default is "This is a test mailing".  If no argument to --body is given, prompt to supply one interactively.  If '-' is supplied, the body will be read from standard input.  If any other text is provided and the text represents an open-able file, the content of that file is used as the body.  If it does not represent an open-able file, the text itself is used as the body.
</blockquote><blockquote>
--attach [attachment-specification]
: When one or more --attach option is supplied, the message is changed into a multipart/mixed MIME message.  The arguments to --attach are processed the same as --body with regard to stdin, file contents, etc.  --attach can be supplied multiple times to create multiple attachments.  By default each attachment is attached as a application/octet-stream file.  See --attach-type for changing this behavior.
: When the message changes to MIME format, the previous body (%B) is attached as a text/plain type as the first attachment.  --body can still be used to specify the contents of this body attachment.
: It is legal for '-' (STDIN) to be specified as an argument multiple times (once for --body and multiple times for --attach).  In this case, the same content will be attached each time it is specified.  This is useful for attaching the same content with multiple MIME types.
</blockquote><blockquote>
--attach-type [mime-type]
: By default, content that gets MIME attached to a message with the --attach option is encoded as application/octet-stream.  --attach-type changes the mime type for every --attach option which follows it.  It can be specified multiple times.
</blockquote><blockquote>
--add-header [header], -ah [header]
: This option allows headers to be added to the DATA.  If %H is present in the DATA it is replaced with the argument to this option.  If %H is not present, the argument is inserted between the first two consecutive newlines in the DATA (that is, it is inserted at the end of the existing headers).
: The option can either be specified multiple times or a single time with multiple headers seperated by a literal '\n' string.  So, "--add-header 'Foo: bar' --add-header 'Baz: foo'" and "--add-header 'Foo: bar\nBaz: foo'" end up adding the same two headers.
</blockquote><blockquote>
--header [header-and-data], --h-Header [data]
: These options allow a way to change headers that already exist in the DATA.  '--header "Subject: foo"' and '--h-Subject foo' are equivalent.  If the header does not already exist in the data then this argument behaves identically to --add-header.  However, if the header already exists it is replaced with the one specified.
</blockquote><blockquote>
-g
: If specified, swaks will read the DATA value for the mail from STDIN.  This is equivalent to "--data -".  If there is a From_ line in the email, it will be removed (but see -nsf option).  Useful for delivering real message (stored in files) instead of using example messages.
</blockquote><blockquote>
--no-strip-from, -nsf
: Don't strip the From_ line from the DATA portion, if present.
</blockquote>

== OUTPUT OPTIONS ==
<blockquote>
By default swaks provides a transcript of its transactions to its caller (STDOUT/STDERR).  This transcript aims to be as faithful a representation as possible of the transaction though it does modify this output by adding informational prefixes to lines and by providing plaintext versions of TLS transactions
</blockquote><blockquote>
The "informational prefixes" are referred to as transaction hints.  These hints are initially composed of those marking lines that are output of swaks itself, either informational or error messages, and those that indicate a line of data actually sent or received in a transaction.  This table indicates the hints and their meanings:
</blockquote><blockquote><blockquote>
{|cellpadding=5 cellspacing=5
|-
| valign=top align=left | ===
| program
| Indicates an informational line generated by swaks
|-
| valign=top align=left | ***
| program
| Indicates an error generated within swaks
|-
| valign=top align=left | ->
| transaction
| Indicates an expected line sent by swaks to target server
|-
| valign=top align=left | ~>
| transaction
| Indicates a TLS-encrypted, expected line sent by swaks to target server
|-
| valign=top align=left | **>
| transaction
| Indicates an unexpected line sent by swaks to the target server
|-
| valign=top align=left | *~>
| transaction
| Indicates a TLS-encrypted, unexpected line sent by swaks to target server
|-
| valign=top align=left | <-
| transaction
| Indicates an expected line sent by target server to swaks
|-
| valign=top align=left | <~
| transaction
| Indicates a TLS-encrypted, expected line sent by target server to swaks
|-
| valign=top align=left | <**
| transaction
| Indicates an unexpected line sent by target server to swaks
|-
| valign=top align=left | <~*
| transaction
| Indicates a TLS-encrypted, unexpected line sent by target server to swaks
|}
</blockquote></blockquote><blockquote>
The following options control what and how output is displayed to the caller.
</blockquote><blockquote>
--suppress-data, -n
: Summarizes the DATA portion of the SMTP transaction instead of printing every line.  This option is very helpful, bordering on required, when using swaks to send certain test emails.  Emails with attachments, for instance, will quickly overwhelm a terminal if the DATA is not supressed.
</blockquote><blockquote>
--show-time-lapse [i], -stl [i]
: Display time lapse between send/receive pairs.  This option is most useful when Time::HiRes is available, in which case the time lapse will be displayed in thousandths of a second.  If Time::HiRes is unavailable or "i" is given as an argument the lapse will be displayed in integer seconds only.
</blockquote><blockquote>
--no-hints, -nth
: Don't show transaction hints (useful in conjunction with -hr to create copy/paste-able transactions).
</blockquote><blockquote>
--hide-receive, -hr
: Don't display lines sent from the remote server being received by swaks
</blockquote><blockquote>
--hide-send, -hs
: Don't display lines being sent by swaks to the remote server
</blockquote><blockquote>
--hide-informational, -hi
: Don't display non-error informational lines from swaks itself.
</blockquote><blockquote>
--hide-all, -ha
: Do not display any content to the terminal.
</blockquote><blockquote>
--silent [level], -S [level]
: Cause swaks to be silent.  If no argument is given or if an argument of "1" is given, print no output unless/until an error occurs, after which all output is shown.  If an argument of "2" is given, only print errors.  If "3" is given, show no output ever.
: Note that this used to be an additive option ("-S -S" was equivalent to "-S 2").  After environment option handling was introduced this was changed.  The additive method still works but is deprecated and will be removed entirely in a future release
</blockquote><blockquote>
--support
: Print capabilities and exit.  Certain features require non-standard perl modules.  This options evaluates whether these modules are present and displays which functionality is available and which isn't, and which modules would need to be added to gain the missing functionality.
</blockquote><blockquote>
--help
: Display this help information.
</blockquote><blockquote>
--version
: Display version information.</blockquote>

== PORTABILITY ==

<blockquote>
Operating Systems
: This program was primarily intended for use on unix-like operating systems, and it should work on any reasonable version thereof.  It has been developed and tested on Solaris, Linux, and Mac OS X and is feature complete on all of these.
</blockquote><blockquote>
: This program is known to demonstrate basic functionality on Windows using ActiveState's Perl.  It has not been fully tested.  Known to work are basic SMTP functionality and the LOGIN, PLAIN, and CRAM-MD5 auth types.  Unknown is any TLS functionality and the NTLM/SPA and Digest-MD5 auth types.
</blockquote><blockquote>
: Because this program should work anywhere Perl works, I would appreciate knowing about any new operating systems you've thoroughly used swaks on as well as any problems encountered on a new OS.
</blockquote><blockquote>
Mail Servers
: This program was almost exclusively developed against Exim mail servers.  It was been used casually by the author, though not thoroughly tested, with Sendmail, Smail, Exchange, Oracle Collaboration Suite, and Communigate.  Because all functionality in swaks is based off of known standards it should work with any fairly modern mail server.  If a problem is found, please alert the author at the address below.
</blockquote>

== EXIT CODES ==
<blockquote>
{|cellpadding=5 cellspacing=5
|-
| valign=top align=right | 0
| no errors occurred
|-
| valign=top align=right | 1
| error parsing command line options
|-
| valign=top align=right | 2
| error connecting to remote server
|-
| valign=top align=right | 3
| unknown connection type
|-
| valign=top align=right | 4
| while running with connection type of "pipe", fatal problem writing to or reading from the child process
|-
| valign=top align=right | 5
| while running with connection type of "pipe", child process died unexpectedly.  This can mean that the program specified with --pipe doesn't exist.
|-
| valign=top align=right | 6
| Connection closed unexpectedly.  If the close is detected in response to the 'QUIT' swaks sends following an unexpected response, the error code for that unexpected response is used instead.  For instance, if a mail server returns a 550 response to a MAIL FROM: and then immediately closes the connection, swaks detects that the connection is closed, but uses the more specific exit code 23 to detail the nature of the failure.  If instead the server return a 250 code and then immediately closes the connection, swaks will use the exit code 6 because there is not a more specific exit code.
|-
| valign=top align=right | 10
| error in prerequisites (needed module not available)
|-
| valign=top align=right | 21
| error reading initial banner from server
|-
| valign=top align=right | 22
| error in HELO transaction
|-
| valign=top align=right | 23
| error in MAIL transaction
|-
| valign=top align=right | 24
| no RCPTs accepted
|-
| valign=top align=right | 25
| server returned error to DATA request
|-
| valign=top align=right | 26
| server did not accept mail following data
|-
| valign=top align=right | 27
| server returned error after normal-session quit request
|-
| valign=top align=right | 28
| error in AUTH transaction
|-
| valign=top align=right | 29
| error in TLS transaction
|-
| valign=top align=right | 32
| error in EHLO following TLS negotiation
|-
|}
</blockquote>

== CONTACT ==
<blockquote>
proj-swaks@jetmore.net
: Please use this address for general contact, questions, patches, requests, etc.
</blockquote><blockquote>
updates-swaks@jetmore.net
: If you would like to be put on a list to receive notifications when a new version of swaks is released, please send an email to this address.
</blockquote><blockquote>
http://www.jetmore.org/john/code/#swaks
:Change logs, this help, and the latest version is found at this link.
</blockquote>
