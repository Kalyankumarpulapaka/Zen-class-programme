# Zen Class Programme MongoDB Queries

This repository contains MongoDB queries for the Zen Class Programme database. Each query is accompanied by an explanation.

## Queries

### Topics taught in October

```javascript
db.topics.find({
    "date": { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") }
}).pretty();
