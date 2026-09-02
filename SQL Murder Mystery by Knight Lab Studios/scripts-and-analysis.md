# SQL Murder Mystery by [Knight Lab Studio](https://mystery.knightlab.com/)
Can you find out whodunnit?

## :file_folder: Prompt
A crime has taken place and the detective needs your help. The detective gave you the
crime scene report, but you somehow lost it. You vaguely remember that the crime
was a murder that occurred sometime on Jan. 15, 2018 and that it took place in SQL
City. Start by retrieving the corresponding crime scene report from the police
department’s database. If you want to get the most out of this mystery, try to work
through it only using your SQL environment and refrain from using a notepad.

## :bookmark_tabs: Initial Details
From the prompt, we can narrow down our case with the following details:
- Type of crime: **murder**
- Date of crime: **January 15, 2018**
- Place of crime: **SQL City**

It is stated that the crime scene report can be retrieved from the police department's database. *Where is the crime scene report in the database?*

## :mag: Query: Database 
To see the list of tables and columns in the database, we run this query:

```sql
SELECT name, sql
FROM sqlite_master
WHERE type = 'table'
```
|          name          |                                                                                                            sql                                                                                                            |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| crime_scene_report     | CREATE TABLE crime_scene_report ( date integer, type text, description text, city text )                                                                                                                                  |
| drivers_license        | CREATE TABLE drivers_license ( id integer PRIMARY KEY, age integer, height integer, eye_color text, hair_color text, gender text, plate_number text, car_make text, car_model text )                                      |
| facebook_event_checkin | CREATE TABLE facebook_event_checkin ( person_id integer, event_id integer, event_name text, date integer, FOREIGN KEY (person_id) REFERENCES person(id) )                                                                 |
| interview              | CREATE TABLE interview ( person_id integer, transcript text, FOREIGN KEY (person_id) REFERENCES person(id) )                                                                                                              |
| get_fit_now_member     | CREATE TABLE get_fit_now_member ( id text PRIMARY KEY, person_id integer, name text, membership_start_date integer, membership_status text, FOREIGN KEY (person_id) REFERENCES person(id) )                               |
| get_fit_now_check_in   | CREATE TABLE get_fit_now_check_in ( membership_id text, check_in_date integer, check_in_time integer, check_out_time integer, FOREIGN KEY (membership_id) REFERENCES get_fit_now_member(id) )                             |
| solution               | CREATE TABLE solution ( user integer, value text )                                                                                                                                                                        |
| income                 | CREATE TABLE income (ssn CHAR PRIMARY KEY, annual_income integer)                                                                                                                                                         |
| person                 | CREATE TABLE person (id integer PRIMARY KEY, name text, license_id integer, address_number integer, address_street_name text, ssn CHAR REFERENCES income (ssn), FOREIGN KEY (license_id) REFERENCES drivers_license (id)) |


From the result, the table we are looking for is `crime_scene_report` based on the columns (although the table name is self-documenting), which match the initial details we identified. The table consists of the `date`, `type`, and `city`.

## :mag: Query: Initial details

Let's preview one row of the table to check what the values look like per column.

```sql
SELECT * 
FROM crime_scene_report
LIMIT 1
```

|   date   |   type  |                    description                    | city |
|----------|---------|---------------------------------------------------|------|
| 20180115 | robbery | A Man Dressed as Spider-Man Is on a Robbery Spree | NYC  |

We can see that the date is formatted this way, allowing us to run our query using the initial details following this format for the date. We run the query using the following details: `murder, 20180115 (Jan. 15, 2018), SQL City`.

```sql
SELECT *
FROM crime_scene_report 
WHERE type = 'murder' -- Type of crime
    AND date = 20180115 -- Date of crime
    AND city = 'SQL City' -- Place of crime
```

|   date   |  type  |                                                                                        description                                                                                        |   city   |
|----------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

#### :bookmark_tabs: Report Details
Based on the description, there are 2 witnesses with the following details:
- Witness 1: Northwestern Dr, last house 
- Witnesss 2: Franklin Ave, named Annabel


#### :card_file_box: Tables Needed
To retrieve the witnesses identity, we will query the following table:
- `person`: address_street_name, address_number, name

## :mag: Query: Witnesses

#### :bust_in_silhouette: Witness 1

