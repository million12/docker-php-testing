# Docker container with PHP and Selenium server
[![Circle CI](https://circleci.com/gh/million12/docker-php-testing.png?style=badge)](https://circleci.com/gh/million12/docker-php-testing)

This is a [million12/php-testing](https://registry.hub.docker.com/u/million12/php-testing) container for running PHP tests using phpunit and/or [Behat](http://behat.org/) tests. Selenium server is installed and running, also there is a VNC server so you can connect to it to inspect the browser while tests are running.

This container is based on the [million12/nginx-php](https://github.com/million12/docker-nginx-php) container. If you use it for you application, you have exactly the same environment for the application and for testing. That gives you consistent results and a guarantee that if your test are passing, your app is working.

For different PHP versions, look up different branches of this repository.  
On Docker Hub you can find them under different tags:  
* `million12/php-testing:latest` - PHP 7.0 # built from `master` branch [![Circle CI](https://circleci.com/gh/million12/docker-php-testing.svg?style=svg)](https://circleci.com/gh/million12/docker-php-testing)
* `million12/php-testing:php70` - PHP 7.0 # built from `php70` branch [![Circle CI](https://circleci.com/gh/million12/docker-php-testing/tree/php70.svg?style=svg)](https://circleci.com/gh/million12/docker-php-testing/tree/php70)
* `million12/php-testing:php56` - PHP 5.6 # built from `php56` branch [![Circle CI](https://circleci.com/gh/million12/docker-php-testing/tree/php56.svg?style=svg)](https://circleci.com/gh/million12/docker-php-testing/tree/php56)
* `million12/php-testing:php55` - PHP 5.5 # built from `php55` branch [![Circle CI](https://circleci.com/gh/million12/docker-php-testing/tree/php55.svg?style=svg)](https://circleci.com/gh/million12/docker-php-testing/tree/php55)


## Usage

Here is an example how you can run your unit, functional and Behat test. In the example we are running TYPO3 Neos tests: unit, functional and Behat altogether.

First, launch containers with TYPO3 Neos (we use [million12/typo3-neos](https://github.com/million12/docker-typo3-neos) image for that):  
```
docker run -d --name=db --env="MARIADB_PASS=my-pass" million12/mariadb
docker run -d --name=neos -p=12345:80 --link=db:db \
    --env="T3APP_DO_INIT_TESTS=true" \
    --env="T3APP_VHOST_NAMES=neos dev.neos behat.dev.neos" \
    million12/typo3-neos
```

Now, having your application running in `neos` container, application data in /data/www/neos, here's how you can run tests against it:
```
docker run -ti --volumes-from=neos --link=neos:web --link=db:db -p=4444:4444 -p=5900:5900 million12/php-testing "
    echo \$WEB_PORT_80_TCP_ADDR \$WEB_ENV_T3APP_VHOST_NAMES >> /etc/hosts && cat /etc/hosts && \
    su www -c \"
        cd /data/www/typo3-app && \
        echo -e '\n\n======== RUNNING TYPO3 NEOS TESTS =======\n\n' && \
        bin/phpunit -c Build/BuildEssentials/PhpUnit/UnitTests.xml && \
        bin/phpunit -c Build/BuildEssentials/PhpUnit/FunctionalTests.xml && \
        bin/behat -c Packages/Application/TYPO3.Neos/Tests/Behavior/behat.yml
    \"
"
```

Have a look at [million12/typo3-neos](https://github.com/million12/docker-typo3-neos) repository for a complete example. Tests there are described in [circle.yml](https://github.com/million12/docker-typo3-neos/blob/master/circle.yml) and are running on [CircleCI](https://circleci.com/gh/million12/docker-typo3-neos).

Note: port 4444 allows you to connect with the browser to Selenium server. Port 5900 allows to connect to VNC server (with VNC client) and see how the tests are executed in the Selenium browser.

## Authors

Author: Marcin Ryzycki (<marcin@m12.io>)  

---

**Sponsored by [Prototype Brewery](http://prototypebrewery.io/)** - the new prototyping tool for building highly-interactive prototypes of your website or web app. Built on top of [Neos CMS](https://www.neos.io/) and [Zurb Foundation](http://foundation.zurb.com/) framework.
