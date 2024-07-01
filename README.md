# Zen Class Programme MongoDB Queries

This repository contains MongoDB queries for the Zen Class Programme database.

## Queries

### 1.Topics taught in October

`db.topics.find({
    "date": { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") }
}).pretty();`

### 2.Find all the company drives which appeared between 15 Oct 2020 and 31 Oct 2020

`db.companyDrives.find({
    "drive_date": { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") }
}).pretty();
`

### 3.Find all the company drives and students who appeared for the placement

`db.companyDrives.aggregate([
    {
        $lookup: {
            from: "users",
            localField: "students_attended.user_id",
            foreignField: "_id",
            as: "students"
        }
    },
    {
        $project: {
            "company_name": 1,
            "drive_date": 1,
            "students": {
                $filter: {
                    input: "$students",
                    as: "student",
                    cond: { $eq: ["$$student._id", "$students_attended.user_id"] }
                }
            }
        }
    }
]).pretty();
`

### 4.Find the number of problems solved by the user in codekata

`db.codekata.aggregate([
    {
        $group: {
            _id: "$user_id",
            total_problems_solved: { $sum: "$problems_solved" }
        }
    }
]).pretty();
`

### 5.Find all the mentors who have more than 15 mentees

`db.mentors.find({
    "students_count": { $gt: 15 }
}).pretty();
`

### 6.Find the number of users who were absent and did not submit tasks between 15 Oct 2020 and 31 Oct 2020

`db.attendance.aggregate([
    {
        $lookup: {
            from: "tasks",
            let: { user_id: "$user_id" },
            pipeline: [
                {
                    $match: {
                        $expr: {
                            $and: [
                                { $eq: ["$user_id", "$$user_id"] },
                                { $not: { $eq: ["$submitted", true] } },
                                { $gte: ["$due_date", ISODate("2020-10-15")] },
                                { $lte: ["$due_date", ISODate("2020-10-31")] }
                            ]
                        }
                    }
                }
            ],
            as: "tasks_not_submitted"
        }
    },
    {
        $match: {
            "present": false,
            "tasks_not_submitted": { $exists: true, $ne: [] }
        }
    },
    {
        $count: "users_count"
    }
]);
`
