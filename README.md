# placme - a Perl client for the ACME protocol

**placme is no longer maintained due to the abundance of other clients that have come into being. I use https://github.com/srvrco/getssl now in case you wanted a recommendation.**

The ACME protocol powers services that automate SSL/TLS certificate
generation, notably [Let's Encrypt](https://letsencrypt.org/).
This is a Perl-based client for it that handles account registration,
authorization (mainly HTTP method), and requesting certificates/revocations.

Design goals:

* Mostly self-contained: the modules and binaries used are *extremely* common
* Allow unprivileged operation
* Provide a toolbox for scripting your own solutions as well as a fully
  automated mode (not implemented yet)

Dependencies:

* JSON Perl package
* LWP Perl package (Debian: libwww-perl)
* OpenSSL binary
* Perl 5.12 or newer

## A brief introduction to ACME with placme

placme stores all of its working data in a single directory (called the config
directory). It defaults to `~/.config/placme`.
Create the config directory, along with a private key for your account:

    placme bootstrap --contact=mailto:jast@fancy.example

placme is pre-configured to work with Let's Encrypt. To use a different
service, you can just set its directory URL (the order of options matters, so
be sure to put the directory option first):

    placme bootstrap --directory=https://acme.example/directory --contact=...

The bootstrap operation outputs a link to the terms of service. After you have
read them, you can confirm that you agree to them by feeding back the URL:

    placme updatereg --agreement=https://acme.example/tos.pdf

To get a certificate, you first need to authorize your account for each of the
domain names you want to include in the certificate. The simplest way to do
that is to have placme automatically write the validation files to your web
directories (this means you need to have a web server set up that responds on
the given hostnames):

    placme authz --domain=my.example --http=send --keyauth-dir=/var/www/my.example/.well-known/acme-challenge --wait 15

This tells placme to request a challenge, set up the response, notify the
server, and wait up to 15 seconds for the ACME service to validate it.

Once that is out of the way, you can ask placme to fetch a certificate, given
either a certificate request (CSR) containing the desired names, or an RSA
private key and a list of domains to make placme generate a request:

    placme cert --csr=my.example.csr.pem
    # or
    placme cert --key=my.example.key.pem my.example

All of these operations have additional options to support a variety of use
cases. Specifically, the `authz` operation can be split into two or three
phases, like this:

    placme authz --domain=my.example
    placme confirm --token=<one of the tokens from authz's output, depending on which challenge you want to respond to>
    # [set up the challenge response using the keyauth output]
    # see the ACME spec for more details
    placme confirm --url=<corresponding challenge URL> --token=<same token again> --wait=15

    placme authz --http --domain=my.example
    # [set up the challenge response using the keyauth output]
    # see the ACME spec for more details
    placme confirm --url=<HTTP challenge URL> --token=<token from authz output> --wait=15

