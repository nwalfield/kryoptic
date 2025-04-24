This is a pkcs11 soft token written in rust

# Dependencies

 * rustc
 * openssl dependencies
 * sqlite

Note, the default feature links against the system installed OpenSSL
libraries, you need the OpenSSL development packages to build with
the default features selection.

On Debian trixie:

    apt install build-essential clang libclang-dev openssl libssl-dev libsqlite3-dev

# Setup

Kryoptic normally builds and dynamically links against a system version
of OpenSSL; alternatively the build system can be pointed to OpenSSL
sources to generate a build with the crypto library statically linked
into the binaries.

For builds that need to include a static build of OpenSSL, download and
unpack the desired version and set the env var KRYOPTIC_OPENSSL_SOURCES
to the path where the source were unpacked.

Example:

    export KRYOPTIC_OPENSSL_SOURCES=/path/to/src/openssl

# Build

Build the rust project:

    $ CONFDIR=/etc cargo build

For the FIPS build, you need to generate the hmac checksum:

    $ ./misc/hmacify.sh target/release/libkryoptic_pkcs11.so

The default build specifies "standard" as the default feature for
ease of use. "Standard" pulls in all the standard algorithms and the
sqlitedb storage backend.

In order to make a different selection you need to use the cargo
switch to disable default features (--no-default-features) and then
specify the features you want to build with:

eg: cargo build --no-default-features --features fips,sqlitedb,nssdb

# Tests

To run test, run the check command:

    $ cargo test

This command accepts the same feature set as the build command

# PCSC

Install pcsc-lite.

If you are doing development, you'll probably want to increase
`pcsc`'s verbosity.  Modify `/etc/default/pcscd` to pass the `--debug`
argument to `pcscd`:

    PCSCD_ARGS="--debug"

You'll need to restart `pcscd` for this to take effect:

    sudo systemctl restart pcscd

To follow the messages, run:

    sudo journalctl -xef -u pcscd

Assuming that you've built kryoptic and you are in that root
directory, copy the library to `/usr/lib/pcsc/drivers`:

    sudo cp target/debug/libkryoptic_pkcs11.so /usr/lib/pcsc/drivers

For development purposes it would be nice to create a symbolic link,
but `pcscd` refuses to follow symbolic links:

    Apr 24 10:16:22 debian-pkcs11 pcscd[12184]: 00000013 ../src/configfile.l:163:evaluatetoken() Error with library /usr/lib/pcsc/drivers/libkryoptic_pkcs11.so: Permission denied

Create the file `/etc/reader.conf.d/kryoptic.conf` with the following content:

    FRIENDLYNAME "kryoptic"
    LIBPATH /usr/lib/pcsc/drivers/libkryoptic_pkcs11.so

Create the `kryoptic` configuration file `/usr/local/etc/token.conf`
with the following content:

    [[slots]]
    slot = 1
    dbtype = "sql"
    dbargs = "/var/lib/kryoptic/token.sql"

Make sure `/var/lib/kryoptic` exists:

    sudo mkdir -p /var/lib/kryoptic

# License

The license is currently set as the GPLv3.0+ as released by the FSF.

This license is compatible with the OpenSSL ASL2.0 license and is a strong
copyleft license which we find useful.

Unlike other copyleft projects we are not dogmatic and chose this license
for the benefits we think it will brings to a self-contained project like
kryoptic. Namely that it strongly encourages modifications to be
contributed back.

If a party asks for it we will pragmatically evaluate a different license
and will be open to make a change if we think that such change would in fact
be in the best interest of the project. Note that requests of this kind
need to come with a well reasoned rationale that shows benefits both for
the requesting party and the upstream project.


# Contributions

Contributions to the project are made under the project's [License][License]
unless otherwise explicitly indicated by the contributor at the time of the
contribution.

See also the [default agreement](https://developercertificate.org/) we assume
for contribution which is currently enforced by the github DCO check.