Let's query the first witness wherein the `address_street_name` should be **Northwestern Dr** and the query should return the row with the highest `address_number` since the report states that the person lives at the **last** house.

```sql
SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr' -- Address of witness
ORDER BY address_number DESC -- Sort the address_number from highest 
LIMIT 1 -- Return the highest number or the last house
```

|   id  |      name      | license_id | address_number | address_street_name |    ssn    |
|-------|----------------|------------|----------------|---------------------|-----------|
| 14887 | Morty Schapiro |     118009 |           4919 | Northwestern Dr     | 111564949 |

#### :bust_in_silhouette: Witness 2

Next, we query the second witness wherein the `address_street_name` should be **Franklin Ave** and their name is **Annabel**.

```sql
SELECT *
FROM person
WHERE address_street_name = 'Franklin Ave' -- Address of witness
    AND name LIKE '%Annabel%' -- Name of the witness
```

|   id  |      name      | license_id | address_number | address_street_name |    ssn    |
|-------|----------------|------------|----------------|---------------------|-----------|
| 16371 | Annabel Miller |     490173 |            103 | Franklin Ave        | 318771143 |

#### :bookmark_tabs: Report Details
Based on the description, the 2 witnesses are:\
    ☑️ Witness 1: Morty Schapiro, 14887, 4919 Northwestern Dr\
    ☑️ Witnesss 2: Annabel Miller, 16371, 103 Franklin Ave

#### :card_file_box: Tables Needed
Now that we have identified the witnesses, let's learn more details by querying the following tables:
- `interview`: transcript
- `person`(optional): reference `person_id` in `interview` table using `id`

## :mag: Query: Witness 1 & 2 Interview

The query will retrieve the testimonies of Morty and Annabel, with IDs 14887 and 16371 respectively.

```sql
SELECT i.person_id, p.name, i.transcript 
FROM interview i 
JOIN person p ON i.person_id = p.id /* JOIN is optional, but used in this query to 
also display the name of the person along with their ID and the transcript */
WHERE i.person_id IN (14887, 16371) -- Return results with IDs matching the IDs of Morty and Annabel
```

| person_id |      name      |                                                                                                            transcript                                                                                                           |
|-----------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     14887 | Morty Schapiro | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |
|     16371 | Annabel Miller | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                                                                           |

#### :bookmark_tabs: Report Details
From the witnesses' transcripts, we can summarize the following details:
- Gender: Male
- Membership number: Starts with "48Z"
- Membership status: gold
- Car plate number: With "H42W"
- Gym check in date: January 9

#### :card_file_box: Tables Needed
To find the person identified by the witnesses, we will query the following tables:
- `drivers_license`: gender and plate_number columns
- `get_fit_now_member `: id(membership number) and membership status
- `get_fit_now_check_in `: check_in_date
- `persons`: contains columns to join the tables mentioned:
    - license_id is id in drivers_license
    - id is person_id in get_fit_now_member


## :mag: Query: The Man in Witnesses' Testimonies

The query will return all the mentioned details by both witnesses, including their `id` and `name`.

```sql
SELECT p.id, p.name, d.gender, d.id as license_id, gym.id as gym_id,
    gym.membership_status, d.plate_number, ci.check_in_date 
    -- select only the important columns instead of returning all
FROM drivers_license d
JOIN person p ON d.id = p.license_id -- JOIN person and their driver's license details
JOIN get_fit_now_member gym ON gym.person_id = p.id -- JOIN person and their gym membership details
JOIN get_fit_now_check_in ci ON gym.id = ci.membership_id -- JOIN membership details and membership check in records

WHERE d.gender = 'male' -- A man ran out
    AND gym.id LIKE '48Z%' -- Membership bag started with 48Z
    AND gym.membership_status = 'gold' -- Gold members own the kind of bag
    AND d.plate_number LIKE '%H42W%'  -- Car with plate number containing H42W
    AND ci.check_in_date = '20180109' -- Seen in gym on January 9

```

|   id  |      name     | gender | license_id | gym_id | membership_status | plate_number | check_in_date |
|-------|---------------|--------|------------|--------|-------------------|--------------|---------------|
| 67318 | Jeremy Bowers | male   |     423327 | 48Z55  | gold              | 0H42W2       |      20180109 |


