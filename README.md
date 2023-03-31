# GymSQLDataBase
Completed gym data model with corresponding subqueries related to efficient management


# TEAM: 21482_8 and Team Members

- Claire Shur (https://github.com/claireshur/Mist4610-Gym)
-  Kole Piercy (https://github.com/piercyko/Gym-Database)
-  Jade Naughton (https://github.com/jnaug0829/Group-Project-4610)
-  Chris Richards (https://github.com/Chris-Richards3/MySQL-Project)
-  Jack Mannion (https://github.com/JackMannion01/GymSQLDataBase)

# Problem Description

Our project was focused on creating a model and database for a gym. We break down all the aspects and interactions that occur at an individual, well-operated commercial gym. Though there are always ways to improve a gym, we focused on our augmentation process on a growing trend, home workouts.

Our first entity we focused on implementing was a virtual session entity. There are countless personal and health limitations that don't allow people to physically go to the gym. These sessions are taught by trainers at the gym to allow people to still stay active and healthy from their home. Users can access these virtual sessions through an application. 

The gym's app includes features that benefit our customers and the gym as a whole. Previously mentioned, it allows members from home to watch these virtual sessions and perform them at home. The app also allows customers coming to pull up a barcode and check in quickly. Members can leave reviews too about their thoughts on the app which allows the gym to receive feedback and improve upon it.


# Data model

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Gym%20Data%20Model.png)

Explanation - There are many different relationships involved with the entities in a data model of a gym. One gym can have many employees, as well as many transactions. In regards to members, members can also have many transactions, can attend many personal sessions, and can attend many group sessions. Group sessions can have many members attend it. Employees that are trainers can work many personal sessions, group sessions, and virtual sessions. A supplier can have many pieces of equipment. Each class type can have many group sessions, and may use many different kinds of equipment. Each app can have many virtual sessions from this gym. Finally, each membership type will be chosen by many members. 

# Data Dictionary 

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/data%20dictionary.pdf)

# 10 Queries

1. List all the classes offered by the gym and the amount of members that have attended each class. List the results from most attendance to least attendance. 

Relevancy - Gym owners should know which classes are most popular and least popular, to determine which classes to add more options of or which might need to be taken away.

SELECT className, COUNT(Attendance.memID)
FROM ClassType
JOIN GroupSessions ON ClassType.classID=GroupSessions.classID
JOIN Attendance ON GroupSessions.groupID=Attendance.groupID
GROUP BY className
ORDER BY COUNT(Attendance.memID) DESC;

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/1.png)

2. List any members that have not participated in a personal session 

Relevancy - Shows which members to speak to next about trying out a personal session, don’t want to market to customers who already participate in personal sessions.

SELECT firstName, lastName
FROM Members
WHERE NOT EXISTS (SELECT memID FROM PersonalSessions WHERE PersonalSessions.memID=Members.memID);

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/2.png)

3. 3. Members who attended at least one group session in the past 3 years, and how many sessions they attended

Relevancy -  Showcases the usage of the various programs implemented into memberships. This information is useful to provide insight to see if these programs are valuable to members; knowing this information can ultimately increase satisfaction in the long run.

SELECT firstName, lastName, COUNT(attenID), dateTime
FROM Members
JOIN Attendance ON Members.memID=Attendance.memID
JOIN GroupSessions ON Attendance.groupID=GroupSessions.groupID
GROUP BY firstName, lastName, dateTime
HAVING dateTime BETWEEN '2020-03-30 12:00:00' AND '2023-03-30 12:00:00';

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/3.png)

4. Average time per session for Group Sessions and Personal Sessions

Relevancy - An estimate of how long to expect a group session to go on for as opposed to a personal session.

SELECT AVG(GroupSessions.duration) AS 'Group Sessions', AVG (PersonalSessions.duration) AS 'Personal Sessions'
FROM GroupSessions
JOIN Employees ON GroupSessions.employeeID = Employees.employeeID
JOIN PersonalSessions ON Employees.employeeID = PersonalSessions.employeeID;

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/4.png)

5. What Group Sessions are hard and over 80 minutes long

Relevancy - To find sessions for customers who want more of a challenge and have a specific amount of time that they want to exercise for. Can be adjusted for difficulty and time limit.

SELECT ClassType.classID, className
FROM ClassType
JOIN GroupSessions ON ClassType.classID = GroupSessions.classID
WHERE difficulty = 'Hard' AND duration > 80;

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/5.png)

6. Which supplier and its specialty has an average maintenance cost of their equipment that is greater than 2 times the average maintenance cost of all equipment

Relevancy - This information is useful to a gym owner to visualize costs and pinpoint which assets are detrimental and which ones need review. This information helps minimize waste, reduce cost, and modify business plans into a more effective, more efficient model. 

SELECT suppName, specialty, AVG(avgMaintCost) AS 'Avg Maint Cost'
FROM Supplier
JOIN Equipment ON Supplier.supplierID=Equipment.supplierID
GROUP BY suppName, Specialty
HAVING AVG(avgMaintCost)>2*(SELECT AVG(avgMaintCost) FROM Equipment);

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/6.png)

7. Percentage of employees that are used for personal sessions

Relevancy - Which employees are involved in conducting personal sessions versus which employees work other positions. Might help determine what type of employee to hire next.

SELECT CONCAT(ROUND((COUNT(PersonalSessions.employeeID)/(SELECT COUNT(Employees.employeeID) FROM Employees)*100),2),'%') AS '% Trainers in Personal Session'
FROM PersonalSessions
JOIN Employees ON PersonalSessions.employeeID=Employees.employeeID;

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/7.png)

8. Name and address of the member who has attended the most personal sessions, and which staff they trained with 

Relevancy - Gym owners would like to know who their most involved customers are, to keep them happy and make sure they spread the word about their gym.

SELECT Members.firstName, Members.lastName, address, Employees.firstName, Employees.lastName, COUNT(PersonalSessions.dateTime)
FROM Members
JOIN PersonalSessions ON Members.memID=PersonalSessions.memID
JOIN Employees ON PersonalSessions.employeeID=Employees.employeeID
GROUP BY Members.firstName, Members.lastName, address, Employees.firstName, Employees.lastName
ORDER BY COUNT(PersonalSessions.dateTime) DESC
LIMIT 1;

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/8.png)

9. Number of customers holding different membership types in descending order

Relevancy: This allows the managers to see the most popular membership options and make decisions based on the numerical distribution.

SELECT memTypeName, COUNT(memID)
FROM MembershipType
JOIN Members ON MembershipType.memTypeID=Members.memTypeID
GROUP BY memTypeName
ORDER BY COUNT(memID) DESC;

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/9.png)

10. What are the app names and platforms and how many virtual sessions it runs with a rating with 4 or higher

Relevancy: This allows the gym managers and trainers to figure out what apps are highly rated, and how many virtual sessions are actually being used by this app. This can be eye opening with how many virtual sessions offered and only very few of them are on highly rated apps, so changes probably need to be made.

SELECT appName, platform, COUNT(virtualID) as 'Virtual Session Ids', rating
FROM Apps
JOIN VirtualSessions ON Apps.appID = VirtualSessions.appID
WHERE rating REGEXP('^4')
GROUP BY Apps.appID;

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/Query%20Response/10.png)

## Query Matrix 

![alt text](https://github.com/JackMannion01/GymSQLDataBase/blob/main/QueryMatrix.PNG)

# DataBase Information

ns_21482_8
