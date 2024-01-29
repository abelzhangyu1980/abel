# Proximity Service
a proximity service is used to discover nearby places i.e restaurants, hotels, theaters,museums..etc. input to the service wil be user's coordination.

## understand the problem
- search radius
- maximal radius allowed
- allow user to change radius
- business maintenance (add/update/delete...)
- NFRs: low latency, data privacy, high availability and scalability

## High level design
- search api
    /v1/search/nearby parameters: latitude, longitude, radius
- business api
    get /v1/businesses/1, post /v1/businesses/ put /v1/businesses/v1 delete /v1/businesses/1

## Data model
table design goes here

address, city, name, latitude, longtitude

### Geo index table
simple way: select  xxx from businss where latitude between ( %latitude% - radius and %latitude% + radius) and longtitude between ()
problem with this approach is database index can only improve search speed on one dimension. as database has to load all entries matching latitude and then another list matching longtitude. and then merge the two lists to get the result.

split map into smaller areas and then build indexes on the small areas. problem with this approach is businesses on the edge. also sparse and dense areas.--boundary issue.

or convert two dimension into one dimension. (code latitude and longtitude into a string .ie. latitude between -90-0, call 1. longtitude -90-0.1. then area in -90,-90, 0,0) will be coded as 11. we can go further to split area into smaller areas. like -90 to -45. 1 .. 1111...

#### geohash

#### quadtree










