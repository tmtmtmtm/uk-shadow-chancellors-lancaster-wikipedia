Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

Item looks fine.

Step 2: Tracking page
=====================

PositionHolderHistory did not exist yet. I created it as:
https://www.wikidata.org/w/index.php?title=Talk:Q25906187&oldid=1259588167

It initially has with 6 dated memberships, 0 undated; and 6 warnigs.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js)
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

```sh
wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
  xargs wd sparql existing-p39s.js | tee wikidata.json
```

Step 5: Scrape
==============

```sh
wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
  xargs bundle exec ruby scraper.rb | tee wikipedia.csv
```

Step 6: Create missing P39s
===========================

```sh
bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
  wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"
```

7 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/a9198f552b45f

Step 7: Add missing qualifiers
==============================

```sh
bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
  wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"
```

nothing to do

Step 8: Refresh the Tracking Page
==================================

New version at
https://www.wikidata.org/w/index.php?title=Talk:Q25906187&oldid=1259616252
