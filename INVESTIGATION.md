# Murder Mystery Solved

A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was
a murder that occurred sometime on January 15 2018 and that it took place in SQL City. All the clues to this mystery are buried in a huge databse, and you need to use SQL 
to navigate through this vast network of information. Your first step to solving the mystery to retrieve the corresponding crime scene report frim the police department's 
database. Take a look at the cheatsheet to learn how to do this! From there, you can use your SQL skills to find the murderer.

Step 1: Find the crime scene report using the date, place and crime type to filter the crimescene report datbase

QUERY:
SELECT * FROM crime_scene_report
WHERE type = 'murder' AND city = 'SQL City' AND date = '20180115';

|   date   |  type  |                                                                                      description                                                                                         |   city   |
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".| SQL City |	

Deduction:
We need to find the witness reports using the descriptions and identifying them through the person database

Step 2: Find the witnesses and query their interviews 

QUERY:
SELECT * FROM interview
WHERE Person_id IN 
(SELECT id FROM person
WHERE address_street_name = 'Franklin Ave' AND name like 'Annabel%')
OR person_id IN 
(SELECT id FROM person 
 WHERE address_street_name = 'Northwestern Dr'
 ORDER BY address_number DESC
 LIMIT 1);

| person_id |                                                                       transcript                                                                                      |
|   14887   | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. |
|           | The man got into a car with a plate that included "H42W".                                                                                                             |
|   16371   | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                 | 

Deduction:

We will be querying the Gym databases to corroborate the witness accounts

Step 3: Find the membership information of the suspect using the witness account details 

QUERY:
SELECT * FROM 
(SELECT p.id,m.person_id, p.name, i.membership_id, d.plate_number
FROM person p
JOIN get_fit_now_member m ON p.id = m.person_id
JOIN get_fit_now_check_in i ON m.id = i.membership_id
JOIN drivers_license d ON p.license_id = d.id
WHERE m.id LIKE '%48Z%' AND m.membership_status = 'gold' AND i.check_in_date = '20180109'
AND d.plate_number LIKE '%H42W%');

|  id   | person_id |      name     | membership_id | plate_number |
| 67318 |   67318   | Jeremy Bowers |     48Z55     |    0H42W2    |

Step 4: Interviewing the Suspect

QUERY:
SELECT * FROM interview 
WHERE person_id = 67318;

| person_id | transcript |
| 67318 | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. |
|       | I know that she attended the SQL Symphony Concert 3 times in December 2017.                                                                                          |

Deduction: Now that we have caught the assasin we have to find the main suspect who hired Jeremy Bowers

Step 5: Catching the killer

QUERY:
SELECT d.id,p.name, COUNT(f.person_id) AS check_in_count,f.person_id 
FROM facebook_event_checkin f
JOIN person p ON f.person_id = p.id
JOIN drivers_license d ON p.license_id = d.id
WHERE height BETWEEN 65 AND 67 AND gender = 'female' AND 
hair_color = 'red' AND  car_make = 'Tesla' AND 
car_model = 'Model S' AND f.event_name = 'SQL Symphony Concert' AND f.date LIKE '201712%' 
group by p.id, p.name
HAVING COUNT(f.person_id) = 3;

|   id   |        name      | check_in_count | person_id |
| 202298 | Miranda Priestly |       3        |   99716   |

We've caught the killer, lets confirm by checking the solution key
INSERT INTO solution VALUES (1, "Miranda Priestly");

SELECT * FROM solution;

| user |                                                                       value                                                                                  |
|  0   | Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne! |




