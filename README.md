# R2D2

[![CircleCI](https://circleci.com/gh/spreedly/r2d2.svg?style=svg)](https://circleci.com/gh/spreedly/r2d2)

R2D2 is a Ruby library for decrypting Android Pay payment tokens.

## Ruby support

Currently, only Ruby v2.2 and below are supported. For Ruby >= 2.3, work will need to be done (similar to [what was done in Gala](https://github.com/spreedly/gala/commit/0a4359ccdd5654b78747f9141645ca510ee255c2)) to use a compatible aead decryption algorithm.

## Install

Add to your `Gemfile`:

```ruby
gem "android_pay", git: "https://github.com/spreedly/android_pay.git"
```

## Usage

R2D2 takes input in the form of the hash of Android Pay token values:

```json
{
  "encryptedMessage": "ZW5jcnlwdGVkTWVzc2FnZQ==",
  "ephemeralPublicKey": "ZXBoZW1lcmFsUHVibGljS2V5",
  "tag": "c2lnbmF0dXJl"
}
```

and the merchant's private key private key (which is managed by a third-party such as a gateway or independent processor like [Spreedly](https://spreedly.com)).

```ruby
require "android_pay"

# token_json = raw token string you get from Android Pay { "encryptedMessage": "...", "tag": "...", ...}
token_attrs = JSON.parse(token_json)
token = R2D2::PaymentToken.new(token_attrs)

private_key_pem = File.read("private_key.pem")
decrypted_json = token.decrypt(private_key_pem)

JSON.parse(decrypted_json)
# =>
{
  “dpan”: “4444444444444444”,
  “expirationMonth”: 10,
  “expirationYear”: 2015 ,
  “authMethod”: “3DS”,
  “3dsCryptogram”: “AAAAAA...”,
  “3dsEciIndicator”: “eci indicator”
}
```

### Performance

The library implements a constant time comparison algorithm for preventing timing attacks. The default pure ruby implementation is quite inefficient, but portable. If performance is a priority for you, you can use a faster comparison algorithm provided by the [fast_secure_compare](https://github.com/daxtens/fast_secure_compare).

To enable `FastSecureCompare` in your environment, add the following to your Gemfile:

```ruby
gem 'fast_secure_compare`
```

and require the extension in your application prior to loading r2d2:

```ruby
require 'fast_secure_compare/fast_secure_compare'
require 'r2d2/payment_token'
```

Benchmarks illustrating the overhead of the pure Ruby version:

```
                          user     system      total        real
secure_compare        1.070000   0.010000   1.080000 (  1.231714)
fast secure_compare   0.050000   0.000000   0.050000 (  0.049753)
```

## Testing

```session
$ bundle exec rake
...
5 tests, 18 assertions, 0 failures, 0 errors, 0 skips
```

## Releasing

To cut a new gem:

### Setup RubyGems account

Make sure you have a [RubyGems account](https://rubygems.org) and have setup your local gem credentials with something like this:

```bash
$ curl -u rwdaigle https://rubygems.org/api/v1/api_key.yaml > ~/.gem/credentials; chmod 0600 ~/.gem/credentials
<enter rubygems account password>
```

If you are not yet listed as a gem owner, you will need to [request access](http://guides.rubygems.org/command-reference/#gem-owner) from @rwdaigle.

### Release

Build and release the gem with (all changes should be committed and pushed to Github):

```bash
$ rake release
```

## Changelog

### v0.1.2

* Setup CircleCI for more exhaustive Ruby version compatibility tests
* Add gem release instructions

## Contributors

* [methodmissing](https://github.com/methodmissing)
