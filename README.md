## Table of Contents
## Cmd
###### Search subfolders bare view save to file
```cmd
dir *.csproj /b /s /ogn >> c:\temp\test.txt
```
###### DelayedExpansion
```cmd
setlocal EnableDelayedExpansion
set var1=true
if "%var1%"=="true" (
  set var2=myvalue
  echo !var2!
)
```
###### Connect via RDC console
```cmd
mstsc/v:server-tdc /console
```
###### Get current folder
```cmd
%cd%
```
## Javascript
###### OP-style Inheritance
```javascript
function inherit_A(Child, Parent)
{
    var F = function () { };
    F.prototype = Parent.prototype;

    Child.prototype = new F();
    Child.prototype.constructor = Child;
    Child.super = Parent.prototype;
}
```
### jQuery
###### AJAX get array
Current version

Request URL
http://page.local/System/Agent/GetSignature?id=2&companyProfileId=1&test%5B%5D=1&test%5B%5D=2&test%5B%5D=3
```javascript
$.ajax({
    url: url,
    data: {
        id: thisId,
        companyProfileId: companyProfileId,
        test: [1,2,3]
    },
    success: function (data) {
        if (data.Success) {
            fillData(data);
        }
    }
});
```
Traditional (either per request or jQuery.ajaxSettings.traditional = true)

Request URL
http://page.local/System/Agent/GetSignature?id=2&companyProfileId=2&test=1&test=2&test=3

Controller

public ActionResult GetSignature(int id, int companyProfileId, IEnumerable<int> test)
```javascript
$.ajax({
    url: url,
    traditional: true,
    data: {
        id: thisId,
        companyProfileId: companyProfileId,
        test: [1,2,3]
    },
    success: function (data) {
        if (data.Success) {
            fillData(data);
        }
    }
});
```
###### AJAX post array
There two important things to note
```
    dataType: "json",
    contentType: 'application/json',
    data: JSON.stringify(caseIdList),
```
```javascript
$.ajax({
    url: caseListDetailsUrl,
    type: "POST",
    dataType: "json",
    contentType: 'application/json',
    data: JSON.stringify(caseIdList),
    success: function (data) {
        if (data.Success) {
            newCompanyProfileId = data.CompanyProfileId;
        }
    }
});
```
Controller
public ActionResult GetCaseListDetails(IEnumerable<int> caseIdList)
### react
###### Conditional output in render
```jsx
    return (
      <div className="previewPanel">
        I am a PreviewPanel.<br/>
        I am showing {this.props.caseId ? "case " + this.props.caseId : "nothing"}
        {previewControl}
      </div>
    );
```
###### Pass data to react component
Store a reference to it, returned by renderComponent method and invoke setState, passing your data.

