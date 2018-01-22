1.

>Select e.firstname, e.lastname
from Employee as e
where e.lastname LIKE 'A%'
union 
select p.firstname, p.lastname
from Patient as p
where p.lastname LIKE 'A%'

1.

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

1.

>select * 
from Questionnare q join filled f
where q.QuestionnareTitle = f.QuestionnareTitle
group by q.QuestionnareTitle
having count(f.QuestionnareTitle > 100)


1. 

>select *
from Patient p
where
admissionDate < '2019-02-02' AND
admissionDate > '2017-02-02'

1.

???


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


1.


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

1.

>Select e.Title, count(d.Employee_EmployeeID) 
from Expertise e join Doctor d
where 
e.Employee_EmployeeID = d.Employee_EmployeeID
group by e.Title