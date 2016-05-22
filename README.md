# This ixp-country-jedi integrates traIXroute

Follow the same directions until the get-measurements step. This version will
create a traixroute.json file inside the working directory, that will have to
be used as input for traIXroute. You can get a ixp-country-jedi-able traIXroute
version at:

[https://github.com/slowr/traIXroute](https://github.com/slowr/traIXroute)

follow the instructions in its README.md

After traIXroute completes, continue with the original ixp-country-jedi steps.





Probe mesh measurements

This codebase contains a couple of scripts to make probe mesh
measurements feasible/easy.

The way the scripts are used/tested is first create a directory for
your measurement campaign, and run the scripts

    mkdir 'SE-2015-03'
    cd SE-2015-03 
    <create config.json file here>
    <run scripts relative to current directory, ie. ../prepare.py ; 
      ../measure.py ...>

Scripts depend on this convention.

Dependencies on external python modules are in 'requirements.txt',
so you can run

    pip install -r requirements.txt

to fulfill these.

Before you use the scripts, create a config file specifying what
mesh you'd want to do.

config.json
===========

This file contains base data used for creating measurements and
analysing the results.  It currently contains 3 parts:

    *country*:   ISO 2 letter code for the country under analysis,
                 or list of countries under analysis.

    *ixps*:      List of IXPs one wants to detect/report on

    *locations*: List of cities on which probe selection will be
                 based.

An example is provided in the *example* directory. Only the *country* is mandatory. 
Without a *locations* section, the capital of *country* (or first country if *country*
is a list) is used for probe selection (see below).

One can set an extra 'location-constraint' config key. The value of this
configuration directive needs to be an integer. When this is set, the probe
selection part of the ixp-country-jedi will only select probes that are within
'location-constraint' kilometers from any of the given locations.
This can be useful if you only want to measure a specific city. In that case
set the 'location'-list to that specific city, and set a 'location-constraint'
to a reasonable value (for instance 50, so you'll cover 50 kilometers from the
city centre).


prepare.py
==========

This script reads config.json and spits out 2 other files basedata.json
and probeset.json

For each of the ixps listed in config.json it will try to find the
member ASNs

Based on the country and locations specified it will do a probe
selection:

    - only select public probes (because we want to measure to their
      public IP address)


    - per ASN will select up to 2 times the number of locations
      specified

If an ASN hosts more probes, the selection will be a maximum of 2x
the number of locations in config.json: the closest as well as the
furthest probe from each location will be selected.  Note that this
can result in less then 2 times the number-of-locations probes per
ASN even when there are more probes available in that ASN, because
a single probe could be the furthest away from multiple locations.

In the preparation/datagathering phase this script uses the GeoNames
service for geocoding.  Please put a valid geonames username in
~/.geonames/auth , for more info on GeoNames see
http://www.geonames.org/export/web-services.html

measure.py
==========

This script does one-off measurements for the probes specified in
probeset.json and stores the results of that in measurementset.json

This uses the RIPE Atlas measurement API for measurement creation,
and it needs a valid measurement creation API key in ~/.atlas/auth
. For more information on RIPE Atlas API keys see
https://atlas.ripe.net/docs/keys/

get-ips.py
==========

This script gathers metadata for all IPs in the collected data.
This is done separately from the rest of analysis-code because it
is time-consuming and ideally is done pretty soon after making all
measurements. If done too soon, not all measurement results are in
yet, if done much later (think days) meta-data, like reverse hostname
mapping might have changed, so ballpark is don't get-ips a few
minutes after measure.py. This step will create an 'ips.json-fragments'
file with reverse DNS lookups, ASNs and geoloc (via OpenIPMap, not
MaxMind) of IPs encountered in traceroutes.

get-measurements.py
===================

This script fetches measurements (from measurementset.json) and
does some initial analyses on them using information from config.json,
basedata.json, probeset.json and ips.json-fragments.  It creates a
local *results* directory and outputs a single json file per
measurement (analysed.<msm_id>.json) which is a list of analysis
results, one result per src/dst combination.

analyse-results.py
==================

This script produces text and/or webpages with analysis and visualisation in a
local *analysis* directory. For webpages these need to be on an actual
webserver for some of the javascript in them to work. One can easily create a
local webserver that would work for this purpose like this: 

    cd analysis ; python -m SimpleHTTPServer 3333

and then pointing your browser at localhost:3333/<viz-name> Note
that some visualisations use libraries in a common directory located
in the 'analysis' directory, so the webserver needs to run with the
'analysis'-directory (or lower) as root.

Templates (HTML,javascript,CSS) for webpages are in the 'templates'
directory and copied over each time the analyse-results.py script
is run, If you want to tweak the visualisations, tweaking them in the
'templates' directory and then running the analyse-results.py script
again will probably do what you want.

