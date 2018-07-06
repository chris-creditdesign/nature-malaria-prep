# Map and data preparation for enhanced Nature News Feature

## Format data

Excel files downloaded from [WWARN](http://www.wwarn.org/).

## Worldwide Malaria death reates (per 100,000) 

Download CSV file from [https://ourworldindata.org/malaria]()

## Create map shapefile 

Download Medium scale data 1:50m world countries shapefile from [Natural Earth Data](http://www.naturalearthdata.com/) 

Install shapefile

	npm intsall -g shapefile

Convert to geojson

	shp2json Natural\ Earth\ Data/50m_cultural/ne_50m_admin_0_countries.shp -o world/countries.json

Install NDJSON

	npm install -g ndjson-cli

Split up the geojson file so that each line ends with a `\n` character

	ndjson-split 'd.features' \
	  < world/countries.json \
	  > world/countries.ndjson

Add a `Code` key for each country, so that it can be used for the join and reduce the amount of properties for each country, to just the name
	
	ndjson-map 'd.Code = d.properties.SOV_A3, d' < world/countries.ndjson \
	| ndjson-map 'd.properties = { Name:d.properties.SOVEREIGNT }, d' \
	  > world/countries-reduced.ndjson

Turn the marlaria-death-rates.csv into a NDJSON file

	csv2json -n \
	  < world/malaria-death-rates-by-year.csv \
	  > world/malaria-death-rates-by-year.ndjson

Join the deaths data to the world map data. Make this a left outer join, so that each country is kept, even if they don't have any malaria data.

	ndjson-join --left 'd.Code' \
	  world/countries-reduced.ndjson \
	  world/malaria-death-rates-by-year.ndjson \
	  > world/countries-malaria-join.ndjson

This results in a file with two json objects per line. Add the second object as a 'deaths' to the first objects 'properties'
To get rid of the second object use ndjson-map and Object.assign

	ndjson-map 'd[0].properties.deaths = d[1], d' < world/countries-malaria-join.ndjson \
	| ndjson-map 'Object.assign(d[0])' \
	  > world/countries-malaria-deaths.ndjson


Convert back from ndjson

	ndjson-reduce  < world/countries-malaria-deaths.ndjson \
	| ndjson-map '{type: "FeatureCollection", features: d}' \
	  > world/countries-malaria.json

Convert from from geojson to topojson

	geo2topo world/countries-malaria.json > world/countries-malaria-topo.json

Simplify the topojson to reduce file size 

	toposimplify -p 0.01 -f < world/countries-malaria-topo.json > world/countries-malaria-topo-simple.json



























