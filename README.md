<h1>CPerlick T-SQL Email Notification</h1>


<h2>Description</h2>
This project alerts a 3rd party delivery team via email when a new order is placed with a specific set of parameters. The script utilizes a stored procedure that checks for new records, inserts the new records into a log table, and utilizes a temporary table and a cursor to send an email notification to the 3rd party. The stored procedure then updates the log table and drops the temp table.
<br />


<h2>Languages and Utilities Used</h2>

- <b>SQL Server Management Studio</b>
- <b>Sage X3 V11 ERP & DB</b> 

<h2>Program walk-through:</h2>

<p align="left">
<h3>1. Check if new records exist:</h3> 
declare @ct int 	/**Delcaring a count variable to check for new records**/<br /> 
select @ct= COUNT(*) from <br /> 
(select <br /> 
h.CHAR1_0 <br /> 
from x3.PROD.AUDITH h <br /> 
inner join x3.PROD.AUDITL d on h.SEQ_0=d.SEQ_0 and TBL_0='SDELIVERY' 	/**using DB audit tables to see when new delivery records were created**/ <br /> 
inner join x3.PROD.SDELIVERY so on so.SDHNUM_0=h.ID1_0 <br /> 
where TBL_0='SDELIVERY' and <br /> 
d.NVAL_0 in ('DELIVERY')<br /> 
and so.MDL_0='DE' and so.STOFCY_0='416'<br /> 
) a <br /> 
where a.CHAR1_0 not in<br /> 
(select SDHNUM_0 from  x3.PROD.YEMLSDHLOGBO) 	/**ensuring only new records are included in the count**/ <br /> 
print @ct 	/**Print the count variable as a check point**/ <br /> 
  <br /> 
if @ct>0 <br /> 
	begin 	/**insert records in LOG Table**/<br /> 
<br />
<br />
<p align="left">
<h3>2. Insert records into log table:</h3>
insert into x3.PROD.YEMLSDHLOGBO <br /> 
	([SDHNUM_0], MDL_0, <br /> STOFCY_0,BPDNAM_0,BPDNAM_1,BPDADDLIG_0,BPDADDLIG_1,BPDADDLIG_2,BPDPOSCOD_0,BPDCTY_0,BPDSAT_0,BPDCRY_0,YMBPITEL_0,YMBPIWEB_0,YMDLVTIM_0,YPICKUPADR_0,SHIDAT_0,[CREDAT_0],<br /> [CREDATTIM_0],EMAILED_0,EMAILDATE_0,EMAILADDRESS_0,SEQ_0,UPDDATTIM_0,AUUID_0)<br /> 
	select h.ID1_0 as Sdhnum, so.MDL_0,<br /> 
  so.STOFCY_0,so.BPDNAM_0,so.BPDNAM_1,<br /> 
  so.BPDADDLIG_0,so.BPDADDLIG_1,<br /> 
  so.BPDADDLIG_2,so.BPDPOSCOD_0,<br /> 
  so.BPDCTY_0,so.BPDSAT_0,<br /> 
  so.BPDCRY_0,so.YMBPITEL_0,<br /> 
  so.YMBPIWEB_0,so.YMDLVTIM_0,<br /> 
  '85i Northern Avenue Boston, MA 02210',so.SHIDAT_0,<br /> 
  getdate(),getdate(),<br /> 
  0,getdate(),'',<br /> 
  d.SEQ_0,getdate(),0<br /> 
	from x3.PROD.AUDITH h<br /> 
	inner join x3.PROD.AUDITL d on h.SEQ_0=d.SEQ_0 and TBL_0='SDELIVERY'<br /> 
	inner join x3.PROD.SDELIVERY so on so.SDHNUM_0=h.ID1_0<br /> 
	where TBL_0='SDELIVERY'<br /> 
	and d.NVAL_0 in ('DELIVERY')	<br /> 
	and so.STOFCY_0='416'<br /> 
  and h.ID1_0 not in<br /> 
	(select SDHNUM_0 from  x3.PROD.YEMLSDHLOGBO)<br /> 
	/**uses the same parameters as the count above**/ 
	end<br /> 
