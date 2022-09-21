# Gaurun [![GitHub release](https://img.shields.io/github/release/mercari/gaurun.svg?style=flat-square)][release] ![GitHub Actions](https://github.com/mercari/gaurun/workflows/Go/badge.svg)

[release]: https://github.com/mercari/gaurun/releases

<img src="https://raw.githubusercontent.com/mercari/gaurun/master/img/logo.png" alt="logo" align="right"/>


Gaurun is a general push notification server written in Golang. It proxies push requests to APNs and FCM and asynchronously executes them via HTTP/2. It helps you when you need to bulkly sends push notification to your users (e.g., when you need to exec 10 million push at once!) or when some other API server which must response quickly needs to push. Since it leverages Golang's powerful concurrent feature, it gives high performance. 

In addition to performance, it's important not to lost pushes over sever crashes or hardware failures. Gaurun can use its access log for kind of transaction journal and can re-push only failed notification later (We provide a special command for this. See [Usage](#usage)). 

Currently we support the following platforms:

- [Apple APNs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html)
- [Google FCM](https://firebase.google.com/docs/cloud-messaging/)

## Status

Production ready.

## Installation

There are two way to install Gaurun; using a precompiled binary or install from source. Downloading a precompiled binary is easiest and recommended.

To install a precompiled binary, download the appropriate zip package for your OS and architecture from [here](https://github.com/mercari/gaurun/releases). Once the zip is downloaded, unzip it and place the binary where you want to use (if you want to access it from the command-line, make sure to put it on `$PATH`).

To compile from source, you need Go1.15 or later. After setup, then clone the source code by running the following command,

```bash
$ git clone https://github.com/mercari/gaurun.git
```

To fetch dependencies and build, run the following make tasks,

```bash
make
```

## Usage

To run `gaurun`, you must provide configuration path via `-c` option (See [CONFIGURATION.md](/CONFIGURATION.md) about details),

```bash
$ bin/gaurun -c conf/gaurun.toml
```

Use `-help` to see more options.

### Crash Recovery

Gaurun can recover from server crashes or hardware failures while pushing. It can use its access log for kind of transaction journal and can re-push only failed notifications later. We provide the special command for this, use it like the following (assuming that access log is generated to `/tmp/gaurun.log`),

```bash
$ bin/gaurun_recover -c conf/gaurun.toml -l /tmp/gaurun.log
```

## Configuration

See [CONFIGURATION.md](/CONFIGURATION.md) about details.

*NOTE*: The default configuration is just for development and not high performant. For production usage, tune the performance with some parameters such as `workers` and `queues` and `pusher_max` in the `core` section.

## Specification

API specification is defined on [SPEC.md](/SPEC.md).

## Contribution

Please read the CLA below carefully before submitting your contribution.

https://www.mercari.com/cla/

## License

Copyright 2014-2019 Mercari, Inc.


Licensed under the MIT License.

## memo
```shell
openssl pkcs12 -clcerts -nokeys -out push_cert.pem -in push_cert.p12
openssl pkcs12 -nocerts -out push_key.pem -in push_cert.p12 -nodes
```
でpemキーを作る。(-nodesでパスフレーズをスキップ、p12をキーチェーンから作る時に空文字でパスフレーズを設定)

```shell
sudo openssl s_client -connect api.push.apple.com:443 -cert push_cert.pem -key push_key.pem
```
で確認する。

### fcm/send
gaurunでは旧APIを使っているぽい。
```shell
curl --location --request POST 'https://fcm.googleapis.com/fcm/send' \
--header 'Content-Type: application/json' \
--header 'Authorization: key=AAAAsUoNtEs:APA91bGLZQ8KTr0TPxPqZ3m9IbUw3kyC2u8_y9lxW6KNpaY9dEPLm5b6VNQfQOi2ByqAKivig9WOn-MPhpPaZPBeoeGU8aMCtKml2q02ZCSYcXYpQSGFX0p7CnZacRX78zzGRzeUnefW' \
--data-raw '{ 
    "notification": {
      "body": "This is an FCM notification message!",
      "time": "FCM Message"
    },
    "registration_ids": [ "cCC7Hyz5h0SsqOmwgShcpf:APA91bF9RwxTjJRB4VaaxXS0bnGznunSrpKoBpdMjMCsj_nOwXtnriu6dFmRW-9KKCWXKy5MZ9FVe5ZAKPWOTXibJyO3Bf4pyGlVKTkWvcjMzvSqa5Z3uRd-mgApIdKx0SXZ-OxSVmxC"]
}'
```
これでメッセージが届いた。