# Map and data preparation for enhanced Nature News Feature

## Format data

Excel files downloaded from [WWARN](http://www.wwarn.org/).

## Create map shapefile 

Download world countries shapefile from [Natural Earth Data](http://www.naturalearthdata.com/) and extract a file containing just Laos, Cambodia, Myanmar, Thailand and Vietnam. 

Convert to geojson

	npm intsall -g shapefile
	shp2json map/malaria-countries.shp -o map/malaria-countries.json

Convert from from geojson to topojson

	geo2topo map/malaria-countries.json > map/malaria-countries-topo.json

Simplify the topojson to reduce file size 

	toposimplify -p 1 -f < map/malaria-countries-topo.json > map/malaria-countries-topo-simple.json




////



Apply the mercator projection and resize to 960 by 960px

	geoproject 'd3.geoMercator().rotate([120,0]).fitSize([960,960], d)' < map/malaria-countries.json > map/malaria-mercator.json

Test the result with an svg

	geo2svg -w 906 -h 960 < map/malaria-mercator.json > map/malaria-mercator.svg

![](map/malaria-mercator.svg)

Install topojson cli

	npm install -g topojson

Convert from from geojson to topojson

	geo2topo map/malaria-mercator.json > map/malaria-topo.json

Simplify the topojson to reduce file size

	toposimplify -p 1 -f < map/malaria-topo.json > map/malaria-simple-topo.json
