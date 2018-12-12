# Final project - TRUTH_TIME_CT

## Using this technologies:
* MySql
* Web api
* WinForms
* Angular

## System diagram:
//לשים

***
## Web api
### Models
#### BasicModels
* DailyPresence:
    * IdDaliyPresence - int - key - 
    * EndDatePresence - DateTime - [EndDatePresenceAttribute]
    * StartDatePresence - DateTime - [StartDatePresenceAttribute]
    * IdUserProjectFK - int
* Project:
    * IdProject - int - key 
    * ProjectName - string - minLength: 5, maxLength:20, reqiered , UniqueProjectNameAttribute
    * ClientName - string - minLength: 5, maxLength:20, reqiered
    * IdTeamLeader - int 
    * HoursForDevelopers - double
    * HoursForQA - double
    * HoursForUI_UX - double
    * Active - bool
    * StartDate - DateTime
    * EndDate - DateTime
* StatusUser:
    * IdStatus - int - key
    * StatusName - string -  MinLength(5),MaxLength(20),
* User:
    * IdUser - int - key
    * UserName - string - MinLength(5),MaxLength(20),Required
    * Password - string - [MinLength(64), MaxLength(64)], [UserPasswordAttribute],Required
    * IdStatus - int - Required
    * SumHours - double ,0
    * IdTeamLeader - int 
    * EmailUser - string - EmailAddress
    * IsActive - bool -true
    * ComputerIp - string - 0
* UserProject:
     * IdUserProject - int
     * HoursProjectUser - int
     * IdUser - int
     * IdProject - int
#### HelpModels

* EmailSent:
    * NameUser - string 
    * Subject - string
    * Body - string
    * Password - string - default value:"!qaz!qaz"
    * fromAddress - string - default value:"user.seldat@gmail.com";
    * ToAddress - string
    * Body - string 
* HoursOfUserProjectByDays:
    * Day - string
    * hours - double
* NameUserAndSumHours:
    * IdUser - int 
    * IdProject - int
    * NameUser - string 
    * ProjectName - string
    * IdTeamLeader - int 
    * NameTeamLeader - string
    * SumHours - double 
    * SumHoursHadToDo - double
    * MonthOfDailyPresence - int
* ProjectWithUserPermissions:
    * Project - Project
    * UserPerminissions - List<User>
* UserHelp:
     * UserName - string - MinLength(5), MaxLength(20),Required
     * Password - string - MinLength(64), MaxLength(64),Required
     * ComputerIP - string - default value="0"
   * UserIPComputer:
      * IPComputer - string - default value="0"
* UserProjectHelp:
     * IdUserProject - int
     * HoursProjectUser - int
     * IdUser - int
     * NameTeamLeader - string
     * EndDate - DateTime
     * StartDate - DateTime
     * IdProject - int
     * NameProject - string
     * NameUser - string
     * TimeLeft - double
     * IdStatusUser - int
     * IdDaliyPresence - int
   
### Controllers
* DailyPresenceController:
    * Post - adding task to user - Adding start date to the task of user 
    * Put - update task - Update the end date of task of user
    * Get - getNamesProjectsAndIdUPTodayForUser
    * Get - GetDailyPresenceThatNotUpdated- return dailyPresence that not updated in endDate
* ProjectsController:
    * GET - get all projects 
    * POST - add project
    * Put -  update project 
    * Get - GetAllProjectsUnderTheDirectionOfTheTeamLeader- return all project under teamLeader  
    * Get - GetHoursThatUsersWorkedOfProject - return how many hours the users worked on spesipic project
    * Get - GetHoursWorkedOnProjectByDays - get dictionary of days and the hours that worked in this day on any project
    * Get - AllHoursThatWorkedUnderSpecipicTeamLeaderByMonth - All Hours ThatWorked Under Specipic TeamLeader By Month
* ReportsController:
    * GET -  ReportsForProject - return all data that connection to project
* StatusUsersController:
    * Get - get all status   
* UsersController:
    * GET - get all users 
    * POST - add user
    * Put - update user 
    * Get - GetAllTeamLeaders
    * Post - Login - login to system by user name and password
    * Post - SentMail - from worker to manager
    * Post - GetUserByComputer - get user by his ip computer in order to otomatic enterance
* UsersProjectsController:
    * GET - get all tasks of user 
    * Get - GetAllUserProjectUnderTeamLeaderWithNames - return all details of users and UsersProjects that under specific teamleader
    * Put - SetAllUsersProjects - update hours of all user project in the list 
    * Get - AllDetailsUserProjectOfSpecipicUser - return all deatails user project under specific user
    * Post - CreateUsersProjectListToAllEditUserThatChangeTeamLeader - all user that their team leader was changed needed to removing of              all preview team leader 
        //after needed update user and adding new list of user project that under his team leader
    * Post - AddUsersPermissionToProject - add user project to all user that sent
***
# Code of each tier

### DAL
```csharp
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace _00_DAL
{
    public static class DBUse
    {
      
        static MySqlConnection Connection = new MySqlConnection
        ("SERVER=127.0.0.1;PORT=3306;UID=root;persistsecurityinfo=True;DATABASE=truth_time_ct;SslMode=none");
        public static int? RunNonQuery(string query)
        {
            try
            {
                Connection.Open();
                MySqlCommand command = new MySqlCommand(query, Connection);
                return command.ExecuteNonQuery();
            }
            catch (Exception)
            {
                return null;
            }
            finally
            {
                if (Connection.State != System.Data.ConnectionState.Closed)
                {
                    Connection.Close();
                }
            }

        }
        public static object RunScalar(string query)
        {
            try
            {
                Connection.Open();
                MySqlCommand command = new MySqlCommand(query, Connection);
                return command.ExecuteScalar();
            }
            catch (Exception)
            {
                return null;
            }
            finally
            {
                if (Connection.State != System.Data.ConnectionState.Closed)
                {
                    Connection.Close();
                }
            }

        }
        public static List<T> RunReader<T>(string query, Func<MySqlDataReader, List<T>> func)
        {
            try
            {
                Connection.Open();
                MySqlCommand command = new MySqlCommand(query, Connection);
                MySqlDataReader reader = command.ExecuteReader();
                return func(reader);
            }
            catch (Exception e)
            {
                return null;
            }
            finally
            {
                if (Connection.State != System.Data.ConnectionState.Closed)
                {
                    Connection.Close();
                }
            }

        }
        public static Dictionary<T, double> RunReaderDictionary<T>(string query, Func<MySqlDataReader, Dictionary<T, double>> func)
        {
            try
            {
                Connection.Open();
                MySqlCommand command = new MySqlCommand(query, Connection);
                MySqlDataReader reader = command.ExecuteReader();
                return func(reader);
            }
            catch (Exception)
            {
                return null;
            }
            finally
            {
                if (Connection.State != System.Data.ConnectionState.Closed)
                {
                    Connection.Close();
                }
            }

        }


    }
}
csharp
```

