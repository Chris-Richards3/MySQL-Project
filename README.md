# Table of Contents
* [Team Members](#Team-Members)

* [Project Description](#Project-Description)

* [Database Model](#Database-Model)

* [10 Complex Queries](#10-Complex-Queries)

    * [Query Matrix](#Query-Matrix)
  
* [Data Dictionary](#Data-Dictionary)

# Team Members

-  Chris Richards (https://github.com/Chris-Richards3/MySQL-Project)
- Claire Shur (https://github.com/claireshur/Mist4610-Gym)
-  Kole Piercy (https://github.com/piercyko/Gym-Database)
-  Jade Naughton (https://github.com/jnaug0829/Group-Project-4610)
-  Jack Mannion (https://github.com/JackMannion01/GymSQLDataBase)

# Project Description

Our project was focused on creating a well-formed database for a gym with a high degree of fidelity. We started the project by breaking down all the aspects and interactions that occur at a single well-operated gym. Our completed database adheres by the First, Second, & Third Normal Form and incorporates several business functions in a gym including Member Information, Memberships, Financial Information, Equipment Information, and all the various services the gym offers (Personal Sessions, Group Sessions, & even Virtual Sessions). The final database is compromsied of 13 different tables that are all linked together with one-to-many identifying & non-identifying relationships. Additionally, all the tables were populated with information using MySQL Workbench, Microsoft Excel, and Mockaroo. 

Our group was also tasked with augmenting our business of choice. Though there are always ways to improve a gym, we focused on our augmentation on a growing trend, home workouts. There are countless personal & health limitations that prevent people from physically going to a gym which justified the need for a new service: Virtual Training Sessions. In order to incorporate our augmentation, our group first created a many-to-many relationship between Employees and Apps with Virtual Sessions as an associative entity that would track pertinent information relating to these sessions. The use of virtual sessions gave our gym a new source of income and we believe it would help retain customer memberships. 

The most challenging deliverable our group completed was creating 10 Complex Queries that utilized high-level MySQL functions & concepts such as multi-table JOINS, correlated subqueries, EXISTS, NOT EXISTS, REGEXP, and more. Each of the queries we created were constructed with the goal of generating pertinent business value to the gym owner. 


# Database Model

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Gym%20Data%20Model.png)

Explanation - There are many different relationships involved with the entities in a data model of a gym. One gym can have many employees, as well as many transactions. In regards to members, members can also have many transactions, can attend many personal sessions, and can attend many group sessions. Group sessions can have many members attend it. Employees that are trainers can work many personal sessions, group sessions, and virtual sessions. A supplier can have many pieces of equipment. Each class type can have many group sessions, and may use many different kinds of equipment. Each app can have many virtual sessions from this gym. Finally, each membership type will be chosen by many members. 


# 10 Complex Queries

**1. List all the classes offered by the gym and the amount of members that have attended each class. List the results from most attendance to least attendance.** 

```
SELECT className, COUNT(Attendance.memID)
FROM ClassType
JOIN GroupSessions ON ClassType.classID=GroupSessions.classID
JOIN Attendance ON GroupSessions.groupID=Attendance.groupID
GROUP BY className
ORDER BY COUNT(Attendance.memID) DESC;
```

Relevancy - Gym owners should know which classes are most popular and least popular, to determine which classes to add more options of or which might need to be taken away.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/1.png)

**2. List any members that have not participated in a personal session.** 

```
SELECT firstName, lastName
FROM Members
WHERE NOT EXISTS (SELECT memID FROM PersonalSessions WHERE PersonalSessions.memID=Members.memID);
```

Relevancy - Shows which members to speak to next about trying out a personal session, don’t want to market to customers who already participate in personal sessions.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/2.png)

**3. List any members who have attended at least one group session in the past 3 years, and also list how many sessions they have attended.**

```
SELECT firstName, lastName, COUNT(attenID), dateTime
FROM Members
JOIN Attendance ON Members.memID=Attendance.memID
JOIN GroupSessions ON Attendance.groupID=GroupSessions.groupID
GROUP BY firstName, lastName, dateTime
HAVING dateTime BETWEEN '2020-03-30 12:00:00' AND '2023-03-30 12:00:00';
```

Relevancy -  Showcases the usage of the various programs implemented into memberships. This information is useful to provide insight to see if these programs are valuable to members; knowing this information can ultimately increase satisfaction in the long run.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/3.png)

**4. Calculate the average time of all sessions for Group Sessions and Personal Sessions.**

```
SELECT AVG(GroupSessions.duration) AS 'Group Sessions', AVG (PersonalSessions.duration) AS 'Personal Sessions'
FROM GroupSessions
JOIN Employees ON GroupSessions.employeeID = Employees.employeeID
JOIN PersonalSessions ON Employees.employeeID = PersonalSessions.employeeID;
```

Relevancy - An estimate of how long to expect a group session to go on for as opposed to a personal session.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/4.png)

**5. List all Group Sessions that are "hard" and were longer than 80 minutes.**

```
SELECT ClassType.classID, className
FROM ClassType
JOIN GroupSessions ON ClassType.classID = GroupSessions.classID
WHERE difficulty = 'Hard' AND duration > 80;
```

Relevancy - To find sessions for customers who want more of a challenge and have a specific amount of time that they want to exercise for. Can be adjusted for difficulty and time limit.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/5.png)

**6. List the supplier name and supplier specialty that has an average maintenance cost of their equipment that is greater than 2 times the average maintenance cost of all equipment.**

```
SELECT suppName, specialty, AVG(avgMaintCost) AS 'Avg Maint Cost'
FROM Supplier
JOIN Equipment ON Supplier.supplierID=Equipment.supplierID
GROUP BY suppName, Specialty
HAVING AVG(avgMaintCost)>2*(SELECT AVG(avgMaintCost) FROM Equipment);
```

Relevancy - This information is useful to a gym owner to visualize costs and pinpoint which assets are detrimental and which ones need review. This information helps minimize waste, reduce cost, and modify business plans into a more effective, more efficient model. 

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/6.png)

**7. Calculate the percentage of employees that have been the trainer for at least one personal session.**

```
SELECT CONCAT(ROUND((COUNT(PersonalSessions.employeeID)/(SELECT COUNT(Employees.employeeID) FROM Employees)*100),2),'%') AS '% Trainers in Personal Session'
FROM PersonalSessions
JOIN Employees ON PersonalSessions.employeeID=Employees.employeeID;
```

Relevancy - Which employees are involved in conducting personal sessions versus which employees work other positions. Might help determine what type of employee to hire next.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/7.png)

**8. List the name and address of the individual member who has attended the most personal sessions, and which employee(s) they have trained with.** 

```
SELECT Members.firstName, Members.lastName, address, Employees.firstName, Employees.lastName, COUNT(PersonalSessions.dateTime)
FROM Members
JOIN PersonalSessions ON Members.memID=PersonalSessions.memID
JOIN Employees ON PersonalSessions.employeeID=Employees.employeeID
GROUP BY Members.firstName, Members.lastName, address, Employees.firstName, Employees.lastName
ORDER BY COUNT(PersonalSessions.dateTime) DESC
LIMIT 1;
```

Relevancy - Gym owners would like to know who their most involved customers are, to keep them happy and make sure they spread the word about their gym.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/8.png)

**9. List the number of customers holding different membership types in descending order.**

```
SELECT memTypeName, COUNT(memID)
FROM MembershipType
JOIN Members ON MembershipType.memTypeID=Members.memTypeID
GROUP BY memTypeName
ORDER BY COUNT(memID) DESC;
```

Relevancy: This allows the managers to see the most popular membership options and make decisions based on the numerical distribution.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/9.png)

**10. List the app name(s), platforms and the amount of virtual sessions it has been connected to. Only list Apps with a rating with 4.0 or greater.**

```
SELECT appName, platform, COUNT(virtualID) as 'Virtual Session Ids', rating
FROM Apps
JOIN VirtualSessions ON Apps.appID = VirtualSessions.appID
WHERE rating REGEXP('^4')
GROUP BY Apps.appID;
```

Relevancy: This allows the gym managers and trainers to figure out what apps are highly rated, and how many virtual sessions are actually being used by this app. This can be eye opening with how many virtual sessions offered and only very few of them are on highly rated apps, so changes probably need to be made.

*Output:*

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/10.png)

## Query Matrix 

![alt text](https://github.com/Chris-Richards3/MySQL-Project/blob/main/Query%20Responses/QueryMatrix.PNG)

# Data Dictionary 

[File](https://github.com/Chris-Richards3/MySQL-Project/blob/main/data%20dictionary.pdf)

