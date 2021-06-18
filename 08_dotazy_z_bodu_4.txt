
SET AUTOTRACE ON;
--1. Vypsání všech neukončených projektů pro konkrétního uživatele - neukončený je když má neukončenou úlohu -- OK

SELECT p.name as ProjectName, t.taskname as TaskName, t.completed as Completed FROM Project p join ProjectTask t on (t.projectId = p.projectId) WHERE p.ownerID = 1 AND t.completed = 0 ;

--2. Vypsání všech projektových úloh pro konkrétní projekt -- OK
select t.taskName AS TaskName FROM ProjectTask t WHERE t.id = 1;

--3. Zobrazit uživatele, který vyvolává nejvíce chybových hlášek - OK

SELECT * FROM (SELECT u.username AS Uziv, COUNT(l.userid) AS ErrorLogs FROM Uzivatel u JOIN Log l ON (u.uzivatelId = l.userid) WHERE l.type = 1 GROUP BY u.username ORDER BY ErrorLogs DESC) WHERE rownum < 2; -- OK

--SELECT u.username AS Uziv, COUNT(l.userid) AS ErrorLogs FROM Uzivatel u JOIN Log l ON (u.uzivatelId = l.userid) WHERE l.type = 1 GROUP BY u.username  ORDER BY ErrorLogs DESC FETCH FIRST 1 ROWS ONLY; -- Oracle 12, mělo by být ryhlejší, protože neobsahuje poddotaz

--4. Zobrazit 10 uživatelů, kteří mají největší počet klientů - OK

select u.username as Uziv, COUNT(uc.clientId) as ClientCount FROM Uzivatel u JOIN Client uc ON (u.uzivatelId = uc.userId) WHERE rownum < 11 GROUP BY u.username ORDER BY ClientCount DESC ;

--5. Součet celkové částky pro položky faktury - OK

--varianta 1
SELECT SUM(taskTotal) AS invoiceTotal FROM 
(SELECT pt.hourRate*DateDiff as taskTotal FROM 
(SELECT (TO_DATE(te.endDate, 'DD.MM.YYYY') - TO_DATE(te.startDate, 'DD.MM.YYYY')) * 24 AS DateDiff, te.projectTaskId as teid FROM TaskEntry te JOIN ProjectTask pt ON (te.projectTaskID = pt.projectTaskId)) join ProjectTask pt on (pt.projectTaskId = teid) join InvoiceProjectTasks ipt on (pt.projectTaskId = ipt.projectTaskId)); --WHERE te.projectTaskID = pt.projectID)

-- varianta 2
select sum(pt.hourRate*(TO_DATE(te.endDate, 'DD.MM.YYYY') - TO_DATE(te.startDate, 'DD.MM.YYYY')) * 24) as invoiceTotal FROM TaskEntry te JOIN ProjectTask pt ON (te.projectTaskID = pt.projectTaskId) join InvoiceProjectTasks ipt on (pt.projectTaskId = ipt.projectTaskId) group by te.projectTaskId;

SET AUTOTRACE OFF;