### BOL
```csharp
using _01_BOL.Validations;
using System.ComponentModel.DataAnnotations;
using System;
namespace _01_BOL
{
    public class DailyPresence
    {
        [Key]
        public int IdDaliyPresence { get; set; }
        [EndDatePresenceAttribute]
        [Required(ErrorMessage = "EndDate Presence is Required")]
        public DateTime EndDatePresence { get; set; }
        [StartDatePresenceAttribute]
        [Required(ErrorMessage = "StartDate Presence is Required")]
        public DateTime StartDatePresence { get; set; }
        [Required(ErrorMessage = "Id User Project is Required")]
        public int IdUserProjectFK { get; set; }

    }
}

```
```csharp
using _01_BOL.Validations;
using System;
using System.ComponentModel.DataAnnotations;

namespace _01_BOL
{
    public class Project
    {
        [Key]
        public int IdProject { get; set; }
        [Required(ErrorMessage = "ProjectName is Required")]
        [MinLength(5),MaxLength(20)]
        [UniqueProjectNameAttribute]
        public string ProjectName { get; set; }
        [MinLength(5), MaxLength(20)]
        public string ClientName { get; set; }
        [Required(ErrorMessage = "IdTeamLeader is Required")]
        public int IdTeamLeader { get; set; }
        public double HoursForDevelopers { get; set; }
        public double HoursForQA { get; set; }
        public double HoursForUI_UX { get; set; }
        public bool Active { get; set; } = true;
        [Required(ErrorMessage = "StartDate Project is Required")]
        public DateTime StartDate { get; set; }
        [EndDateProjectAttribute]
        [Required(ErrorMessage = "EndDate Project is Required")]
        public DateTime EndDate { get; set; }

    }
}
```
```csharp
using System.ComponentModel.DataAnnotations;

namespace _01_BOL
{
    public class StatusUser
    {
        [Key]
        public int IdStatus { get; set; }
        [Required(ErrorMessage = "Status Name is Required")]
        public string StatusName { get; set; }
    }
}

```
```csharp
using _01_BOL.Validations;
using System.ComponentModel.DataAnnotations;

namespace _01_BOL
{
    public class User
    {
        [Key]
        public int IdUser { get; set; }
        [MinLength(5),MaxLength(20)]
        [Required(ErrorMessage = "User Name is Required")]
        public string UserName { get; set; }
        [MinLength(64), MaxLength(64)]
        [UserPasswordAttribute]
        [Required(ErrorMessage = "Password is Required")]
        public string Password { get; set; }
        [Required(ErrorMessage = "Id Status is Required")]
        public int IdStatus { get; set; }

        public double? SumHours { get; set; } = 0;
        public int IdTeamLeader { get; set; }
        [Required(ErrorMessage = "Email is Required")]
        [EmailAddress]
        public  string EmailUser { get; set; }
        public bool IsActive { get; set; } = true;
        public string ComputerIp { get; set; } = "0";
    }
}
```
```csharp
using System.ComponentModel.DataAnnotations;

namespace _01_BOL
{
    public class UserProject
    {
        [Key]
        public int IdUserProject { get; set; }
        [Required(ErrorMessage = "Hours for User Project is Required")]
        public int HoursProjectUser { get; set; }
        [Required(ErrorMessage = "Id User is Required")]
        public int IdUser { get; set; }
        [Required(ErrorMessage = "Id Project is Required")]
        public int IdProject { get; set; }
       
    }
}
```
```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Net.Mail;

namespace _01_BOL.HelpModels
{
    public class EmailSent
    {
        public string NameUser { get; set; }
        public string Subject { get; set; }
        public string Body { get; set; }
        public string Password { get; } = "!qaz!qaz";//ConfigurationManager.AppSettings["password"];
        public string fromAddress { get; } = "user.seldat@gmail.com";
        public string ToAddress { get; set; }
        //change 18/11
        public List<string> Attachments { get; set; } = new List<string>();
    }
}
```
```csharp
namespace _01_BOL.HelpDepartment
{
    public class HoursOfUserProjectByDays
    {
        public string Day;
        public double hours;
    }
}
```
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace _01_BOL.HelpModels
{ 
    public class NameUserAndSumHours
    {      
        public int IdUser { get; set; }
        public int IdProject { get; set; }
        public string NameUser { get; set; }
        public string ProjectName { get; set; }
        public int IdTeamLeader { get; set; }
        public string NameTeamLeader { get; set; }
        public double SumHours { get; set; }
        public double SumHoursHadToDo { get; set; }
        public int MonthOfDailyPresence { get; set; }
    }
}
```
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace _01_BOL.HelpModels
{
  public  class ProjectWithUserPermissions
    {
       public Project Project { get; set; }
        public List<User> UserPerminissions { get; set; }
    }
}
```
```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace _01_BOL.HelpModels
{
    public class UserHelp
    {
        [MinLength(5), MaxLength(20)]
        [Required(ErrorMessage = "User Name is Required")]
        public string  UserName{ get; set; }
        [MinLength(64), MaxLength(64)]
        [Required(ErrorMessage = "Password is Required")]
        public string  Password{ get; set; }
        public string ComputerIP { get; set; } = "0";
    }
}
```
```csharp
namespace _01_BOL.HelpModels
{
    public class UserIPComputer
    {
        public string IPComputer { get; set; } = "0";
    }
}
```
```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace _01_BOL.HelpDepartment
{
    public class UserProjectHelp
    {
        public int IdUserProject { get; set; }
        public int HoursProjectUser { get; set; }
        public int IdUser { get; set; }
        public string NameTeamLeader { get; set; }
        public DateTime EndDate { get; set; }
        public DateTime StartDate { get; set; }
        public int IdProject { get; set; }
        public string NameProject { get; set; }
        public string NameUser { get; set; }
        public double TimeLeft { get; set; }
        public int IdStatusUser { get; set; }
        public int IdDaliyPresence { get; set; }
    }
}
```
```csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace _01_BOL.Validations
{
    public class EndDatePresenceAttribute : ValidationAttribute
    {
        override protected ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            DateTime startDatePresence = (validationContext.ObjectInstance as DailyPresence).StartDatePresence;
            DateTime endDatePresence = (DateTime)value;
            return ((endDatePresence - startDatePresence).TotalHours >= 0) ? null :
                new ValidationResult("EndDateTime has to be bigger than StartDateTime");
        }

    }
}
```
```csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace _01_BOL.Validations
{
    class EndDateProjectAttribute: ValidationAttribute
    {
        override protected ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            DateTime startDateProject = (validationContext.ObjectInstance as Project).StartDate;
            DateTime endDateProject = (DateTime)value;
            return ((endDateProject - startDateProject).TotalHours >= 0) ? null :
                new ValidationResult("EndDateTime has to be bigger than StartDateTime");
        }
    }
}
```
```csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace _01_BOL.Validations
{

    public class StartDatePresenceAttribute : ValidationAttribute
    {
        override protected ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            DateTime startPresence = (DateTime)value;
            return ((startPresence - DateTime.Now).TotalHours <= 0) ? null :new ValidationResult("DateTime has to be smaller than    today");
        }

    }
}
```
```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Reflection;

namespace _01_BOL.Validations
{
    class UniqueProjectNameAttribute : ValidationAttribute
    {

        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            ValidationResult validationResult = ValidationResult.Success;
            try
            {
                int idProject = (validationContext.ObjectInstance as Project).IdProject;
                string nameProject = value.ToString();
                Assembly assembly = Assembly.LoadFrom(@"C:\Users\user1\Desktop\חדש\TruthTimeCT\02_BLL\bin\Debug\02_BLL.dll");
                Type projectServiceType = assembly.GetTypes().First(t => t.Name.Equals("LogicProjects"));
                MethodInfo getAllUsersMethod = projectServiceType.GetMethods().First(m => m.Name.Equals("GetAllProjects"));
                List<Project> projects = getAllUsersMethod.Invoke(Activator.CreateInstance(projectServiceType), new object[] { }) as List<Project>;
                if (projects == null) return validationResult;//there are not project yet
                bool isUnique = projects.Any(project => project.ProjectName==nameProject && project.IdProject != idProject) == false;
                if (isUnique == false)
                {
                    ErrorMessage = "project name must be unique";
                    validationResult = new ValidationResult(ErrorMessageString);
                }
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return validationResult;
        }

    }
}
```
```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Reflection;

namespace _01_BOL.Validations
{
    class UserPasswordAttribute : ValidationAttribute
    {

        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            ValidationResult validationResult = ValidationResult.Success;
            try
            {
                int idUser = (validationContext.ObjectInstance as User).IdUser;
                string userPassword = value.ToString();
                Assembly assembly = Assembly.LoadFrom(@"C:\Users\user1\Desktop\חדש\TruthTimeCT\02_BLL\bin\Debug\02_BLL.dll");
                Type userServiceType = assembly.GetTypes().First(t => t.Name.Equals("LogicUsers"));
                MethodInfo getAllUsersMethod = userServiceType.GetMethods().First(m => m.Name.Equals("GetAllUsers"));
                List<User> users = getAllUsersMethod.Invoke(Activator.CreateInstance(userServiceType), new object[] { }) as List<User>;
                bool isUnique = users.Any(user => user.Password == (userPassword) && user.IdUser != idUser) == false;
                if (isUnique == false)
                {
                    ErrorMessage = "user password must be unique";
                    validationResult = new ValidationResult(ErrorMessageString);
                }
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return validationResult;
        }

    }
}
```
### BLL
```csharp
using _00_DAL;
using _01_BOL;
using _01_BOL.HelpDepartment;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace _02_BLL
{
    public static class LogicDailyPresence
    {
        /// <summary>
        /// update Daily presence when user begin work and after when he finish
        /// the function check if create new daliy presence or just to update
        /// </summary>
        /// <param name="daliyPresence">all data on daily Presence</param>
        /// <returns>true  when update succeeded</returns>
        public static bool UpdateDailyPresence(DailyPresence daliyPresence)
        {
            daliyPresence.StartDatePresence = daliyPresence.StartDatePresence.ToLocalTime();
            daliyPresence.EndDatePresence = daliyPresence.EndDatePresence.ToLocalTime();
            //check if first time or only update end time
            string formatForMySqlEndDatePresence = daliyPresence.EndDatePresence.ToString("yyyy-MM-dd HH:mm:ss");
            string query = $"SELECT distinct d2.idDaliyPresence FROM truth_time_ct.daily_presence as d2 where d2.idUserProject ={ daliyPresence.IdUserProjectFK} and d2.startDatePresence = d2.endDatePresence;";
            string rezult = DBUse.RunScalar(query).ToString();
            query = $" UPDATE truth_time_ct.daily_presence as d SET d.endDatePresence = '{ formatForMySqlEndDatePresence}'" +
                    $" where d.idDaliyPresence = {rezult}";
            return DBUse.RunNonQuery(query) == 1;
        }
        public static bool AddDailyPresence(DailyPresence daliyPresence)
        {

            daliyPresence.StartDatePresence = daliyPresence.StartDatePresence.ToLocalTime();
            daliyPresence.EndDatePresence = daliyPresence.EndDatePresence.ToLocalTime();
            string formatForMySqlEndDatePresence = daliyPresence.EndDatePresence.ToString("yyyy-MM-dd HH:mm:ss");
            string formatForMySqlStartDatePresence = daliyPresence.StartDatePresence.ToString("yyyy-MM-dd HH:mm:ss");
            string query = $"INSERT INTO truth_time_ct.daily_presence VALUES ({daliyPresence.IdDaliyPresence},'{formatForMySqlEndDatePresence}','{formatForMySqlStartDatePresence}'" +
                $",{daliyPresence.IdUserProjectFK})";
            return DBUse.RunNonQuery(query) == 1;
        }
        /// <summary>
        /// the function returns all the user project that their date in the range of projects includes today
        /// </summary>
        /// <param name="IdUser">for which user get actual projects</param>
        /// <returns>return all list project</returns>
        public static List<UserProjectHelp> GetNamesProjectsAndIdUPTodayForUser(int IdUser)
        {
            string query = $"SELECT idUserProject,projectName FROM truth_time_ct.users_projects as up join truth_time_ct.projects as p " +
                $"on p.idProject=up.idProject where up.idUser={IdUser} and up.idProject " +
                $"in(SELECT idProject from truth_time_ct.projects where startDate <= DATE(NOW()) and endDate>= DATE(NOW()) and idProject " +
                $"in (select idProject from truth_time_ct.users_projects where idUser = {IdUser}))";
            Func<MySqlDataReader, List<UserProjectHelp>> func = (reader) =>
            {
                List<UserProjectHelp> TasksForTodayOfCurrentUsers = new List<UserProjectHelp>();
                while (reader.Read())
                {
                    TasksForTodayOfCurrentUsers.Add(new UserProjectHelp()
                    {
                        IdUserProject = int.Parse(reader[0].ToString()),
                        NameProject = reader[1].ToString()
                    });
                }
                return TasksForTodayOfCurrentUsers;
            };
            return DBUse.RunReader(query, func);
        }
        public static List<UserProjectHelp> GetDailyPresenceThatNotUpdated(int idUser)
        {
            string query = $"select d.*,up.hoursProjectUser,up.idProject,up.idUser,p.projectName from daily_presence d join users_projects up on d.idUserProject=up.idUserProject join projects p on p.idProject=up.idProject where up.idUser = {idUser} and d.startDatePresence = d.endDatePresence and up.idProject = p.idProject";
            Func<MySqlDataReader, List<UserProjectHelp>> func = (reader) =>
            {
                List<UserProjectHelp> userProjectHelps = new List<UserProjectHelp>();
                while (reader.Read())
                {
                    userProjectHelps.Add(new UserProjectHelp()
                    {
                        IdDaliyPresence = (int)reader[0],
                        EndDate = (DateTime)reader[1],
                        StartDate = (DateTime)reader[2],
                        IdUserProject = (int)reader[3],
                        HoursProjectUser = (int)reader[4],
                        IdProject = (int)reader[5],
                        IdUser = (int)reader[6],
                        NameProject = (string)reader[7],
                    });

                };
                return userProjectHelps;

            };
            var listUserProjectHelp = DBUse.RunReader(query, func);
            return listUserProjectHelp;

        }
    }
}
```
```csharp
using _01_BOL.HelpModels;
using System;
using System.IO;
using System.Net;
using System.Net.Mail;

namespace _02_BLL.Logic
{
    public class LogicEmail
    {
        /// <summary>
        /// sent mail to manager
        /// </summary>
        /// <param name="emailSent">All contents of the email</param>
        /// <returns>if mail was sent</returns>
        public static bool SentMail(EmailSent emailSent)
        {
            if (emailSent.ToAddress == null)//need to manager else this for users
                emailSent.ToAddress = LogicUsers.GetAllSpesificStatus("manager")[0].EmailUser;
            MailMessage mail = new MailMessage(emailSent.fromAddress, emailSent.ToAddress);
            SmtpClient client = new SmtpClient();
            client.Port = 25;
            client.DeliveryMethod = SmtpDeliveryMethod.Network;
            client.UseDefaultCredentials = true;
            client.Credentials = new NetworkCredential(emailSent.fromAddress, emailSent.Password);
            client.EnableSsl = true;
            client.Host = "smtp.gmail.com";
            mail.Subject = emailSent.Subject + " is sent from " + emailSent.NameUser;
            mail.Body = emailSent.Body;
            foreach (string filePath in emailSent.Attachments)
            {
                string fileName = Path.GetFileName(filePath);
                mail.Attachments.Add(new Attachment(filePath));
            }
            try
            {
                client.Send(mail);
                mail.IsBodyHtml = false;
                return true;
            }
            catch (Exception)
            {
                return false;
            }
        }
    }
}

```
```csharp
using _00_DAL;
using _01_BOL;
using _01_BOL.HelpDepartment;
using _01_BOL.HelpModels;
using _02_BLL.Logic;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;
using System.Linq;

namespace _02_BLL
{
    public class LogicProjects
    {
        //return all projects
        public static List<Project> GetAllProjects()
        {
            string query = $"SELECT * FROM truth_time_ct.projects p where p.active=1";

            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> Projects = new List<Project>();
                while (reader.Read())
                {
                    Projects.Add(new Project
                    {
                        IdProject = (int)reader[0],
                        ProjectName = (string)reader[1],
                        ClientName = (string)reader[2],
                        IdTeamLeader = (int)reader[3],
                        StartDate = (DateTime)reader[4],
                        EndDate = (DateTime)reader[5],
                        HoursForDevelopers = (double)reader[6],
                        HoursForQA = (double)reader[7],
                        HoursForUI_UX = (double)reader[8],
                        Active = reader.GetBoolean(9)

                    });
                }
                return Projects;
            };

            return DBUse.RunReader(query, func);
        }
        //return project id by nameProject
        public static string GetProjectId(string name)
        {
            string query = $"SELECT idProject FROM truth_time_ct.projects WHERE projectName='{name}'";
            return DBUse.RunScalar(query).ToString();
        }
        //update project
        public static bool UpdateProject(Project project)
        {
            string query = $"UPDATE truth_time_ct.projects SET projectName='{project.ProjectName}'," +
                $"idTeamLeader={project.IdTeamLeader},active={project.Active},startDate='{project.StartDate}',endDate='{project.EndDate}',hoursForDevelopers={project.HoursForDevelopers},hoursForQA={project.HoursForQA},hoursForUI_UX={project.HoursForUI_UX},clientName={project.ClientName} WHERE idProject = {project.IdProject}";
            return DBUse.RunNonQuery(query) == 1;
        }
        //add project
        public static bool AddProject(Project project, ProjectWithUserPermissions projectWithUserPermissions)
        {
            //projectWithUserPermissions.Project.StartDate = projectWithUserPermissions.Project.StartDate.ToLocalTime();
            //projectWithUserPermissions.Project.EndDate = projectWithUserPermissions.Project.EndDate.ToLocalTime();
            string formatForMySqlStartDate = projectWithUserPermissions.Project.StartDate.ToString("yyyy-MM-dd");
            string formatForMySqlEndDate = projectWithUserPermissions.Project.EndDate.ToString("yyyy-MM-dd");
            bool isWorked = false;
            string query = $"INSERT INTO truth_time_ct.projects VALUES (0,'{project.ProjectName}'," +
                $"'{project.ClientName}',{project.IdTeamLeader},'{formatForMySqlStartDate}','{formatForMySqlEndDate}',{project.HoursForDevelopers},{project.HoursForQA},{project.HoursForUI_UX},{project.Active})";
            isWorked = DBUse.RunNonQuery(query) == 1;
            if (isWorked)
            {
                int idProject = int.Parse(GetProjectId(project.ProjectName));
                LogicUserProject.CreateUsersProjectList(idProject, project.IdTeamLeader);
                LogicUserProject.AddUsersPermission(projectWithUserPermissions.UserPerminissions, idProject);
            }
            return isWorked;
        }
        //return all projects under teamLeader
        public static List<Project> GetProjectsUnderTheDirectionOfTheTeamLeader(int IdTeamLeader)
        {
            //select all projects under teamLeader

            string query = $"SELECT * FROM truth_time_ct.projects WHERE idTeamLeader={IdTeamLeader}";
            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> Projects = new List<Project>();
                while (reader.Read())
                {
                    Projects.Add(new Project
                    {
                        IdProject = (int)reader[0],
                        ProjectName = (string)reader[1],
                        ClientName = (string)reader[2],
                        IdTeamLeader = (int)reader[3],
                        StartDate = (DateTime)reader[4],
                        EndDate = (DateTime)reader[5],
                        HoursForDevelopers = (double)reader[6],
                        HoursForQA = (double)reader[7],
                        HoursForUI_UX = (double)reader[8],
                        Active = reader.GetBoolean(9)

                    });
                }
                return Projects;
            };

            return DBUse.RunReader(query, func);
        }
        //return project by id project
        public static Project GetProjectByIdProject(int idProject)
        {
            string query = $"SELECT * FROM truth_time_ct.projects WHERE idProject={idProject}";
            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> Projects = new List<Project>();
                while (reader.Read())
                {
                    Projects.Add(new Project
                    {
                        IdProject = (int)reader[0],
                        ProjectName = (string)reader[1],
                        ClientName = (string)reader[2],
                        IdTeamLeader = (int)reader[3],
                        StartDate = (DateTime)reader[4],
                        EndDate = (DateTime)reader[5],
                        HoursForDevelopers = (double)reader[6],
                        HoursForQA = (double)reader[7],
                        HoursForUI_UX = (double)reader[8],
                        Active = reader.GetBoolean(9)

                    });
                }
                return Projects;
            };
            Project myProject = DBUse.RunReader(query, func)[0];
            return myProject;
        }
        //return hours that defined for project
        public static int GetHoursOfProject(int idProject)
        {
            string query = $"SELECT hoursForProject FROM truth_time_ct.projects WHERE idProject={idProject}";
            return int.Parse(DBUse.RunScalar(query).ToString());
        }
        //return how many hours the users worked on spesipic project   
        public static List<NameUserAndSumHours> GetNamesAndHoursThatWorkedOfProject(int idProject)
        {
            double sumHours = 0;
            //get the hours by dateDiff between start and end date
            string query = $"SELECT userName, sum(sum_hours) from(SELECT u.userName, " +
                $"idProject, TIMESTAMPDIFF(MINUTE, startDatePresence, endDatePresence) /60 as sum_hours " +
                $"FROM truth_time_ct.daily_presence d right join truth_time_ct.users_projects up on d.idUserProject = " +
                $"up.idUserProject right join truth_time_ct.users u on u.idUser = up.idUser WHERE  idProject = {idProject}) as table1 group by " +
                $"userName UNION SELECT userName, sum(sum_hours) from(SELECT u.userName, idProject, TIMESTAMPDIFF(MINUTE, startDatePresence, " +
                $"endDatePresence) /60 as sum_hours FROM truth_time_ct.daily_presence d left join truth_time_ct.users_projects up on d.idUserProject = up.idUserProject left join truth_time_ct.users u on u.idUser = up.idUser WHERE  idProject = {idProject}) as table1 group by userName";
            Func<MySqlDataReader, List<NameUserAndSumHours>> func = (reader) =>
            {
                List<NameUserAndSumHours> NamesAndSumHours = new List<NameUserAndSumHours>();
                while (reader.Read())
                {
                    sumHours = 0;
                    if (reader[1].ToString() != "")
                        sumHours = double.Parse(reader[1].ToString());
                    NamesAndSumHours.Add(

                        new NameUserAndSumHours()
                        {
                            NameUser = reader[0].ToString(),
                            SumHours = sumHours
                        }
                        );
                }
                NamesAndSumHours.Add(
                                     new NameUserAndSumHours()
                                     {
                                         NameUser = "not yet",
                                         SumHours = NamesAndSumHours.Sum(s => s.SumHours)
                                     }
                                     );
                return NamesAndSumHours;
            };

            return DBUse.RunReader(query, func);

        }
        //return dictionary of userProject and the hours that the user worked to this userProject; 
        public static Dictionary<UserProject, double> GetDictionaryOfHoursThatUserWorkedOnProjectInPrecent(int idUser)
        {
            //for user select all data of UsersProjects of him with the time that he worked for that project;
            string query = $"SELECT *,sum(SELECT DATEDIFF(hour, startDatePresence, endDatePresence) FROM truth_time_ct.daily_presence WHERE idProject=up.idProject and idUser={idUser}) from truth_time_ct.users_projects up WHERE up.idUser={idUser} Group By up.idUserProject,up.hoursProjectUser,up.idUser,up.idProject";
            Func<MySqlDataReader, Dictionary<UserProject, double>> func = (reader) =>
            {
                Dictionary<UserProject, double> myDictionary = new Dictionary<UserProject, double>();
                while (reader.Read())
                {
                    myDictionary.Add(new UserProject
                    {
                        IdUserProject = (int)reader[0],
                        HoursProjectUser = (int)reader[1],
                        IdUser = (int)reader[2],
                        IdProject = (int)reader[3]
                    }, (int)reader[4]);
                }
                return myDictionary;
            };

            return DBUse.RunReaderDictionary(query, func);
        }
        //get dictionary of days and the hours that users worked in this day on any project
        public static List<HoursOfUserProjectByDays> GetHoursWorkedOnProjectByDays(int idUser, int month)
        {
            //select days and for every day the sum of hours that the users worked on specipic project on specipic month
            string query = $"SELECT  dayofmonth(startDatePresence),TIME_TO_SEC(timediff(endDatePresence,startDatePresence))/3600 FROM daily_presence " +
                $"WHERE idUserProject IN(SELECT idUserProject FROM truth_time_ct.users_projects WHERE idUser = {idUser}) AND MONTH(startDatePresence)={month}";
            Func<MySqlDataReader, List<HoursOfUserProjectByDays>> func = (reader) =>
              {
                  List<HoursOfUserProjectByDays> DailySupply = new List<HoursOfUserProjectByDays>();
                  while (reader.Read())
                  {
                      DailySupply.Add(
                          new HoursOfUserProjectByDays()
                          {
                              Day = reader[0].ToString(),
                              hours = double.Parse(reader[1].ToString())
                          }
                          );
                  }
                  return DailySupply;
              };

            return DBUse.RunReader(query, func);
        }
        //get dictionary of users and hours that worked
        public static Dictionary<UserProject, double> GetUsersAndHoursThatWorkedOnProject(int idProject)
        {

            List<User> UsersOfProject = LogicUsers.GetUsersOfProject(idProject);
            Dictionary<UserProject, double> UsersAndHoursWorkedOnProject = new Dictionary<UserProject, double>();
            foreach (User user in UsersOfProject)
            {
                UserProject userProject = LogicUserProject.GetSpesipicUserProject(user.IdUser, idProject);
                UsersAndHoursWorkedOnProject.Add(userProject, GetDictionaryOfHoursThatUserWorkedOnProjectInPrecent(user.IdUser)[userProject]);
            }
            return UsersAndHoursWorkedOnProject;
        }
        //return all projects under specipic teamLeader
        public static List<Project> GetProjectsUnderTeamLeader(int idTeamLeader)
        {
            string query = $"SELECT * FROM truth_time_ct.Projects WHERE idTeamLeader={idTeamLeader}";

            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> Projects = new List<Project>();
                while (reader.Read())
                {
                    Projects.Add(new Project
                    {
                        IdProject = (int)reader[0],
                        ProjectName = (string)reader[1],
                        ClientName = (string)reader[2],
                        IdTeamLeader = (int)reader[3],
                        StartDate = (DateTime)reader[4],
                        EndDate = (DateTime)reader[5],
                        HoursForDevelopers = (double)reader[6],
                        HoursForQA = (double)reader[7],
                        HoursForUI_UX = (double)reader[8],
                        Active = reader.GetBoolean(9)

                    });
                }
                return Projects;
            };
            return DBUse.RunReader(query, func);
        }
        //all Hours That Worked Under Specipic teamLeader By month
        public static List<NameUserAndSumHours> AllHoursThatWorkedUnderSpecipicTeamLeaderByMonth(int idTeamLeader, int month)
        {
            string query = $"select sum(allocated) as allocated, userName, sum(sum_hours) as sum_hours from(SELECT allocated, userName, sum(sum_hours) as sum_hours " +
                $"from(SELECT up.hoursProjectUser as allocated, up.idUserProject, u.userName, p.idProject, TIMESTAMPDIFF(MINUTE, startDatePresence, endDatePresence) / 60 as sum_hours FROM truth_time_ct.daily_presence d  right join truth_time_ct.users_projects up " +
                $"on d.idUserProject = up.idUserProject  join truth_time_ct.users u on u.idUser = up.idUser   join truth_time_ct.projects p on p.idProject = up.idProject " +
                $"WHERE p.idTeamLeader = {idTeamLeader} and u.idTeamLeader = {idTeamLeader} and  month(d.startDatePresence) = {month}) as table1 group by idUserProject " +
                $"union select(up.hoursProjectUser) as allocated, u.userName,0 as sum_hours from users u left join users_projects up on u.idUser = up.idUser where u.idTeamLeader = {idTeamLeader} " +
                $"and up.idUserProject not in(select idUserProject from daily_presence where month(startDatePresence)={month}))as table2 group by userName";
            Func<MySqlDataReader, List<NameUserAndSumHours>> func = (reader) =>
            {
                List<NameUserAndSumHours> NamesAndSumHours = new List<NameUserAndSumHours>();
                while (reader.Read())
                {

                    NamesAndSumHours.Add(

                        new NameUserAndSumHours()
                        {
                            SumHoursHadToDo = double.Parse(reader[0].ToString()),
                            NameUser = reader[1].ToString(),
                            SumHours = double.Parse(reader[2].ToString())
                        }
                        );
                }
                return NamesAndSumHours;
            };

            return DBUse.RunReader(query, func);
        }
        //sent mail for all user that didn't finish their task day before deadline
        public static void AllEmailUserThatNeededReminderAboutDeadline()
        {
            string query = $"SELECT u.userName,p1.projectName, u.emailUser,p1.endDate " +
                $"FROM users u INNER JOIN users_projects up " +
                $"ON u.idUser = up.idUser INNER JOIN projects p1 " +
                $"ON p1.idProject = up.idProject " +
                $"WHERE up.idProject IN " +
                $"( " +
                $"SELECT P.idProject " +
                $"FROM projects P " +
                $"WHERE P.endDate IN (CURDATE(), CURDATE() +INTERVAL 1 DAY) " +
                $")  " +
                $"AND up.idUserProject IN " +
                $"( " +
                $"SELECT UP2.idUserProject " +
                $"FROM users_projects UP2 INNER JOIN daily_presence D " +
                $"ON UP2.idUserProject= D.idUserProject " +
                $"GROUP BY UP2.idUserProject, up2.hoursProjectUser " +
                $"having SUM(time_to_sec( " +
                $"timediff(D.endDatePresence, D.startDatePresence))/ 3600 " +
                $")< up2.hoursProjectUser " +
                $")";
            Func<MySqlDataReader, List<EmailSent>> func = (reader) =>
            {
                List<EmailSent> emails = new List<EmailSent>();
                while (reader.Read())
                {
                    var email = new EmailSent()
                    {
                        NameUser = (string)reader[0],
                        Subject = "Dear " + (string)reader[0],
                        Body = "The project deadline " + (string)reader[1] + " is " + Convert.ToDateTime(reader.GetString(3)),
                        ToAddress = (string)reader[2]
                    };
                    LogicEmail.SentMail(email);
                }
                return emails;
            };
            List<EmailSent> mails = DBUse.RunReader(query, func);
        }
    }
}

```
```csharp
using _00_DAL;
using _01_BOL.HelpModels;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace _02_BLL.Logic
{
    public class LogicReports
    {
        /// <summary>
        /// get all data of all the system
        /// </summary>
        /// <returns>how many hours the users worked for every project</returns>
        public static List<NameUserAndSumHours> ReportsForProject()
        {
            double sumHours = 0;
            int month = 0;
            //get the hours by dateDiff between start and end date
            string query = $"SELECT idUser,idProject,userName,projectName,idTeamLeader,nameTeam, sum(sum_hours),sum(allocate) ,monthOfDailyPresence " +
                $"from(SELECT  u.idUser, up.idProject, u.userName,projectName,u.idTeamLeader,u2.userName as nameTeam, TIMESTAMPDIFF(MINUTE, startDatePresence, endDatePresence) / 60 as sum_hours, up.hoursProjectUser as allocate, month(startDatePresence) as monthOfDailyPresence " +
                $"FROM truth_time_ct.daily_presence d right join truth_time_ct.users_projects up " +
                $"on d.idUserProject = up.idUserProject  join truth_time_ct.users u on u.idUser = up.idUser join projects p on p.idProject=up.idProject " +
                $" join truth_time_ct.users u2 on u2.idUser = u.idTeamLeader" +
                $") as table1 " +
                $"group by idUser,idProject " +
                $"UNION " +
                $"SELECT idUser,idProject,userName,projectName,idTeamLeader,nameTeam, sum(sum_hours) , sum(allocate) , monthOfDailyPresence " +
                $"from( " +
                $"SELECT  u.idUser, up.idProject, u.userName,projectName,u.idTeamLeader,u2.userName as nameTeam, TIMESTAMPDIFF(MINUTE, startDatePresence, endDatePresence) / 60 as sum_hours, up.hoursProjectUser as allocate, month(startDatePresence) as monthOfDailyPresence " +
                $"FROM truth_time_ct.daily_presence d left join truth_time_ct.users_projects up " +
                $"on d.idUserProject = up.idUserProject  join truth_time_ct.users u on u.idUser = up.idUser join projects p on p.idProject=up.idProject " +
                $" join truth_time_ct.users u2 on u2.idUser = u.idTeamLeader" +
                $") as table1 " +
                $"group by idUser,idProject";
            Func<MySqlDataReader, List<NameUserAndSumHours>> func = (reader) =>
            {
                List<NameUserAndSumHours> NamesAndSumHours = new List<NameUserAndSumHours>();
                while (reader.Read())
                {
                    sumHours = 0;
                    month = 0;
                    if (reader[6].ToString() != "")
                        sumHours = double.Parse(reader[6].ToString());
                    if (reader[8].ToString() != "")
                        month = int.Parse(reader[8].ToString());
                    NamesAndSumHours.Add(

                        new NameUserAndSumHours()
                        {
                            IdUser = (int)reader[0],
                            IdProject = (int)reader[1],
                            NameUser = (string)reader[2],
                            ProjectName = (string)reader[3],
                            IdTeamLeader = (int)reader[4],
                            NameTeamLeader = (string)reader[5],
                            SumHours = sumHours,
                            SumHoursHadToDo = double.Parse(reader[7].ToString()),
                            MonthOfDailyPresence = month
                        }
                        );
                }
                return NamesAndSumHours;
            };

            return DBUse.RunReader(query, func);

        }
    }
}
```

