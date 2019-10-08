# srsd - Sender Rewriting Scheme Daemon

A daemon written in Go that accepts and rewrites email addresses according to
the [Sender Rewriting System](https://en.wikipedia.org/wiki/Sender_Rewriting_Scheme).

The daemon listens on a Unix socket for requests to rewrite email addresses
either for forwarding or in reverse for bounce handling.

The [mileusna/srs](https://github.com/mileusna/srs) package is used perform the
rewriting.


## Compiling

From the source directory, run `go build` to produce a srsd executable.


## Configuration

srsd uses a JSON configuration file. A sample is provided as srsd.json. A
default secret can be provided in addition to secrets for specific target
SRS domains.


## Executing

Start the daemon by running `srsd`.

By default, srsd will look for a configuration file named srsd.json in the
current directory and open a socket named srsd.sock in the current directory. To
specify different configuration file and socket locations, use the
`-config-file` and `-socket-path` options respectively. For example:

```
srsd -config-file /etc/srsd.conf -socket-path /run/srsd/srsd.sock
```

The process will remain running in the foreground when started. To terminate
send the process an INT or TERM signal.

## Protocol

srsd accepts connections made to the Unix socket. Each connection can be used to
perform multiple operations (provided a syntax error is not encountered).
Rewrite commands are issued one per line (terminated with a line feed `\n`).

Requesting a forwarding rewrite for the sender address from@example.org using
the SRS domain srs.example.com:

```
> Fsrs.example.com:from@example.org
< SRS0=PmRI=YB=example.org=from@srs.example.com
```

Requesting a reverse rewrite for the SRS address
SRS0=PmRI=YB=example.org=from@srs.example.com:

```
> RSRS0=PmRI=YB=example.org=from@srs.example.com
< from@example.org
```

Errors will be reported prefixed with "ERROR: ":

```
> RSRS0=aaaa=YB=example.org=from@srs.example.com
< ERROR: Hash invalid in SRS address
> RSRS0=PmRI=aa=example.org=from@srs.example.com
< ERROR: Time stamp out of date
> Ffrom@example.org
< ERROR: Missing SRS domain or email address
```

The final error in the above example will result in the connection being closed
due to the syntax error.


## systemd

A systemd service file is provided as srsd.service.
