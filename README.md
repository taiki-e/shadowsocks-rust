# shadowsocks-rust

[![Build Status](https://img.shields.io/travis/shadowsocks/shadowsocks-rust.svg)](https://travis-ci.org/shadowsocks/shadowsocks-rust)
[![Build status](https://ci.appveyor.com/api/projects/status/h3ny0dov7v9xioa5?svg=true)](https://ci.appveyor.com/project/zonyitoo/shadowsocks-rust-0grjf)
[![License](https://img.shields.io/github/license/zonyitoo/shadowsocks-rust.svg)](https://github.com/zonyitoo/shadowsocks-rust)
[![crates.io](https://img.shields.io/crates/v/shadowsocks-rust.svg)](https://crates.io/crates/shadowsocks-rust)
[![Release](https://img.shields.io/github/release/shadowsocks/shadowsocks-rust.svg)](https://github.com/shadowsocks/shadowsocks-rust/releases)
[![CircleCI](https://circleci.com/gh/shadowsocks/shadowsocks-rust.svg?style=svg)](https://circleci.com/gh/shadowsocks/shadowsocks-rust)

This is a port of [shadowsocks](https://github.com/shadowsocks/shadowsocks).

shadowsocks is a fast tunnel proxy that helps you bypass firewalls.

## Dependencies

* libcrypto (OpenSSL) (Required for `aes-*`, `camellia-*` and `rc4` ciphers)
* libsodium >= 1.0.7 (Required for ciphers that are provided by libsodium)

## Build & Install

### Optional Features

* `sodium` - Enabled linking to [`libsodium`](https://github.com/jedisct1/libsodium), which will also enable ciphers that depending on `libsodium`.

* `rc4` - Enabled `rc4` encryption algorithm. Some OpenSSL Crypto does not ship with `rc4`, because it was already deprecated in 2015.

* `aes-cfb` - Enabled `aes-*-cfb` encryption algorithm.

* `aes-ctr` - Enabled `aes-*-ctr` encryption algorithm.

* `camellia-cfb` - Enabled `camellia-*-cfb` encryption algorithm.

* `aes-pmac-siv` - Enabled `aes-*-pmac-siv` encryption algorithm. (Experimental)

* `single-threaded` - Let `sslocal` and `ssserver` run in single threaded mode (by using Tokio's `basic_scheduler`).

* `trust-dns` - Uses [`trust-dns-resolver`](https://crates.io/crates/trust-dns-resolver) as DNS resolver instead of `tokio`'s builtin.

* `local-http` - Allow using HTTP protocol for `sslocal`

Default features: `["sodium", "rc4", "aes-cfb", "aes-ctr", "trust-dns", "local-http"]`.

NOTE: To disable dependency of OpenSSL, just disable feature `rc4`, `aes-cfb`, `aes-ctr`, `camellia-cfb`.

### **crates.io**

Install from [crates.io](https://crates.io/crates/shadowsocks-rust):

```bash
cargo install shadowsocks-rust
```

then you can find `sslocal` and `ssserver` in `$CARGO_HOME/bin`.

### **Download release**

Requirements:

* Linux x86\_64
* Windows x86\_64

Download static-linked build [here](https://github.com/shadowsocks/shadowsocks-rust/releases).

Nightly builds could be downloaded from [CircleCI](https://app.circleci.com/pipelines/github/shadowsocks/shadowsocks-rust). [HOW TO](https://github.com/shadowsocks/shadowsocks-rust/issues/251#issuecomment-628692564)

* `build-windows`: Build for `x86_64-pc-windows-msvc`
* `build-linux`: Build for `x86_64-unknown-linux-gnu`, Debian 9 (Stretch), GLIBC 2.18
* `build-docker`: Build for `x86_64-unknown-linux-musl`, `x86_64-pc-windows-gnu`, ... (statically linked)

### **Build from source**

Use cargo to build.

```bash
cargo build --release
```

NOTE: [`libsodium-sys`](https://crates.io/crates/libsodium-sys) builds and links statically to the `libsodium` in its package. Here are some useful environment variables for customizing build processes:

* Find `libsodium` installed in customized path:

  * `SODIUM_LIB_DIR` - Directory path to `libsodium.a` or `sodium.lib`
  * `SODIUM_SHARED` - Dynamic-link instead of default static-link

* Find `libsodium` with `pkg-config` (*nix), `vcpkg` (MSVC)

  * `SODIUM_USE_PKG_CONFIG=1`

```bash
SODIUM_USE_PKG_CONFIG=1 cargo build --release
```

Then `sslocal` and `ssserver` will appear in `./target/(debug|release)/`, it works similarly as the two binaries in the official ShadowSocks' implementation.

```bash
make install TARGET=release
```

Then `sslocal`, `ssserver`, `sstunnel` and `ssurl` will be installed in `/usr/local/bin` (variable PREFIX).

For Windows users, if you have encountered any problem in building, check and discuss in [#102](https://github.com/shadowsocks/shadowsocks-rust/issues/102).

### **Build standalone binaries**

Requirements:

* Docker

```bash
./build/build-release
```

Then `sslocal`, `ssserver`, `sstunnel` and `ssurl` will be packaged in

* `./build/shadowsocks-${VERSION}-stable.x86_64-unknown-linux-musl.tar.xz`
* `./build/shadowsocks-${VERSION}-stable.x86_64-pc-windows-gnu.zip`

Read `Cargo.toml` for more details.

### Build OpenSSL from source

Specify feature `openssl-vendored` to let [openssl](https://crates.io/crates/openssl) build from source.

```bash
cargo build --features "openssl-vendored"
```

## Getting Started

Create a ShadowSocks' configuration file. Example

```json
{
    "server": "my_server_ip",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "mypassword",
    "timeout": 300,
    "method": "aes-256-gcm"
}
```

Detailed explanation could be found in [shadowsocks' documentation](https://github.com/shadowsocks/shadowsocks/wiki).

In shadowsocks-rust, we also have an extended configuration file format, which is able to define more than one servers:

```json
{
    "servers": [
        {
            "address": "127.0.0.1",
            "port": 1080,
            "password": "hello-world",
            "method": "aes-256-gcm",
            "timeout": 300
        },
        {
            "address": "127.0.0.1",
            "port": 1081,
            "password": "hello-kitty",
            "method": "chacha20-ietf-poly1305"
        }
    ],
    "local_port": 8388,
    "local_address": "127.0.0.1"
}
```

The `sslocal` will use a load balancing algorithm to dispatch packages to all servers.

Start local and server ShadowSocks with
If you Build it with Makefile:

```bash
sslocal -c config.json
ssserver -c config.json
```

If you Build it with Cargo:

```bash
cargo run --bin sslocal -- -c config.json
cargo run --bin ssserver -- -c config.json
```

List all available arguments with `-h`.

## Usage

### Socks5 Local client

```bash
# Read local client configuration from file
sslocal -c /path/to/shadowsocks.json

# Pass all parameters via command line
sslocal -b "127.0.0.1:1080" -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty" --plugin "obfs-local" --plugin-opts "obfs=tls"

# Pass server with SIP002 URL
sslocal -b "127.0.0.1:1080" --server-url "ss://YWVzLTI1Ni1nY206cGFzc3dvcmQ@127.0.0.1:8388/?plugin=obfs-local%3Bobfs%3Dtls"
```

### HTTP Local client

```bash
# Read local client configuration from file
sslocal -c /path/to/shadowsocks.json --protocol http
```

All parameters are the same as Socks5 client, except `--protocol http`.

### Tunnel Local client

```bash
# Read local client configuration from file
# Set 127.0.0.1:8080 as the target for forwarding to
sslocal -c /path/to/shadowsocks.json -f "127.0.0.1:8080" --protocol tunnel
```

### Transparent Proxy Local client

**NOTE**: This is currently only supports

* Linux (with `iptables` targets `REDIRECT` and `TPROXY`)
* BSDs (with `pf`), such as OS X 10.10+, FreeBSD, ...

```bash
# Read local client configuration from file
sslocal -c /path/to/shadowsocks.json --protocol redir
```

Redirects connections with `iptables` configurations to the port that `sslocal` is listening on.

### Server

```bash
# Read server configuration from file
ssserver -c /path/to/shadowsocks.json

# Pass all parameters via command line
ssserver -s "[::]:8388" -m "aes-256-gcm" -k "hello-kitty" --plugin "obfs-server" --plugin-opts "obfs=tls"
```

### Server Manager

Supported [Manage Multiple Users](https://github.com/shadowsocks/shadowsocks/wiki/Manage-Multiple-Users) API:

* `add` - Starts a server instance
* `remove` - Deletes an existing server instance
* `list` - Lists all current running servers
* `ping` - Lists all servers' statistic data

NOTE: `stat` command is not supported. Because servers are running in the same process with the manager itself.

```bash
# Start it just with --manager-address command line parameter
ssmanager --manager-address "127.0.0.1:6100"

# For *nix system, manager can bind to unix socket address
ssmanager --manager-address "/tmp/shadowsocks-manager.sock"

# You can also provide a configuration file
#
# `manager_address` key must be provided in the configuration file
ssmanager -c /path/to/shadowsocks.json

# Create one server by UDP
echo 'add: {"server_port":8388,"password":"hello-kitty"}' | nc -u '127.0.0.1' '6100'

# Close one server by unix socket
echo 'remove: {"server_port":8388}' | nc -Uu '/tmp/shadowsocks-manager.sock'
```

For manager UI, check more details in the [shadowsocks-manager](https://github.com/shadowsocks/shadowsocks-manager) project.

Example configuration:

```jsonc
{
    // Required option
    // Address that ssmanager is listening on
    "manager_address": "127.0.0.1",
    "manager_port": 6100,

    // Or bind to a Unix Domain Socket
    "manager_address": "/tmp/shadowsocks-manager.sock",

    "servers": [
        // These servers will be started automatically when ssmanager is started
    ],

    // Outbound socket binds to this IP address
    // For choosing different network interface on the same machine
    "local_address": "xxx.xxx.xxx.xxx",

    // Other options that may be passed directly to new servers
}
```

## Configuration

```jsonc
{
    // LOCAL: Listen address
    // SERVER: Bind address for remote sockets, mostly used for choosing interface
    "local_address": "127.0.0.1",
    "local_port": 1080,

    // Server's configuration
    "server": "0.0.0.0",
    "server_port": 8388,
    "method": "aes-256-gcm",
    "password": "your-password",
    "plugin": "v2ray-plugin",
    "plugin_opts": "mode=quic;host=www.shadowsocks.com",
    "timeout": 5, // Timeout for TCP relay server (in seconds)

    // Extended multiple server configuration
    // LOCAL: Choosing the best server to connect dynamically
    // SERVER: Creating multiple servers in one process
    "servers": [
        {
            // Fields are the same as the single server's configuration
            "address": "0.0.0.0",
            "port": 8389,
            "method": "aes-256-gcm",
            "password": "your-password",
            "plugin": "...",
            "plugin_opts": "...",
            "timeout": 5,
        }
    ],

    // Global configurations for UDP associations
    "udp_timeout": 5, // Timeout for UDP associations (in seconds), 5 minutes by default
    "udp_max_associations": 512, // Maximum UDP associations to be kept in one server, unlimited by default

    // Options for Manager
    "manager_address": "127.0.0.1", // Could be a path to UNIX socket, /tmp/shadowsocks-manager.sock
    "manager_port": 5300, // Not needed for UNIX socket

    // DNS server's address for resolving domain names
    // For *NIX and Windows, it uses system's configuration by default
    //
    // Value could be IP address of DNS server, for example, "8.8.8.8".
    // DNS client will automatically request port 53 with both TCP and UDP protocol.
    //
    // It also allows some pre-defined well-known public DNS servers:
    // - google (TCP, UDP)
    // - cloudflare (TCP, UDP)
    // - cloudflare_tls (TLS), enable by feature "dns-over-tls"
    // - cloudflare_https (HTTPS), enable by feature "dns-over-https"
    // - quad9 (TCP, UDP)
    // - quad9_tls (TLS), enable by feature "dns-over-tls"
    //
    // The field is only effective if feature "trust-dns" is enabled.
    "dns": "google",

    // Mode, could be one of the
    // - tcp_only
    // - tcp_and_udp
    // - udp_only
    "mode": "tcp_only",

    // TCP_NODELAY
    "no_delay": false,

    // Soft and Hard limit of file descriptors on *NIX systems
    "nofile": 10240,

    // Try to resolve domain name to IPv6 (AAAA) addresses first
    "ipv6_first": false
}
```

## Supported Ciphers

### Stream Ciphers

* `aes-128-cfb`, `aes-128-cfb1`, `aes-128-cfb8`, `aes-128-cfb128`
* `aes-192-cfb`, `aes-192-cfb1`, `aes-192-cfb8`, `aes-192-cfb128`
* `aes-256-cfb`, `aes-256-cfb1`, `aes-256-cfb8`, `aes-256-cfb128`
* `aes-128-ctr`
* `aes-192-ctr`
* `aes-256-ctr`
* `camellia-128-cfb`, `camellia-128-cfb1`, `camellia-128-cfb8`, `camellia-128-cfb128`
* `camellia-192-cfb`, `camellia-192-cfb1`, `camellia-192-cfb8`, `camellia-192-cfb128`
* `camellia-256-cfb`, `camellia-256-cfb1`, `camellia-256-cfb8`, `camellia-256-cfb128`
* `rc4`, `rc4-md5`
* `chacha20`, `salsa20`, `chacha20-ietf`
* `plain` (No encryption, just for debugging)

### AEAD Ciphers

* `aes-128-gcm`, `aes-256-gcm`
* `chacha20-ietf-poly1305`, `xchacha20-ietf-poly1305`
* `aes-128-pmac-siv`, `aes-256-pmac-siv` (experimental)

## ACL

`sslocal`, `ssserver`, and `ssmanager` support ACL file with syntax like [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev). Some examples could be found in [here](https://github.com/shadowsocks/shadowsocks-libev/tree/master/acl).

### Available sections

* For local servers (`sslocal`, `ssredir`, ...)
  * Modes:
    * `[bypass_all]` - ACL runs in `BlackList` mode. Bypasses all addresses that didn't match any rules.
    * `[proxy_all]` - ACL runs in `WhiteList` mode. Proxies all addresses that didn't match any rules.
  * Rules:
    * `[bypass_list]` - Rules for connecting directly
    * `[proxy_list]` - Rules for connecting through proxies
* For remote servers (`ssserver`)
  * Modes:
    * `[reject_all]` - ACL runs in `BlackList` mode. Rejects all clients that didn't match any rules.
    * `[accept_all]` - ACL runs in `WhiteList` mode. Accepts all clients that didn't match any rules.
  * Rules:
    * `[white_list]` - Rules for accepted clients
    * `[black_list]` - Rules for rejected clients
    * `[outbound_block_list]` - Rules for blocking outbound addresses.

### Example

```ini
# SERVERS
# For ssserver, accepts requests from all clients by default
[accept_all]

# Blocks these clients
[black_list]
1.2.3.4
127.0.0.1/8

# Disallow these outbound addresses
[outbound_block_list]
127.0.0.1/8
::1
(^|\.)baidu.com

# CLIENTS
# For sslocal, ..., bypasses all targets by default
[bypass_all]

# Proxy these addresses
[proxy_list]
(^|\.)google.com
8.8.8.8
```

## Useful Tools

1. `ssurl` is for encoding and decoding ShadowSocks URLs (SIP002). Example:

  ```plain
  ss://YWVzLTI1Ni1jZmI6cGFzc3dvcmQ@127.0.0.1:8388/?plugin=obfs-local%3Bobfs%3Dhttp%3Bobfs-host%3Dwww.baidu.com
  ```

## Notes

It supports the following features:

* [x] SOCKS5 CONNECT command
* [x] SOCKS5 UDP ASSOCIATE command (partial)
* [x] SOCKS4/4a CONNECT command
* [x] Various crypto algorithms
* [x] Load balancing (multiple servers) and server delay checking
* [x] [SIP004](https://github.com/shadowsocks/shadowsocks-org/issues/30) AEAD ciphers
* [x] [SIP003](https://github.com/shadowsocks/shadowsocks-org/issues/28) Plugins
* [x] [SIP002](https://github.com/shadowsocks/shadowsocks-org/issues/27) Extension ss URLs
* [x] HTTP Proxy Supports ([RFC 7230](http://tools.ietf.org/html/rfc7230) and [CONNECT](https://tools.ietf.org/html/draft-luotonen-web-proxy-tunneling-01))
* [x] Defend against replay attacks, [shadowsocks/shadowsocks-org#44](https://github.com/shadowsocks/shadowsocks-org/issues/44)
* [x] Manager APIs, supporting [Manage Multiple Users](https://github.com/shadowsocks/shadowsocks/wiki/Manage-Multiple-Users)
* [x] ACL (Access Control List)
* [x] Support HTTP/HTTPS Proxy protocol

## TODO

* [x] Documentation
* [x] Extend configuration format
* [x] Improved logging format (waiting for the new official log crate)
* [ ] Support more ciphers without depending on `libcrypto` (waiting for an acceptable Rust crypto lib implementation)
* [x] Windows support.
* [x] Build with stable `rustc`.
* [x] Support HTTP Proxy protocol
* [ ] One-time Auth. (Already deprecated according to Shadowsocks' community)
* [x] AEAD ciphers. (proposed in [SIP004](https://github.com/shadowsocks/shadowsocks-org/issues/30), still under discussion)
* [x] Choose server based on delay #152
* [ ] Support TCP Fast Open

## License

[The MIT License (MIT)](https://opensource.org/licenses/MIT)

Copyright (c) 2014 Y. T. CHUNG

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