### BLL
```csharp
using _00_DAL;
using _01_BOL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace _02_BLL
{
    public static class LogicStatusUsers
    {
        /// <summary>
        /// all status that exit in system
        /// </summary>
        /// <returns>status</returns>
        public static List<StatusUser> GetAllStatusUsers()
        {
            string query = $"SELECT * FROM truth_time_ct.status_users";

            Func<MySqlDataReader, List<StatusUser>> func = (reader) =>
            {
                List<StatusUser> status = new List<StatusUser>();
                while (reader.Read())
                {
                    status.Add(new StatusUser
                    {
                        IdStatus = (int)reader[0],
                        StatusName = (string)reader[1]
                    });
                }
                return status;
            };

            return DBUse.RunReader(query, func);
        }
    }
}
```

### BLL
```csharp
using _00_DAL;
using _01_BOL;
using System;
using System.Collections.Generic;
using MySql.Data.MySqlClient;
using _01_BOL.HelpDepartment;
using System.Linq;

namespace _02_BLL
{
    public class LogicUserProject
    {
        public static List<UserProject> GetAllUserProject()
        {
            string query = $"SELECT * FROM truth_time_ct.users_projects";

            Func<MySqlDataReader, List<UserProject>> func = (reader) =>
            {
                List<UserProject> users_projects = new List<UserProject>();
                while (reader.Read())
                {
                    users_projects.Add(new UserProject
                    {
                        IdUserProject = (int)reader[0],
                        HoursProjectUser = (int)reader[1],
                        IdProject = (int)reader[2],
                        IdUser = (int)reader[3]
                    });
                }
                return users_projects;
            };

            return DBUse.RunReader(query, func);
        }
        public static bool RemoveUserProjectByUserAndProject(int idUser, int idProject)
        {
            string query = $"DELETE FROM truth_time_ct.users_projects WHERE idUser={idUser} and idProject={idProject}";
            return DBUse.RunNonQuery(query) == 1;
        }
        //update hours for  userProject
        public static bool UpdateUserProject(UserProject userProject)
        {
            string query = $"UPDATE truth_time_ct.users_projects SET hoursProjectUser={userProject.HoursProjectUser} WHERE idUserProject = {userProject.IdUserProject}";
            return DBUse.RunNonQuery(query) == 1;
        }
        public static bool AddUserProject(UserProject userProject)
        {
            string query = $"INSERT INTO truth_time_ct.users_projects VALUES (0,{userProject.HoursProjectUser}" +
                $",{userProject.IdProject},{userProject.IdUser})";
            return DBUse.RunNonQuery(query) == 1;
        }
        public static UserProject GetSpesipicUserProject(int idUser, int idProject)
        {
            string query = $"SELECT * FROM truth_time_ct.users_projects where idUser={idUser} and idProject={idProject}";

            Func<MySqlDataReader, List<UserProject>> func = (reader) =>
             {
                 List<UserProject> UserProject = new List<UserProject>();
                 UserProject.Add(new UserProject
                 {
                     IdUserProject = (int)reader[0],
                     HoursProjectUser = (int)reader[1],
                     IdProject = (int)reader[2],
                     IdUser = (int)reader[3]
                 });
                 return UserProject;
             };

            return DBUse.RunReader(query, func)[0];
        }
        //return all deatails of specipic user project
        public static List<UserProjectHelp> AllDetailsUserProjectOfSpecipicUser(int idUser)
        {
            string query = $"" +
                $"SELECT p.projectName,p.startDate,p.endDate ,up.hoursProjectUser,u.userName FROM truth_time_ct.Projects p join truth_time_ct.users_projects up on p.idProject = up.idProject" +
                $" join truth_time_ct.users u on p.idTeamLeader = u.idUser" +
                $" WHERE up.idUser = {idUser}";

            Func<MySqlDataReader, List<UserProjectHelp>> func = (reader) =>
            {
                List<UserProjectHelp> userProjectHelp = new List<UserProjectHelp>();
                while (reader.Read())
                {
                    userProjectHelp.Add(new UserProjectHelp
                    {
                        NameProject = (string)reader[0],
                        StartDate = (DateTime)reader[1],
                        EndDate = (DateTime)reader[2],
                        HoursProjectUser = (int)reader[3],
                        NameTeamLeader = (string)reader[4]

                    });
                }
                return userProjectHelp;
            };

            return DBUse.RunReader(query, func);
        }
        //add usersProjects To Specipic Project
        public static void CreateUsersProjectList(int idProject, int idTeamLeader)
        {
            List<User> allUsersUnderTeamLeader = LogicUsers.GetAllUsersUnderTeamLeader(idTeamLeader);
            UserProject userProject;
            foreach (User user in allUsersUnderTeamLeader)
            {
                userProject = new UserProject()
                {
                    IdProject = idProject,
                    IdUser = user.IdUser,
                    HoursProjectUser = 0
                };
                AddUserProject(userProject);
            }

        }
        //return all names of users and UsersProjects under teamleader
        public static List<UserProjectHelp> GetAllUserProjectUnderTeamLeaderWithNames(int idTeamLeader)
        {
            int idStatus = 0;
            double allocate = 0;
            string query = $"SELECT up.*,sub.catch,sub.idStatus FROM truth_time_ct.users_projects up join truth_time_ct.projects p on up.idProject = p.idProject join users u on up.idUser = u.idUser " +
                $"join(select idStatus, idProject, sum(hoursProjectUser) as catch from(SELECT up.*, idStatus FROM truth_time_ct.users_projects up join truth_time_ct.projects p" +
                $" on up.idProject = p.idProject join users u on up.idUser = u.idUser where p.active = true and p.idTeamLeader = {idTeamLeader}) as t1" +
                $" group by idProject,idStatus) sub on p.idProject = sub.idProject and u.idStatus = sub.idStatus where p.active = true and p.idTeamLeader = {idTeamLeader}";
            List<User> allUsers = LogicUsers.GetAllUsers();
            List<Project> allProjectUnderTeamLeader = LogicProjects.GetProjectsUnderTeamLeader(idTeamLeader);
            Func<MySqlDataReader, List<UserProjectHelp>> func = (reader) =>
            {
                List<UserProjectHelp> users_projects_help = new List<UserProjectHelp>();

                while (reader.Read())
                {
                    idStatus = allUsers.FirstOrDefault(p => p.IdUser == (int)reader[3]).IdStatus;
                    switch (idStatus)
                    {
                        case 3:
                            allocate = allProjectUnderTeamLeader.FirstOrDefault(p => p.IdProject == (int)reader[2]).HoursForDevelopers;
                            break;
                        case 4:
                            allocate = allProjectUnderTeamLeader.FirstOrDefault(p => p.IdProject == (int)reader[2]).HoursForQA;
                            break;
                        case 5:
                            allocate = allProjectUnderTeamLeader.FirstOrDefault(p => p.IdProject == (int)reader[2]).HoursForUI_UX;
                            break;
                    }
                    users_projects_help.Add(new UserProjectHelp
                    {
                        IdUserProject = (int)reader[0],
                        HoursProjectUser = (int)reader[1],
                        IdProject = (int)reader[2],
                        IdUser = (int)reader[3],
                        NameProject = allProjectUnderTeamLeader.FirstOrDefault(p => p.IdProject == (int)reader[2]).ProjectName,
                        NameUser = allUsers.FirstOrDefault(p => p.IdUser == (int)reader[3]).UserName,
                        TimeLeft = allocate - double.Parse(reader[4].ToString()),
                        IdStatusUser = (int)reader[5]
                    });
                }

                return users_projects_help;
            };

            return DBUse.RunReader(query, func);
        }
        //set all hours of usersProjects
        public static bool SetAllUsersProjects(List<UserProject> userProjectForEdit)
        {
            foreach (UserProject userProject in userProjectForEdit)
            {
                if (!UpdateUserProject(userProject))
                    return false;
            }
            return true;
        }
        //add UsersProjectsToAllNewUser
        public static bool CreateUsersProjectListToNewUser(User oldUser, Boolean isNew)
        {
            string idUser = LogicUsers.GetUserIdByPassword(oldUser.Password);
            User newUser = LogicUsers.GetUserByIdUser(int.Parse(idUser));
            if (oldUser.IdTeamLeader == newUser.IdTeamLeader && isNew != false)
                return true;
            //if (oldUser.IdTeamLeader == user.IdTeamLeader)
            //    return true;//because the teamLeader didnt changed so no need add userProjects
            List<Project> allProjectUnderTeamLeader = LogicProjects.GetProjectsUnderTheDirectionOfTheTeamLeader(newUser.IdTeamLeader);
            UserProject userProject;
            bool b = true;
            foreach (Project project in allProjectUnderTeamLeader)
            {
                userProject = new UserProject()
                {
                    IdProject = project.IdProject,
                    IdUser = newUser.IdUser,
                    HoursProjectUser = 0
                };
                if (!AddUserProject(userProject))
                    b = false;
            }
            return b;
        }
        //remove UsersProjectsTothe user that passed to another teamLeader
        public static bool RemoveUsersProjectListToUserThatPassedToAnotherTeamLeader(User user)
        {
            User userWithThePreviousTeamLeader = LogicUsers.GetUserByIdUser(user.IdUser);
            if (userWithThePreviousTeamLeader.IdTeamLeader == user.IdTeamLeader)
                return true;//because the teamLeader didnt changed so no need remove userProjects
            List<Project> allProjectUnderTeamLeader = LogicProjects.GetProjectsUnderTheDirectionOfTheTeamLeader(userWithThePreviousTeamLeader.IdTeamLeader);
            bool b = true;
            foreach (Project project in allProjectUnderTeamLeader)
            {
                RemoveUserProjectByUserAndProject(userWithThePreviousTeamLeader.IdUser, project.IdProject);
            }
            return b;
        }
        //add userProject for permmissinUsers
        public static bool AddUsersPermission(List<User> users, int idProject)
        {
            UserProject userProject;
            bool b = true;
            foreach (User user in users)
            {
                userProject = new UserProject()
                {
                    IdProject = idProject,
                    IdUser = user.IdUser,
                    HoursProjectUser = 0
                };
                if (!AddUserProject(userProject))
                    b = false;
            }
            return b;
        }
        //remove UsersProjectsTothe user that deleted
        public static bool RemoveUsersProjectListToUserThatDeleted(int idUser)
        {
            string query = $"DELETE FROM truth_time_ct.users_projects WHERE idUser={idUser}";
            DBUse.RunScalar(query);
            return true;
        }
    }
}
```

