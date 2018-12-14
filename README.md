# about this repo

collect data using SNMP with telegraf.  
store collected data in influxdb  
query influxdb database with cli and python to extract data   

# about telegraf

telegraf is an open source collector written in GO.  
Telegraf collects data and writes them into a database.  
It is plugin-driven (it has input plugins, output plugins, ...)  

# requirements 

## docker

you need to install docker.  
This is not covered in this repository

## Junos 

here's some details from the junos devices   
```
jcluser@vMX-addr-0> show configuration snmp
community public;
```
# influxdb

pull docker images 
```
$ docker pull influxdb
```
Verify
```
$ docker images influxdb
```
Instanciate an influxdb container
```
$ docker run -d --name influxdb -p 8083:8083 -p 8086:8086 influxdb
```
Verify
```
$ docker ps | grep influxdb
```
for troubleshooting purposes you can run this command
```
$ docker logs influxdb
```
start a shell session in the influxdb container
```
$ docker exec -it influxdb bash
```
run this command to read the influxdb configuration file
```
# more /etc/influxdb/influxdb.conf
```
create a user and a database
```
# influx
Connected to http://localhost:8086 version 1.7.0
InfluxDB shell version: 1.7.0
Enter an InfluxQL query
> CREATE DATABASE juniper
> show databases
name: databases
name
----
_internal
juniper
>  CREATE USER "juniper" WITH PASSWORD 'juniper'
> show users
user   admin
----   -----
influx false
> exit
# 
```
exit the influxdb container
```
# exit
```

# telegraf

get ip address used by containers
```
$ ifconfig docker0
```

pull docker images 
```
$ docker pull telegraf
```
Verify
```
$ docker images telegraf
```
create a telegraf configuration file ([use this file](telegraf.conf))  
it will use SNMP and influxb output plugin (database to store the data collected)  

```
$ vi telegraf.conf
```
instanciate a telegraf container
```
$ docker run --name telegraf -d -v $PWD/telegraf.conf:/etc/telegraf/telegraf.conf:ro telegraf
```
verify
```
$ docker ps | grep telegraf
```
for troubleshooting purposes you can run this command
```
$ docker logs telegraf
```
start a shell session in the telegraf container
```
$ docker exec -it telegraf bash
```
verify the telegraf configuration file
```
# more /etc/telegraf/telegraf.conf
```
exit the telegraf container
```
# exit
```
# query the influxdb database with cli to verify
start a shell session in the influxdb container
```
$ docker exec -it influxdb bash
```
query the database
```
# influx
Connected to http://localhost:8086 version 1.7.0
InfluxDB shell version: 1.7.0
Enter an InfluxQL query
> show databases
name: databases
name
----
_internal
juniper
> use juniper
Using database juniper
> show measurements
name: measurements
name
----
```
```
> select * from "/interfaces/" order by desc limit 1
```
```
> select * from "/network-instances/network-instance/protocols/protocol/bgp/" order by desc limit 1
```
exit 
```
> exit
#
```
exit the influxdb container
```
# exit 
```
# query the influxdb database with python to verify
install the influxdb python library.
This python library is a client for interacting with InfluxDB.
```
$ pip install influxdb
```
Verify
```
$ pip list | grep influx
```
you can now interact with InfluxDB using Python

run this command to start a python interactive session

```
$ python
```
connect to InfluxDB using Python
```
>>> from influxdb import InfluxDBClient
>>> influx_client = InfluxDBClient('localhost',8086)
```
list the databases
```
>>> influx_client.query('show databases')
ResultSet({'(u'databases', None)': [{u'name': u'_internal'}, {u'name': u'juniper'}]})
```
list measurements for a database
```
>>> influx_client.query('show measurements', database='juniper')
ResultSet({'(u'measurements', None)': [{u'name': u'/interfaces/'}, {u'name': u'/network-instances/network-instance/protocols/protocol/bgp/'}]})
```
query data from a particular measurement and database
```
>>> gp = influx_client.query('select * from "/interfaces/"  order by desc limit 2 ', database='juniper').get_points()
>>> for item in gp:
...     print item['/interfaces/interface/@name']
...
ge-0/0/7
ge-0/0/6
>>> gp = influx_client.query('select * from "/interfaces/"  order by desc limit 2 ', database='juniper').get_points()
>>> for item in gp:
...     print item['/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets']
...
36
26277618
>>>
