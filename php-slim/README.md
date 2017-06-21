# build web apps on mobingiALM

 - install docker for macOS
 - build docker image
 - test apps on local machine
 - use mobingi private registry
 - push mobingi private registry
 - lunch ALM from build image.


## install docker for macOS

## build docker image

clone sample git repository
https://github.com/hkdstamp/ubuntu-apache2-php7

Change　this Dockerfile for dev.
and build docker image on local.
run docker image on local and debug source.
after all test pass, lunch ALM by using build image.

after lunch ALM stack, try to connect git repository.
above done, it is ready for development throght git.
local env change and push, ALM stack rebuild and switch new env automatic.