## SQL
###### how to create sysadmin
I. Force SQL server to support mixed-mode authentication.  
1. Run REGEDIT  
2. Go to HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL10.SQLEXPRESS\MSSQLServer  
NOTE: This key may vary slightly based on the installed version and instance name.  
3. Set "LoginMode" to 2.  
4. Restart SQL Server.  
(Source: http://support.microsoft.com/kb/285097)

II. Force SQL server to let you in temporarily  
1. Go to services.  
2. Stop SQL Server.  
3. Grab the SQL server command-line (right click the service - properties).  Mine is:  
"C:\Program Files\Microsoft SQL Server\MSSQL10.SQLEXPRESS\MSSQL\Binn\sqlservr.exe" -sSQLEXPRESS  
4. Open an administrative command prompt.  
5. Run the command-line from step 3, but add -m -c for single-user maintenance mode command-line.  
6. Open another administrative command prompt.  
7. Run "sqlcmd -S localhost\SQLEXPRESS" from that same directory (replace with your server and instance name)  
8. Now you can do all the stuff everyone told you to do that didn't work.  For example, to create a hero user with administrative access:  
```sql
CREATE LOGIN hero WITH PASSWORD='123', DEFAULT_DATABASE=[master], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
EXEC sys.sp_addsrvrolemember @loginname = 'hero', @rolename = 'sysadmin'
GO
```
9. QUIT and close the command-prompt  
10. Go to the SQL Server command-line window and hit ctrl+C.  It will prompt "Do you wish to shutdown SQL Server (Y/N)?" and enter Y.  
11. Close the command-prompt  
(Source: http://msdn.microsoft.com/en-us/library/dd207004.aspx)  

III. Finally, login using your hero:  
1. Restart the SQL Server service  
2. Login using SQL Server authentication as the user "hero" with password "123"  
3. *BAM* now you are in.  Now give yourself sysadmin access and delete the temporary user.  
###### allow executing assembly
```sql
Declare @sqlstsmt nvarchar(200)
set @sqlstsmt=N'ALTER AUTHORIZATION ON DATABASE::['+DB_NAME()+N'] TO [sa]'
exec(@sqlstsmt)
go

Declare @sql nvarchar(200)
SET @sql = 'ALTER DATABASE [' + db_name() + '] SET TRUSTWORTHY ON'
EXEC sp_executeSql @sql 
go

sp_configure 'clr enabled', 1
go
```
###### variables sample
```sql
DECLARE @orderId INT;
SET @orderId = (SELECT TOP 1 orderId FROM [Order] ORDER BY OrderId DESC);

DECLARE @nextInvoiceDate dateTime;
SET @nextInvoiceDate = '12-31-2013';

UPDATE
	Domain
SET
	HasRunSetup = 1,
	SetupComplete = 1,
	NextInvoiceDate = @nextInvoiceDate
WHERE DomainId in (
	SELECT domainId
	FROM OrderDomain
	WHERE OrderId = @orderId
	)
```
###### print multiline
```sql
-- Option 1
DECLARE @text NVARCHAR(MAX)

SELECT @text = [BodyText]
FROM [support].[dbo].[CommunicationEmail]
WHERE CommunicationId = 183

PRINT(@text)


-- Option 2, in case of very long texts

DECLARE @limit as int,
    	@charLen as int,
    	@current as int,
    	@chars as varchar(8000)

SET @limit = 8000

SELECT  TOP 1 @charLen = DATALENGTH ([FopWithGiro])
FROM    [outerface].[dbo].[Invoice]
where [InvoiceDraftId] = 122869

SET @current = 1

WHILE @current < @charLen
BEGIN
    SELECT	TOP 1 @chars = SUBSTRING([FopWithGiro],@current,@limit)
    FROM	[outerface].[dbo].[Invoice]
    where [InvoiceDraftId] = 122869
    PRINT @chars

    SET @current = @current + @limit
END
```
###### populate table based on static data and another table
```sql
INSERT INTO [webhusetlogin].[dbo].[EmployeeGroupLine]
	SELECT EmployeeId, EmployeeGroupId
	FROM [webhusetlogin].[dbo].[Employee]
	CROSS JOIN (
		SELECT 1400 AS EmployeeGroupId
		UNION SELECT 1401
	) Acl
```
###### Using TOP with variable
```sql
USE AdventureWorks;
GO
DECLARE @p AS int;
SELECT @p=10
SELECT TOP(@p) *
FROM HumanResources.Employee;
GO
```
## Regex
###### Any char including multiline
Should use
```regex
\s\S
```
instead of 
```regex
(.|[\r\n])
```
###### Any char including multiline
Non-greedy quantifier(?)
```regex
<test>.*?</test>
```
will match first tag instead of the whole paragraph in
```
<test>12345</test><test>2</test><test>3</test>
```
## Unix
###### delete a folder with subfolders
```bash
rm -rf folder_name
```
###### delete symlink
```bash
rm linkname
```
or
```bash
unlink linkname
```
###### list folder info
```bash
ls -la /var/lib/tomcat7/webapps/ 
```
###### Kill ethernet connection
```bash
ifconfig eth1 down
```
## Git
###### Clone specific branch
```bash
git clone -b my-branch git@github.com:user/myproject.git
```
###### If you don't care about any local changes and just want a copy from the repo
```bash
git reset --hard HEAD
git pull
```
###### Remove local changes
```bash
git stash save --keep-index
git stash drop
```
###### Create a new branch
```bash
git checkout -b feature-branch-name source_branch_name
```
###### Push newly created branch to origin
```bash
git push -u origin feature-branch-name
```
###### Push changes to a remote repo
```bash
git push origin CORE-734_second_attempt
```
###### Staging
```bash
git status
git add <some-file>
git commit
```
###### Stage all pending changes
```bash
git add .
```
###### Retrieve and switch to a remote branch. Get latest changes.
```bash
git fetch && git checkout CORE-723-incorrect-handling-of-enable_websso
git pull
```
###### Make git ignore "x" file permissions bit
```bash
git config core.filemode false
```
###### See modifications
```bash
git diff file_path
```
###### Revert to previous commit. Works with local commits.
```bash
git reset --hard
```
###### Merge
```bash
git checkout master
git merge feature_branch_name
// fix all the merge conflicts
git add .
// use default message for commit - it's good enough
git commit
git push origin master
```
###### Revert file changes.
```bash
git checkout filename
```
###### Get latest changes and preserve current work.
```bash
git stash
git pull
git stash pop
```
###### Unstage a file.
```bash
git reset fil_name
```
###### If you screwed to push changes, create a separate branch, create a pull request from it. For existing branch do.
```bash
git reset --hard commit_id
```
###### Remove local copies of no longer existing remote branches
```bash
git remote prune origin
```
###### Remove local(not remote) branch
```bash
git branch -d the_local_branch
```
