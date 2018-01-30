# MongoDB datasource for Grafana

## Features
Allows MongoDB to be used as a data source for Grafana by providing a proxy to convert the Grafana Data source [API](http://docs.grafana.org/plugins/developing/datasources/) into MongoDB aggregation queries

## Requirements

* **Grafana** > 3.x.x
* **MongoDB** > 3.4.x

## Installation

### Install the Grafana plugin components

* Copy the whole mongodb-grafana dir into the Grafana plugins dir ( /usr/local/var/lib/grafana/plugins )
* Restart the Grafana server. If installed via Homebrew, this will be `brew services restart grafana`

### Install Start the MongoDB proxy server

* Open a command prompt in the mongodb-grafana directory
* Run `npm install` to install the node.js dependencies
* Run `npm server` to start the REST API proxy to MongoDB. By default, the server listens on http://localhost:3333

## Examples

Create a new data source of type MongoDB as shown below. The MongoDB details are :

* **MongoDB URL** - `mongodb://rpiread:rpiread@rpi-sensorreadings-shard-00-00-fgscn.mongodb.net:27017,rpi-sensorreadings-shard-00-01-fgscn.mongodb.net:27017,rpi-sensorreadings-shard-00-02-fgscn.mongodb.net:27017/test?ssl=true&replicaSet=RPI-SensorReadings-shard-0&authSource=admin`
* **MongoDB Database** - `rpi`



<img src="src/img/sample_datasource.png" alt="Sample Data Source" style="width: 500px;"/>

Save the data source, then import the dashboard in `examples\RPI MongoDB - Atlas.json`

This should show a graph of light sensor values from a Raspberry PI with an [EnviroPHAT](https://thepihut.com/products/enviro-phat) board feeding readings every minute into a MongoDB Atlas database.

<img src="src/img/sample_dashboard.png" alt="Sample Dashboard" style="width: 800px;"/>

Clicking on the title of the graph allows you to see the aggregation query being run against the 'RPI Atlas' data source

<img src="src/img/sample_query.png" alt="Sample Query" style="width: 800px;"/>

The query here is

```javascript
db.sensor_value.aggregate ( [ 
{ "$match" :    {   "sensor_type" : "$sensor",   "host_name" : "$host",   "ts" : { "$gte" : "$from", "$lte" : "$to" } }  },        
 {"$sort" : {"ts" : 1}},            
 {"$project" :   {  "name" : "value",   "value" : "$sensor_value",  "ts" : "$ts_epoch",  "_id" : 0} } ])
 ```

 The API is expecting back documents with the following fields

 * `name` - Name of the series ( will be displayed on the graph)
 * `value` - The float value of the point
 * `ts` - The epoch time of the data point

 These documents are then converted into the [Grafana API](http://docs.grafana.org/plugins/developing/datasources/)

`$from` and `$to` are expanded by the plugin as ISODates based on the range settings on the UI.

`$sensor` and `$host` are template variables that are filled in by Grafana based on the drop down. The sample template queries are shown below. They expect documents to be returned with a single `_id` field.

 
<img src="src/img/sample_template.png" alt="Sample Templates" style="width: 800px;"/>
 







