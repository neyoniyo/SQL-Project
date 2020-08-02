# SQL-Project
--Main SQL Project
--Import CSV
select * from gradeRecordModuleV;

select distinct * from gradeRecordModuleV
order by studentID asc;

--First normal form
--Check for duplicates
select studentid, count(*)
from gradeRecordModuleV group by studentid
having count(studentid)>1 order by studentid;

--Resolve to use the top 50 records because of duplicates
--in the supposed general record
select distinct top 50 * 
into graderecord50
from gradeRecordModuleV
order by studentID asc;

--Check for duplicates in the selected top 50 record
Use Premierdatabase;
select distinct * from graderecord50;

select studentid, count(*)
from gradeRecord50 group by studentid
having count(studentid)>1 order by studentid;

--Check datatype of each column
SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'graderecord50'
ORDER BY ORDINAL_POSITION;

select distinct * from graderecord50;

--Second normal form
--Create StudentInfo table
select studentID, first_name, lastname
into StudentInfo
from graderecord50
order by studentID asc;

select * from StudentInfo;

--check for duplicates
select studentid, count(*)
from studentinfo group by studentid
having count(studentid)>1 order by studentid;

--Create ResultInfo table
select studentID,
		Midtermexam,
		Finalexam,
		assignment1,
		assignment2,
		Totalpoints,
		Studentaverage,
		Grade
into ResultInfo
from graderecord50
order by studentID asc;

--Check for duplicates
select * from ResultInfo
order by grade asc;

select studentid, count(*)
from ResultInfo group by studentid
having count(studentid)>1 order by studentid;

--Add Grade_ID column to ResultInfo
ALTER TABLE ResultInfo
ADD Grade_ID int;

Update ResultInfo
Set Grade_ID = case
		when Grade = 'A' then 1
		when Grade = 'A+' then 2
		when Grade = 'A-' then 3
		when Grade = 'B' then 4
		when Grade = 'B+' then 5
		when Grade = 'B-' then 6
		when Grade = 'C' then 7
		when Grade = 'C+' then 8
		when Grade = 'C-' then 9
		when Grade = 'D' then 10
		when Grade = 'D+' then 11
		when Grade = 'D-' then 12
		when Grade = 'E' then 13
		when Grade = 'E+' then 14
		when Grade = 'E-' then 15
		when Grade = 'F' then 16
		when Grade = 'F+' then 17
		when Grade = 'F-' then 18
		Else 0
End
where Grade IN('A','A+','A-','B','B+','B-','C','C+','C-',
				'D','D+','D-','E','E+','E-','F','F+','F-');

ALTER TABLE ResultInfo
Alter column Grade_ID int not null;

select * from ResultInfo
order by grade asc;

--Third Normal Form
--Drop Grade Column from Resultinfo
ALTER TABLE ResultInfo
Drop column Grade;

select * from ResultInfo;

--Add ResultID column to ResultInfo
ALTER TABLE ResultInfo
ADD ResultID INT IDENTITY(1,1) NOT NULL;

--Create Gradeinfo table
SELECT Grade, Grade_ID
INTO GradeInfo
FROM ResultInfo 
WHERE Grade_ID=1 OR Grade_ID=2 OR Grade_ID=3
		OR Grade_ID=4 OR Grade_ID=5 OR Grade_ID=6
		OR Grade_ID=7 OR Grade_ID=8 OR Grade_ID=9
		OR Grade_ID=10 OR Grade_ID=11 OR Grade_ID=12
		OR Grade_ID=13 OR Grade_ID=14 OR Grade_ID=15
		OR Grade_ID=16 OR Grade_ID=17 OR Grade_ID=18;

select * from GradeInfo;

--Check for duplicates
select distinct * from GradeInfo;

--Create another table after removing the duplicates
select distinct * 
into GradeInfo2
from GradeInfo;

--The three tables formed
select * from GradeInfo2;

select * from ResultInfo;

Select * from StudentInfo;

--Add Primary and Foreign keys to tables
Alter table GradeInfo2
add constraint pk_grade_id primary key (grade_id);

Alter table ResultInfo
add constraint pk_ResultID primary key (ResultID);

Alter table StudentInfo
add constraint pk_studentid primary key (studentid);

ALTER TABLE ResultInfo
ADD CONSTRAINT fk_studentID FOREIGN KEY (studentID) 
REFERENCES StudentInfo(studentID);

ALTER TABLE ResultInfo
ADD CONSTRAINT fk_grade_ID FOREIGN KEY (grade_ID) 
REFERENCES GradeInfo2(grade_ID);

--Use the Join Statement combining the three tables
--to create a new table
select studentinfo.studentid, studentinfo.first_name,
		studentinfo.lastname, resultinfo.midtermexam,
		resultinfo.finalexam, resultinfo.assignment1,
		resultinfo.assignment2, resultinfo.totalpoints,
		resultinfo.studentaverage, gradeinfo2.grade
into ConsolidatedTable
from studentinfo inner join resultinfo
on studentinfo.studentID = resultinfo.studentID
inner join gradeinfo2
on resultinfo.grade_ID = gradeinfo2.grade_ID;

--The new table formed
select * from ConsolidatedTable;

--Comparing new table to the Premier table
select distinct * from graderecord50;
