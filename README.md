# Docker with Selenium for Behat testing
[![Circle CI](https://circleci.com/gh/million12/docker-behat-selenium.png?style=badge)](https://circleci.com/gh/million12/docker-behat-selenium)

This is a [million12/behat-selenium](https://registry.hub.docker.com/u/million12/behat-selenium/) container for running PHP tests using Behat. Selenium server is installed and running, also there is a VNC server so you can connect to it to inspect the browser while tests are running.

This container is based on container with PHP, [million12/php-app](https://github.com/million12/docker-php-app). If you use the same one for you application (and you should), you have exactly the same environment for the application and your test. That gives you consistent results and guarantee that if your test are passing, your app is working.

Note: this container does not actually contain Behat installed. Presumably it's available in your application's directory (and some specific version of it). Same applies for phpunit. If that's not the case, there's a composer tool so you can easily install it.

## Usage

Here is an example how you can run your test. In the example we are running TYPO3 Neos tests, unit, functional and Behat altogether.

First, launch your complete application:  
```
docker run -d --name=db --env="MARIADB_PASS=my-pass" tutum/mariadb
docker run -d --name=neos -p=8080:80 --link=db:db \
    --env="NEOS_APP_DO_INIT=false" \
    --env="NEOS_APP_DO_INIT_TESTS=true" \
    --env="NEOS_APP_VHOST_NAMES=neos dev.neos behat.dev.neos" \
    million12/typo3-neos
```

Now, having your application running in `neos` container, application data in /data/www/neos, here's how you can run tests against it:
```
docker run -ti --volumes-from=neos --link=neos:web --link=db:db -p=4444:4444 -p=5900:5900 million12/behat-selenium "
    echo \$WEB_PORT_80_TCP_ADDR \$WEB_ENV_NEOS_APP_VHOST_NAMES >> /etc/hosts && cat /etc/hosts && \
    su www -c \"
        cd /data/www/neos && \
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

**Sponsored by** [Typostrap.io - the new prototyping tool](http://typostrap.io/) for building highly-interactive prototypes of your website or web app. Built on top of TYPO3 Neos CMS and Zurb Foundation framework.
