# Data Analysis with MongoDB


## Aggregation Framework Operators

Each operator execute operation and send result for next step/operator.
```javascript
db.collection.aggregate([query]);
```

- `$project` used in pipeline to extraction fields and create new document for next steps.
- `$match` used to filter documents.
- `$group` used to group documents by fields.
- `$sort` used to sort documents, 1 is asc and -1 is desc.
- `$skip` used to skip documents.
- `$limit` used to limit results for next steps.
- `$unwind` used to extract values in array and create new documents.

### Query examples

##### Top user with more ratio of friends x followers
```json
[{
  "$match":{
    "user.friends_count":{"$gt":0},
    "user.followers_count":{"$gt":0}
  }
},
{
  "$project": {
    "ratio": {
      "$divide":["$user.followers_count", "$user.friends_count"]
    },
    "screen_name": "$user.screen_name"
  }
}, 
{
  "$sort": {"ratio": -1}
},
{
  "$limit": 1
}]
```

##### Top user with more followers
```json
[{
    "$match": { 
      "user.time_zone": "Brasilia", 
      "user.statuses_count": { "$gte": 100 } 
    }
},
{
    "$project":{
        "followers": "$user.followers_count",
        "screen_name": "$user.screen_name",
        "tweets": "$user.statuses_count"
    }
},
{
    "$sort": { "followers": -1 }
},
{
    "$limit": 1
}]
``` 

##### Using $unwind operator to get top users with more user mentions in our tweets
```json
[{
    "$unwind": "$entities.user_mentions"
},
{
    "$group": {
        "_id": "$user.screen_name",
        "count": { "$sum": 1 }
    }
},
{
    "$sort": { "count": -1 }
}]
```

##### Using avg operator
```json
[{
    "$unwind": "$entities.hashtags"
},
{
    "$group": {
        "_id": "$entities.hashtags.text",
        "retweet_avg": { "$avg": "$retweet_count" }
    }
},
{
    "$sort": { "retweet_avg": -1 }
}]
```

##### Using $addToSet (add unique items from array, for all using $push)
```json
[{
    "$unwind": "$entities.hashtags"
},
{
    "$group": {
        "_id": "$user.screen_name",
        "unique_hashtags": { "$addToSet": "$entities.hashtags.text" }
    }
}]
```