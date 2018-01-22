### Projekt Teil 2

Aufgaben 1, 3, 4, 5: siehe `proj2.md`
<br><br>Aufgabe 2: siehe `jdbcdemo.tar.gz` (IntelliJ project)


### Queries

**1)**

>Select e.firstname, e.lastname
from Employee as e
where e.lastname LIKE 'A%'
union 
select p.firstname, p.lastname
from Patient as p
where p.lastname LIKE 'A%'

**2)**

>select plz.PLZ, plz.ORT, count(p.PatientID)
from Patient p join PLZORT plz
where
p.PLZORT_PLZ = plz.PLZ
group by plz.PLZ
union
select plz.PLZ, plz.ORT, count(p.EmployeeID)
from Employee e join PLZORT plz
where
e.PLZORT_PLZ = plz.PLZ
group by plz.PLZ

**3)**

>select * 
from Questionnare q join filled f
where q.QuestionnareTitle = f.QuestionnareTitle
group by q.QuestionnareTitle
having count(f.QuestionnareTitle > 100)


**4)**

>select *
from Patient p
where
admissionDate < '2019-02-02' AND
admissionDate > '2017-02-02'

**5)**


>select s.StudyTitle, avg(cte.nummer)
from 
(select q.QuestionnareTitle , avg(qa.nummer) as nummer
from Questionnare q join Question qa
where
q.QuestionnareTitle = qa.Questionare_QuestionareTitle 
group by q.QuestionnareTitle) as cte 
join Study s join StudyContainsQuestion sc 
where
cte.QuestionnareTitle = sc.Questionnare_Title AND
s.StudyTitle = sc.Study_StudyTitle AND
group by s.StudyTitle


**6)**


>select *
from Employee e join works w join
(
select dp.idDepartment
from Department dp join SeniorDoctor d join Expertise e
where 
dp.Head_EmployeeID = d.Employee_EmployeeID AND
d.Employee_EmployeeID = e.Employee_EmployeeID
e.Title LIKE '%'
) as cte
where 
e.EmployeeID = w.Employee_EmployeeID AND
w.Department_idDepartment = cte.idDepartment AND
e.salary > 100

**7)**

>Select e.Title, count(d.Employee_EmployeeID) 
from Expertise e join Doctor d
where 
e.Employee_EmployeeID = d.Employee_EmployeeID
group by e.Title

### JDBC

The implementation can be found, zipped, as Maven project in `/` of this folder.

### Performance

**Query 1**:  39 total, Query took 0.0008 seconds.<br>
**Query 2**:  81 total, Query took 0.0008 seconds.<br>
**Query 3**:  2 total, Query took 0.0004 seconds.<br>
**Query 4**:  14 total, Query took 0.0001 seconds.<br>
**Query 5**:  94 total, Query took 0.0008 seconds.<br>
**Query 6**:  204 total, Query took 0.001 seconds.<br>
**Query 7**:  193 total, Query took 0.0008 seconds.<br>
**Query 8**:  193 total, Query took 0.0008 seconds.<br>

#### Thoughts

Obviously querying is a breeze with our small-ish datasets. Noteworthy might be that we ran all queries thru phpMyAdmin on a MySQL server with 250MB of RAM. Moreover, even if these results were sufficient to predict performance on a larger scale, some result might be skewed, as we relied on semi-realistic data. For example, we used words of length 2-6 for most strings, randomly generated ones, that is.

Supposedly there is a performance boost when replacing certain joins with selects. Although it worked after some tinkering, we didn't notice any difference (well).

Primary keys are often Strings, which is anything but optimal. Relying on integers would be both more convenient, and yield better performance (again, noticeable with larger datasets).

We used the InnoDB Engine (default), which is ACID-compliant. Some suggest using myISAM would yield performance benefits in some cases, but these are (supposedly) few and far between; at the cost of reduced integrity. Moreover, InnoDB uses clustered indexes, which makes picking the primary key a delicate matter.

Perhaps measures such as tweaking the persistence of the cache (which is deleted on restarts, and has to be rebuilt) could increase performance after such events.

### Materialized view

>create table patient_view as (
select PatientID, firstname, lastname, svn, admissionDate
from Patient
)

CREATE TABLE: Query took 0.1797 seconds.

>select * from patient_view

SELECT: (1000 Elements): Query took 0.0002 seconds.

INSERT 9000 Elements, via CSV: > 5 minutes (on a 250 MB RAM Server). Import script timed out at the following point:
`6641 Elements inserted`, in estimated `5 minutes`.

>select * from patient_view

SELECT: (6641 Elements): Query took 0.0002 seconds.

The SSN is an Integer, so we added the Suffix `-A` to every first Name.

>update patient_view set firstname = CONCAT(svn, "-A")

UPDATE: 6641 rows affected: Query took 0.1424 seconds.

>delete from patient_view

DELETE: 6641 rows deleted: Query took 0.0772 seconds.


### Materialized view with triggers

**CREATE TABLE:**

>create table patient_view as (
select PatientID, firstname, lastname, svn, admissionDate
from Patient
)

**INSERT TRIGGER:** 

>DROP TRIGGER IF EXISTS upd
DELIMITER //
CREATE TRIGGER upd
AFTER INSERT ON Patient
FOR EACH ROW
BEGIN
select PatientID, firstname, lastname, svn, admissionDate
from Patient
where Patient.PatientID = NEW.PatientID
into @id, @first, @last, @svn, @admdate;
replace into patient_view
values (@id, @first, @last, @svn, @admdate);
END;
//
DELIMITER ;

**DELETE TRIGGER:**

>DROP TRIGGER IF EXISTS upd
DELIMITER //
CREATE TRIGGER upd
AFTER DELETE ON Patient
FOR EACH ROW
BEGIN
Delete from patient_view
where
patient_view.PatientID = OLD.PatientID;
END;
//
DELIMITER ;

Another option would be ON CASCADE DELETE.

INSERTs seemed to perform better than building the materialized view on-demand, but we were kept from testing the DELETE trigger due to foreign key constraint violations caused by crappy ER-Modelling. The former is hardly surprising, considering it is compared to building a table on-demand, while with these triggers, we are finding, and updating affected entries only.


