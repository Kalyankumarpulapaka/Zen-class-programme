Zen Class Programme Database Queries
This README.md provides MongoDB queries with comments and explanations for the Zen Class Programme database.

Topics taught in October
javascript
Copy code
db.topics.find({
    "date": { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") }
}).pretty();
Explanation:

Retrieves topics taught in October from the topics collection based on the date field.
Tasks with due date in October
javascript
Copy code
db.tasks.find({
    "due_date": { $gte: ISODate("2020-10-01"), $lte: ISODate("2020-10-31") }
}).pretty();
Explanation:

Fetches tasks from the tasks collection that have a due date in October.
Find all the company drives which appeared between 15 Oct 2020 and 31 Oct 2020
javascript
Copy code
db.companyDrives.find({
    "drive_date": { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") }
}).pretty();
Explanation:

Retrieves company drives from the companyDrives collection that occurred between October 15th and October 31st, 2020.
Find all the company drives and students who appeared for the placement
javascript
Copy code
db.companyDrives.aggregate([
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
Explanation:

Aggregation pipeline to fetch details of company drives and students who attended from companyDrives and users collections respectively.
Find the number of problems solved by the user in codekata
javascript
Copy code
db.codekata.aggregate([
    {
        $group: {
            _id: "$user_id",
            total_problems_solved: { $sum: "$problems_solved" }
        }
    }
]).pretty();
Explanation:

Aggregation query to calculate total problems solved by each user in the codekata collection.
Find all the mentors who have more than 15 mentees
javascript
Copy code
db.mentors.find({
    "students_count": { $gt: 15 }
}).pretty();
Explanation:

Retrieves mentors from the mentors collection who have more than 15 mentees.
Find the number of users who were absent and did not submit tasks between 15 Oct 2020 and 31 Oct 2020
javascript
Copy code
db.attendance.aggregate([
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
Explanation:

Aggregation pipeline to count users who were absent and did not submit tasks between October 15th and October 31st, 2020.