#### :bookmark_tabs: Report Details
From the query, we can see that one person named Jeremy Bowers matched all the details:\
    ☑️ Gender: Male\
    ☑️ Membership number: 48Z55\
    ☑️ Membership status: gold\
    ☑️ Car plate number: 0H42W2\
    ☑️ Gym check in date: January 9, 2018

#### :card_file_box: Tables Needed
We will check the `interview` table once again to know about Jeremy's testimony:
- `interview`: transcript
- `person`(optional): reference `person_id` in `interview` table using `id`


## :mag: Query: Interview of Jeremy Bowers

The query will retrieve the testimony of Jeremy with the ID 67318. 

```sql
SELECT i.person_id, p.name, i.transcript
FROM interview i
JOIN person p ON i.person_id = p.id /* JOIN is optional, but used in this query to 
also display the name of the person along with their ID and the transcript */
WHERE person_id = 67318 -- Return results using the ID of identified person
```

| person_id |      name     |                                                                                                                    transcript                                                                                                                    |
|-----------|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|     67318 | Jeremy Bowers | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. |

#### :bookmark_tabs: Report Details
From Jeremy's interview, we can summarize the following details:
- Gender: Female
- Height: Around 65"-67"
- Hair color: red
- Car: Tesla Model S
- Event: SQL Symphony Concert
- Times attended the event: 3 times
- Event date: December 2017

#### :card_file_box: Tables Needed
To find the woman who hired Jeremy, the tables we need are the following:
- `drivers_license`: gender, hair color, height, car_make, and car_model
- `facebook_event_checkin`: event_name, date
- `persons`: contains columns that join the tables mentioned:
    - license_id is id in `drivers_license`
    - id is person_id in `facebook_event_checkin`
    - ssn in `income`
- `income` (optional): annual_income

## :mag: Query: The Woman
We look for the person by using the mentioned tables in our query, as well as using a CTE to create a temporary table for the event details:

```sql
/* 
    Create a temporary table returning all columns and rows from facebook_event_checkin table
    where the event is SQL Symphony Concert attended on the whole month of December.
    The rows are grouped per person and filter rows
    where the number of times attended should be exactly 3.
*/
WITH event as(
    SELECT *, COUNT(*) as times_attended
    FROM facebook_event_checkin
    WHERE event_name = "SQL Symphony Concert" -- The woman attended this event
        AND date BETWEEN 20171201 AND 20171231 -- The woman attended the event in December 2017
    GROUP BY person_id -- Group the information by person
    HAVING COUNT(*) = 3 -- The woman attended the event 3 times
  )
  
SELECT event.event_name, event.date, event.times_attended, event.person_id, p.name,
    d.id as license_id, d.hair_color, d.gender, d.height, d.car_make,
    d.car_model, i.annual_income -- Select the details provided by Jeremy. Annual income is optional
FROM event
JOIN person p on event.person_id = p.id -- JOIN person and their attended events
JOIN drivers_license d on d.id = p.license_id -- JOIN person and their driver's license details
JOIN income i using (ssn) -- JOIN person and their income
WHERE d.gender = 'female' -- Hired by a woman
    AND d.height BETWEEN 65 and 67 -- Height is around 65-67
    AND d.hair_color = 'red' -- Has red hair
    AND d.car_make = 'Tesla' -- Drives a Tesla
    AND d.car_model = 'Model S' -- Drives a Tesla with this model
```

|      event_name      |   date   | times_attended | person_id |       name       | license_id | hair_color | gender | height | car_make | car_model | annual_income |
|----------------------|----------|----------------|-----------|------------------|------------|------------|--------|--------|----------|-----------|---------------|
| SQL Symphony Concert | 20171229 |              3 |     99716 | Miranda Priestly |     202298 | red        | female |     66 | Tesla    | Model S   |        310000 |


#### :bookmark_tabs: Report Details
From the query, we can see that one person named Miranda Priestly matched all the details:\
    ☑️ Gender: Female\
    ☑️ Height: 66\
    ☑️ Hair color: red\
    ☑️ Car: Tesla Model S\
    ☑️ Event: SQL Symphony Concert\
    ☑️ Times attended the event: 3 times\
    ☑️ Event date: December 29, 2017

We have found the murderer named **Miranda Priestly**.


## :judge: Guilty or Not Guilty?

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly'); 

SELECT value FROM solution;
```

|                                                                            value                                                                            |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne |




