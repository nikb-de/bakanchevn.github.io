# Nulls handling 
We all know that NULL is a special value (pointer) in SQL. It is used to represent missing or unknown values.
To work with NULLs we have to use special operators and functions. When we use it just by filtering we use IS NULL and IS NOT NULL operators or COALESCE functions. Usually, it is a bit annoying but we handle it.

But there can be situations when we don't expect it and it can cause some problems.
Here is the story begins. 

## The problem. 

We had a simple request from the business to get daily updates about the number of new users on the platform.
We designed a table with the following structure:

| Column | Type | Description |
| --- | --- | --- |
| customer_id | int | Unique identifier of the customer |
| created_at | datetime | Date and time when the customer was created |
| customer_name | varchar(255) | Name of the customer |
| customer_status | varchar(255) | Status of the customer |

The data had been coming from the microservice via Kafka, which was responsible for customer record creation.
So on daily basis, we could get the number of new customers.
The request was quite simple: 

```sql  
SELECT COUNT(*) 
  FROM customers 
 WHERE created_at >= DATEADD(DAY, -1, GETDATE())
```

It worked well for a while. But the business grew up and we got a new requirement. Filter out the VIP and Enterprise customers. This query was for the Marketing Department and they didn't work with VIP customers. So we had to filter out the customers with the following statuses:

```sql
SELECT COUNT(*) 
  FROM customers 
 WHERE created_at >= DATEADD(DAY, -1, GETDATE()) 
   AND customer_status NOT IN ('VIP', 'Enterprise')
```

After a while, we got a new requirement. The VIP department can enrich customer types with new statuses. With the current query, we would have to change hardcoded statuses every time we received a new one. 

That looked like a good candidate to change the approach. We needed to have the ability to filter out customers with statuses that should be excluded. 
The idea was the following: 

> Implement the table with the statuses that should be excluded. This table can be updated by the business. 
