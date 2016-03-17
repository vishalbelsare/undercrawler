A crawler
=========

A crawler to collect all components together:

* splash with HH scripts
* autologin + login keychain

TODO:

* autoregister
* crazy-form-submitter
* captcha solving
* pagination


Installation
------------

Requires Python 3.4:

    pip install -r requirements.txt
    python -c "import formasaurus; formasaurus.extract_forms('a')"

You also will need splash (but you can just use docker, see below),
and install autologin separately (https://github.com/TeamHG-Memex/autologin),
from the branch crawler-integration.

Run crawler
-----------

Start splash:

    docker run -p 8050:8050 scrapinghub/splash

Start autologin HTTP API with the ``autologin-http-api`` command,
and the UI server with ``autologin-server``.

Specify url to crawl via the ``url`` param, and run the ``base`` spider:

    scrapy crawl base -a url=http://127.0.0.1:8001

You can also specify a file to read urls from, with ``-a url=./urls.txt'``,
but in this case you must disable autologin with ``-s AUTOLOGIN_ENABLED=0``,
or ensure that all urls use common authentication.

Useful options to tweak (add to the above command via ``-s NAME=value``):

- ``ADBLOCK`` - set to 1 to enable AdBlock filters (they can make crawling faster)
- ``AUTOLOGIN_ENABLED`` - set to 0 to disable autologin middleware
- ``AUTOLOGIN_URL`` - url of the autologin HTTP API
- ``CDR_CRAWLER``, ``CDR_TEAM`` - CDR export metadata constants
- ``DOWNLOAD_DELAY`` - set to 0 when crawling local test server
- ``FORCE_TOR`` - crawl via tor to avoid blocking
- ``HARD_URL_CONSTRAINT`` - set to 1 to treat start urls as hard constraints
 (by default we start from given url but crawl the whole domain)
- ``MAX_DOMAIN_SEARCH_FORMS`` - max number of search forms considered for domain
- ``PREFER_PAGINATION`` - set to 0 to disable pagination handling
- ``RUN_HH`` - set to 0 to skip running full headless-horesman scripts
- ``SEARCH_TERMS_FILE`` - file with extra search terms to use (one on a line)
- ``SPLASH_URL`` - url of the splash instance

Data is stored in CDR format, we put some custom stuff into ``extracted_metadata``:

- ``is_page``: page was reached via pagination
- ``is_onclick``: page url was extracted from ``onclick``, not from a normal link
- ``is_iframe``: page url was extracted from an ``iframe``
- ``is_search``: this is a search result page
- ``from_search``: page was reached from search results
- ``depth``: page depth
- ``form``: forms metadata extracted by formasaurus

Use ``./scripts/crawl_stats.py`` to analyze extracted metadata.

Scripts
-------

* ``./scripts/crawl_stats.py``:
  show crawling stats, including ``extracted_metadata``
* ``./scripts/gen_supervisor_configs.py``:
  generate supervisord configs for crawlers from a list of urls
