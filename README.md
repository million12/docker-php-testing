

```
docker run -d --volumes-from=typo3neos_neosWeb_1 --link=typo3neos_neosWeb_1:web --link=typo3neos_neosDb_1:db --rm million12/behat-selenium "
    env; \
    echo \$WEB_PORT_80_TCP_ADDR \$WEB_ENV_NEOS_APP_VHOST_NAMES >> /etc/hosts; \
    cat /etc/hosts; \
    su www -c \"
        cd && cd neos && \
        echo && echo '======== RUNNING TYPO3 NEOS TESTS =======' && echo && \
        bin/behat -c Packages/Application/TYPO3.Neos/Tests/Behavior/behat.yml
    \"
"
```
