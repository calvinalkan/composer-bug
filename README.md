# Composer dev dependencies bug [#10913](https://github.com/composer/composer/issues/10913)

## Situation

Required package A (`laminas/laminas-httphandlerrunner`) declares a dependency on [psr/http-message-implementation](https://github.com/laminas/laminas-httphandlerrunner/blob/2.2.x/composer.json#L35).
Dev package B (`snicco/testing-bundle`) does not provide an implementation for any psr7 package but depends on [`codeception/codeception` which requires `guzzlehttp/psr7`](https://github.com/Codeception/Codeception/blob/4.2.1/composer.json#L23).

## Expected behaviour:

When running `composer install --no-dev` the installation should wail with the warning that `laminas/laminas-httphandlerrunner` requires a psr7 implemenation.

## Actual behaviour

Composer will silently install `guzzlehttp/psr7` without it being declared anywhere.

## Reproduce

1. ` 
   rm -rf vendor composer.lock && composer install --no-dev && composer show | grep guzzle
   ` 
   
   Output: `guzzlehttp/psr7                   2.4.0 PSR-7 message implementation that also provides common utility methods`


2. `
   composer why guzzlehttp/psr7
   `
   
   Output: `There is no installed package depending on "guzzlehttp/psr7"`


3. `composer remove --dev snicco/testing-bundle && rm -rf vendor composer.lock && composer install --no-dev`
   
   This fails now with the prompt to install a psr7 implementation.

## composer diagnose

```log
No license specified, it is recommended to do so. For closed-source software you may use "proprietary" as license.
require.laminas/laminas-httphandlerrunner : exact version constraints (2.1.0) should be avoided if the package follows semantic versioning
Checking platform settings: OK
Checking git settings: OK
Checking http connectivity to packagist: OK
Checking https connectivity to packagist: OK
Checking github.com oauth access: OK
Checking disk free space: OK
Checking pubkeys: 
Tags Public Key Fingerprint: 57815BA2 7E54DC31 7ECC7CC5 573090D0  87719BA6 8F3BB723 4E5D42D0 84A14642
Dev Public Key Fingerprint: 4AC45767 E5EC2265 2F0C1167 CBBB8A2B  0C708369 153E328C AD90147D AFE50952
OK
Checking composer version: OK
Composer version: 2.3.8
PHP version: 7.4.28
PHP binary path: /usr/local/Cellar/php@7.4/7.4.28_1/bin/php
OpenSSL version: OpenSSL 1.1.1m  14 Dec 2021
cURL version: 7.82.0 libz 1.2.11 ssl (SecureTransport) OpenSSL/1.1.1m
zip: extension present, unzip present, 7-Zip not available
```