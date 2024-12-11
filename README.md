1

    db.getCollection('movies').aggregate(
    [
        {
        "$lookup": {
            "from": "comments",
            "localField": "_id",
            "foreignField": "movie_id",
            "as": "comments"
        }
        },
        {
        "$project": {
            "_id": 0,
            "title": 1,
            "comments": {
            "$map": {
                "input": "$comments",
                "as": "comment",
                "in": {
                "name": "$$comment.name",
                "email": "$$comment.email",
                "text": "$$comment.text",
                "date": "$$comment.date"
                }
            }
            }
        }
        }
    ]
    )
 
2 

    db.movies.aggregate(
        [
            {
            "$lookup": {
                "from": "comments",
                "localField": "_id",
                "foreignField": "movie_id",
                "as": "comments"
            }
            },
            {
            "$project": {
                "_id": 0,
                "title": 1,
                "commentCount": {
                "$size": "$comments"
                }
            }
            }
        ]
    )

3

    db.movies.aggregate(
        [
            {
            "$lookup": {
                "from": "comments",
                "localField": "_id",
                "foreignField": "movie_id",
                "as": "comments"
            }
            },
            {
            "$project": {
                "_id": 0,
                "title": 1,
                "imdbRating": "$imdb.rating",
                "commentCount": {
                "$size": "$comments"
                }
            }
            },
            {
            "$sort": {
                "imdbRating": -1
            }
            },
            {
            "$limit": 5
            }
        ]
    )

4

    db.movies.aggregate(
        [
        { "$unwind": "$cast" },
        {
        "$group": {
            "_id": "$cast",
            "movieCount": { "$sum": 1 }
         }
        },
        {
        "$replaceRoot": {
            "newRoot": {
            "castMember": "$_id",
            "movieCount": "$movieCount"
            }
          }
        },
        { "$sort": { "movieCount": -1 } }
        ]
    )

5

    db.movies.aggregate(
        [
            {
            "$match": {
                "year": { "$lt": 1950 },
                "imdb.rating": { "$gte": 7.0 }
            }
            },
            {
            "$lookup": {
                "from": "comments",
                "localField": "_id",
                "foreignField": "movie_id",
                "pipeline": [{ "$sort": { "date": 1 } }, { "$limit": 2 }],
                "as": "comments"
            }
            },
            {
            "$project": {
                "_id": 0,
                "title": 1,
                "releaseYear": "$year",
                "genres": 1,
                "imdbRating": "$imdb.rating",
                "comments": {
                "$map": {
                    "input": "$comments",
                    "as": "comment",
                    "in": {
                    "name": "$$comment.name",
                    "text": "$$comment.text"
                        }
                      }
                    }
                }
            }
        ]
    )