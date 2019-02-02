---
title: Learning Mongodb
desc: Learning NoSQL for the first time.
published: true
date: 2019-01-14
categories: project
tags: intern mongodb database
---

This is a simple wrap up of introduction to MongoDB. I didn't have the chance to work with it unfortunately after I 

sudo service mongodb start
sudo systemctl status mongodb

To use MongoDB run the following command.
mongo

To get a list of commands, type db.help() in MongoDB client. This will give you a list of commands as shown in the following screenshot.

To get stats about MongoDB server, type the command db.stats() in MongoDB client. This will show the database name, number of collection and documents in the database. Output of the command is shown in the following screenshot.

Creating a database using “use” command
 use employee

Creating a Collection/Table using insert()
``` 
var myEmployee=
	[
	
		{
			"Employeeid" : 1,
			"EmployeeName" : "Smith"
		},
		{
			"Employeeid"   : 2,
			"EmployeeName" : "Mohan"
		},
		{
			"Employeeid"   : 3,
			"EmployeeName" : "Joe"
		},

	];

	db.Employee.insert(myEmployee);

db.Employee.find().forEach(printjson)
```