### BLL
```csharp
using _00_DAL;
using _01_BOL;
using System;
using System.Collections.Generic;
using MySql.Data.MySqlClient;

namespace _02_BLL
{
    public class LogicUsers
    {
        public static List<User> GetAllUsers()
        {
            string query = $"SELECT * FROM truth_time_ct.users u where isActive=1";
            Func<MySqlDataReader, List<User>> func = (reader) =>
            {
                List<User> users = new List<User>();
                while (reader.Read())
                {
                    int p = (int)reader[0];
                    string p1 = (string)reader[1];
                    string p2 = (string)reader[2];
                    int p3 = (int)reader[3];
                    string p4 = (string)reader[4];
                    double p5 = (double)reader[5];
                    int p6 = 0;
                    int.TryParse(reader[6].ToString(), out p6);
                    bool p7 = reader.GetBoolean(7);


                    //StatusUser s = new StatusUser() { IdStatus = p3, StatusName = LogicStatusUsers.GetStatusName(p3) };
                    users.Add(new User
                    {
                        IdUser = p,
                        UserName = p1,
                        Password = p2,
                        IdStatus = p3,
                        EmailUser = p4,
                        SumHours = p5,
                        IdTeamLeader = p6,
                        IsActive = p7
                    });
                }
                return users;
            };

            return DBUse.RunReader(query, func);
        }
        public static bool RemoveUser(User user)
        {
            string query = $"UPDATE truth_time_ct.users SET isActive ={ user.IsActive} WHERE idUser = { user.IdUser }";
            return DBUse.RunNonQuery(query) == 1;
        }
        public static bool UpdateUser(User user)
        {
            User oldUser = LogicUsers.GetUserByIdUser(user.IdUser);
            string query = $"UPDATE truth_time_ct.users SET userName='{user.UserName}',password='{user.Password}',idStatus={user.IdStatus},emailUser='{user.EmailUser}',sumHours={user.SumHours},idTeamLeader={user.IdTeamLeader},isActive={user.IsActive},ipComputer='{user.ComputerIp}' WHERE idUser={user.IdUser}";
            DBUse.RunNonQuery(query);
            return LogicUserProject.CreateUsersProjectListToNewUser(oldUser, true);
        }
        public static string GetUserIdByPassword(string pasword)
        {
            string query = $"SELECT idUser FROM truth_time_ct.users WHERE password='{pasword}'";
            return DBUse.RunScalar(query).ToString();
        }
        public static bool AddUser(User user)
        {
            string query = $"INSERT INTO truth_time_ct.users VALUES (0,'{user.UserName}','{user.Password}'" +
     $",{user.IdStatus},'{user.EmailUser}',{user.SumHours},{user.IdTeamLeader},{user.IsActive},'0')";
            return DBUse.RunNonQuery(query) == 1 && LogicUserProject.CreateUsersProjectListToNewUser(user, false);
        }
        //return all users under TeamLeader
        public static List<User> GetAllUsersUnderTeamLeader(int idTeamLeader)
        {
            string query = $"SELECT * FROM truth_time_ct.users WHERE idTeamLeader={idTeamLeader}";

            Func<MySqlDataReader, List<User>> func = (reader) =>
            {
                List<User> users = new List<User>();
                while (reader.Read())
                {
                    int p = (int)reader[0];
                    string p1 = (string)reader[1];
                    string p2 = (string)reader[2];
                    int p3 = (int)reader[3];
                    string p4 = (string)reader[4];
                    double p5 = (double)reader[5];
                    int p6 = 0;
                    int.TryParse(reader[6].ToString(), out p6);
                    bool p7 = reader.GetBoolean(7);
                    users.Add(new User
                    {
                        IdUser = p,
                        UserName = p1,
                        Password = p2,
                        IdStatus = p3,
                        //StatusUserFK =s,
                        EmailUser = p4,
                        SumHours = p5,
                        IdTeamLeader = p6,
                        IsActive = p7

                    });


                }
                return users;
            };

            return DBUse.RunReader(query, func);
        }
        //get user by idUser
        public static User GetUserByIdUser(int idUser)
        {
            string query = $"SELECT * FROM truth_time_ct.users WHERE idUser={idUser}";
            Func<MySqlDataReader, List<User>> func = (reader) =>
            {
                List<User> users = new List<User>();
                while (reader.Read())
                {
                    int p6 = 0;
                    int.TryParse(reader[6].ToString(), out p6);
                    bool p7 = reader.GetBoolean(7);
                    users.Add(new User
                    {
                        IdUser = (int)reader[0],
                        UserName = (string)reader[1],
                        Password = (string)reader[2],
                        IdStatus = (int)reader[3],
                        EmailUser = (string)reader[4],
                        SumHours = (double)reader[5],
                        IdTeamLeader = p6,
                        IsActive = p7
                    });
                }
                return users;
            };
            User myUser = DBUse.RunReader(query, func)[0];
            return myUser;
        }
        //all users that worked on specipic project
        public static List<User> GetUsersOfProject(int idProject)
        {
            string query = $"SELECT * FROM truth_time_ct.users u join truth_time_ct.users_projects up on u.idUser=up.idUser where up.idProject={idProject}";
            Func<MySqlDataReader, List<User>> func = (reader) =>
            {
                List<User> Users = new List<User>();
                while (reader.Read())
                {
                    int p6 = 0;
                    int.TryParse(reader[6].ToString(), out p6);
                    bool p7 = reader.GetBoolean(7);
                    Users.Add(new User
                    {
                        IdUser = (int)reader[0],
                        UserName = (string)reader[1],
                        Password = (string)reader[2],
                        IdStatus = (int)reader[3],
                        EmailUser = (string)reader[4],
                        SumHours = (double)reader[5],
                        IdTeamLeader = p6,
                        IsActive = p7
                    });
                }
                return Users;
            };
            return DBUse.RunReader(query, func);
        }
        //return all team leader
        public static List<User> GetAllTeamLeader()
        {
            string query = $"SELECT  * FROM truth_time_ct.users u join truth_time_ct.status_users s on u.idStatus = s.idStatus where s.statusName LIKE '%leader%'";
            Func<MySqlDataReader, List<User>> func = (reader) =>
            {
                List<User> Users = new List<User>();
                while (reader.Read())
                {
                    int p6 = 0;
                    int.TryParse(reader[6].ToString(), out p6);
                    bool p7 = reader.GetBoolean(7);
                    Users.Add(new User
                    {
                        IdUser = (int)reader[0],
                        UserName = (string)reader[1],
                        Password = (string)reader[2],
                        IdStatus = (int)reader[3],
                        EmailUser = (string)reader[4],
                        SumHours = (double)reader[5],
                        IdTeamLeader = p6,
                        IsActive = p7
                    });
                }
                return Users;
            };

            return DBUse.RunReader(query, func);
        }
        //return all user that their status contains the variable got
        public static List<User> GetAllSpesificStatus(string statusName)
        {
            string query = $"SELECT  * FROM truth_time_ct.users u join truth_time_ct.status_users s on u.idStatus = s.idStatus where s.statusName LIKE '%{statusName}%'";
            Func<MySqlDataReader, List<User>> func = (reader) =>
            {
                List<User> Users = new List<User>();
                while (reader.Read())
                {
                    int p6 = 0;
                    int.TryParse(reader[6].ToString(), out p6);
                    bool p7 = reader.GetBoolean(7);
                    Users.Add(new User
                    {
                        IdUser = (int)reader[0],
                        UserName = (string)reader[1],
                        Password = (string)reader[2],
                        IdStatus = (int)reader[3],
                        EmailUser = (string)reader[4],
                        SumHours = (double)reader[5],
                        IdTeamLeader = p6,
                        IsActive = p7
                    });
                }
                return Users;
            };

            return DBUse.RunReader(query, func);
        }
        //return user by ip computer
        public static User GetUserByIPComputer(string computerIP)
        {

            string query = $"SELECT * FROM truth_time_ct.users u where ipComputer='{computerIP}'";
            Func<MySqlDataReader, List<User>> func = (reader) =>
            {
                List<User> Users = new List<User>();
                while (reader.Read())
                {
                    int p6 = 0;
                    int.TryParse(reader[6].ToString(), out p6);
                    bool p7 = reader.GetBoolean(7);
                    Users.Add(new User
                    {
                        IdUser = (int)reader[0],
                        UserName = (string)reader[1],
                        Password = (string)reader[2],
                        IdStatus = (int)reader[3],
                        EmailUser = (string)reader[4],
                        SumHours = (double)reader[5],
                        IdTeamLeader = p6,
                        IsActive = p7,
                        ComputerIp = (string)reader[8]
                    });
                }
                return Users;
            };
            User user = new User();
            var listUsers = new List<User>();
            listUsers = DBUse.RunReader(query, func);
            if (listUsers.Count > 0)
                return listUsers[0];
            return user;

        }
    }
}