<br />
<h3>3.Check that previous temp table is dropped:</h3>
  begin<br /> 
		If object_id('tempdb..##spudB') is not null<br /> 
		drop table ##spudB<br /> 
		print 'Drop table complete'<br /> 
	end<br /> 
<br />
<h3>4.Create Temp table:</h3>
  begin <br /> 
		declare @check int<br /> 
		/**Declare variable to establish if there are records to report upon**/<br /> 
		create table ##spudB<br /> 
		(	[SDHNUM_0][nvarchar](20) null, [MDL_0][nvarchar](20) null<br /> 
    ,[STOFCY][nvarchar](5) null,[BPDNAM_0][nvarchar](50) null<br /> 
		,[BPDNAM_1][nvarchar](50) null,[BPDADDLIG_0][nvarchar](100) null<br /> 
		,[BPDADDLIG_1][nvarchar](100) null,[BPDADDLIG_2][nvarchar](100) null<br /> 
		,[BPDPOSCOD_0][int] null,[BPDCTY_0][nvarchar](100) null<br /> 
		,[BPDSAT_0][nvarchar](3) null,[BPDCRY_0][nvarchar](4) null<br /> 
		,[YMBPITEL_0][nvarchar](100) NULL,[YMBPIWEB_0][nvarchar](100) NULL<br /> 
		,[YMDLVTIM_0][int] null,[YPICKUPADR_0] [nvarchar](100) null<br /> 
		,[SHIDAT_0][datetime] NULL,[CREDAT_0] [datetime] NULL<br /> 
		,[EMAILED_0] [int] null)<br /> 
		print 'Create table complete'<br /> 
<br />
<h3>5.Insert data into Temp Table </h3>
		insert into [##spudB]<br /> 
		(<br /> 
		[SDHNUM_0],MDL_0, <br /> STOFCY,BPDNAM_0,BPDNAM_1,BPDADDLIG_0,BPDADDLIG_1,BPDADDLIG_2,BPDPOSCOD_0,BPDCTY_0,BPDSAT_0,BPDCRY_0,YMBPITEL_0,YMBPIWEB_0,YMDLVTIM_0,YPICKUPADR_0,SHIDAT_0,<br /> [CREDAT_0],EMAILED_0)<br /> 
		select<br /> 
			sd.SDHNUM_0,sd.MDL_0,<br /> 
			sd.STOFCY_0,sd.BPDNAM_0,<br /> 
			sd.BPDNAM_1,sd.BPDADDLIG_0,<br /> 
      sd.BPDADDLIG_1,sd.BPDADDLIG_2,<br /> 
			sd.BPDPOSCOD_0,sd.BPDCTY_0,<br /> 
			sd.BPDSAT_0,sd.BPDCRY_0,<br /> 
			sd.YMBPITEL_0,sd.YMBPIWEB_0,<br /> 
			sd.YMDLVTIM_0,B.YPICKUPADR_0,<br /> 
			sd.SHIDAT_0,sd.CREDAT_0,<br /> 
			B.EMAILED_0<br /> 
		from x3.PROD.YEMLSDHLOGBO B<br /> 
		left join x3.PROD.SDELIVERY sd on sd.SDHNUM_0 = B.SDHNUM_0<br /> 
		where sd.MDL_0='DE' and sd.STOFCY_0 ='416' and EMAILED_0 =0<br /> 
		print 'Insert into table complete'<br /> 
		/**Verification that records do exist**/<br /> 
		select @check = count(*) from [##spudB]<br /> 
	end<br />
<br />
<h3>6.Create Cursor for dynamic emails: </h3>
DECLARE @spidemail nvarchar(50)<br /> 
<br />
<br />
<h3>7.Clean up Cursor: </h3>  
Cleanup of cursor<br /> 
		IF (SELECT CURSOR_STATUS('global','spid_cursor')) >= -1<br /> 
		 BEGIN<br /> 
				IF (SELECT CURSOR_STATUS('global','spid_cursor')) > -1<br /> 
				BEGIN<br /> 
				CLOSE spid_cursor<br /> 
				END<br /> 
				DEALLOCATE spid_cursor<br /> 
			END<br /> 
<br />
<h3>8.Declare the cursor from a SQL query: It will list valid email addresses </h3>  
		DECLARE spid_cursor CURSOR FOR <br /> 
		select adr.EXTNUM_0 from ##spudB l<br /> 
		inner join x3.PROD.FACILITY f on f.FCY_0=l.STOFCY<br /> 
		inner join x3.PROD.BPADDRESS adr on adr.BPATYP_0=3 and adr.BPANUM_0=f.FCY_0<br /> 
		group by adr.EXTNUM_0<br /> 
<br />
<h3>9.Open the cursor / guest list. this is when you start going down your list of records </h3>
OPEN spid_cursor
<br />
<br />
<h3>10.Select the first non-checked record from the list into the variables defined in # 6. </h3>  
FETCH NEXT FROM spid_cursor<br /> 
			INTO @spidemail;<br /> 
			print @spidemail<br /> 
<br />
<h3>11.Standard syntax that means there are still records left in the cursor</h3>
WHILE @@FETCH_STATUS = 0<br /> 
BEGIN
<br />
<br />
<h3>12.Start Printing email </h3>  
	print 'Sending Email'<br /> 
	DECLARE @email nvarchar(MAX)<br /> 
	DECLARE @html nvarchar(MAX);<br /> 
	EXEC x3.PROD.spQueryToHtmlTable @html = @html OUTPUT,  <br /> 
	@query = <br /> 
	'select <br /> 
	SPACE(1) +l.SDHNUM_0+ SPACE(1)  as [  Order ], <br /> 
	CASE<br /> 
		WHEN o.MDL_0 = ''DE''<br /> 
		THEN ''Delivery''<br /> 
		WHEN o.MDL_0 = ''PU''<br /> 
		THEN ''Pick-Up''<br /> 
		END AS '' Method ''<br /> 
		,CASE <br /> 
		when o.YMDLVTIM_0 = 2 then SPACE(1) + SPACE(1)+''9:00AM - 11:00AM''+ SPACE(1) + SPACE(1)<br /> 
		when o.YMDLVTIM_0 = 3 then SPACE(1) + SPACE(1)+''11:00AM - 1:00PM''+ SPACE(1) + SPACE(1)<br /> 
		when o.YMDLVTIM_0 = 4 then SPACE(1) + SPACE(1)+''1:00PM - 3:00PM''+ SPACE(1) + SPACE(1)<br /> 
		when o.YMDLVTIM_0 = 5 then SPACE(1) + SPACE(1)+''3:00PM - 5:00PM''+ SPACE(1) + SPACE(1)<br /> 
		when o.YMDLVTIM_0 = 6 then SPACE(1) + SPACE(1)+''5:00PM - 7:00PM''+ SPACE(1) + SPACE(1)<br /> 
		when o.YMDLVTIM_0 = 7 then SPACE(1) + SPACE(1)+''7:00PM - 9:00PM''+ SPACE(1) + SPACE(1) <br /> 
		end as ''Delivery Time Slot''<br /> 
	,SPACE(1) + SPACE(1)+ convert(varchar(8),o.SHIDAT_0,1)+SPACE(1) + SPACE(1) as [ Delivery Date ]<br /> 
			,SPACE(1) + SPACE(1)+ o.BPDNAM_0+ SPACE(1) +o.BPDNAM_1+SPACE(1) + SPACE(1) as [ Recipient Name ],<br /> 
			SPACE(1) + SPACE(1)+ o.BPDADDLIG_0+ SPACE(1) + o.BPDADDLIG_1 + SPACE(1) + o.BPDCTY_0 +'',''+ SPACE(1) + o.BPDSAT_0 + SPACE(1) + o.BPDPOSCOD_0 + SPACE(1) as [ Drop Off Address ],<br /> 
			SPACE(1) + SPACE(1)+ o.YMBPDTEL_0+ SPACE(1)+ SPACE(1) +SPACE(1)   as [ Recipient Telephone ],<br /> 
			SPACE(1) + SPACE(1)+ o.YMBPDWEB_0+ SPACE(1)+ SPACE(1) + SPACE(1)  as [ Recipient Email ],<br /> 
			SPACE(1) + SPACE(1)+ o.YMBPITEL_0+ SPACE(1)+ SPACE(1) +SPACE(1)   as [ Purchaser Telephone ],<br /> 
			SPACE(1) + SPACE(1)+ o.YMBPIWEB_0+ SPACE(1)+ SPACE(1) + SPACE(1)  as [ Purchaser Email ],<br /> 
			SPACE(1) + SPACE(1)+ l.YPICKUPADR_0+ SPACE(1)+ SPACE(1) + SPACE(1)  as [ Pick Up Location ]<br /> 
	from ##spudB l<br /> 
	inner join x3.PROD.SDELIVERY o on o.SDHNUM_0=l.SDHNUM_0';<br /> 
	print @html<br /> 
<br />
<br />
<h3>13.Declare vaiables and text that will be used in the Email</h3>
	declare @text varchar(500)<br /> 
	select @text = 'Delivery Request Alert'+'<br>'+'<br>'+'<br>'+
	'This email includes the upcoming delivery orders that will require a courier.'<br /> 
	'Thank You.'
	+'<br>'+'<br>'+
	+'<br>'+'<br>'+<br /> 
	'Operations Team'<br /> 
	set @email=@text + @html<br /> 
 <br/> 
 <br/>
 <h3>14.Send Email Notificication: </h3>
 EXEC msdb.dbo.sp_send_dbmail  <br /> 
		@profile_name = 'Email Alert',  <br /> 
		@recipients ='cperlick@fordham.edu',<br /> 
		@body = @email,<br /> 
		@body_format = 'HTML',<br /> 
		@subject = 'Delivery Request Alert' , <br /> 
		@query_no_truncate = 1,<br /> 
		@attach_query_result_as_file = 0;<br /> 
	print 'Email sent complete'<br /> 
 <br/>
 <h3>15.Marking records as emailed:</h3>
 update t<br /> 
	set t.EMAILED_0 =2, t.EMAILDATE_0=getdate(),t.[EMAILADDRESS_0]=@spidemail<br /> 
	from x3.PROD.YEMLSDHLOGBO t<br /> 
	inner join ##spudB u on t.SDHNUM_0=u.SDHNUM_0<br /> 
	inner join <br /> 
	(<br /> 
	select adr.EXTNUM_0,f.FCY_0<br /> 
	from PROD.FACILITY f<br /> 
	inner join x3.PROD.BPADDRESS adr on adr.BPATYP_0=3 and adr.BPANUM_0=f.FCY_0<br /> 
	where adr.EXTNUM_0<>''<br /> 
	group by adr.EXTNUM_0, f.FCY_0<br /> 
	) a on a.FCY_0 =u.STOFCY<br /> 
	where a.EXTNUM_0=@spidemail --Filter update of records only for this email.<br /> 
 print 'Update complete'<br /> 
 <br/> 
 <h3>16.Cleaning up </h3>
 /**dropping temporary table & deallocating cursor**/<br /> 
CLOSE spid_cursor<br /> 
DEALLOCATE spid_cursor<br /> 
drop table ##spudB <br /> 
 <br />
/**Clear records older than 30 days for CLOSED orders**/<br /> 
delete <br /> 
from x3.PROD.YEMLSDHLOG<br /> 
where EMAILDATE_0<=getdate()-365 and EMAILED_0=1<br /> 
and SDHNUM_0 in<br /> 
(Select SDHNUM_0 from x3.PROD.SDELIVERY where CFMFLG_0=2 /*Closed orders*/)<br /> 
GO<br /> 
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
