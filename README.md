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

### BLL
```csharp
using BOL;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Mail;
using System.Timers;

namespace BLL
{
    public static class HomeLogic
    {
        public static Worker Login(string userName, string password)
        {
            string query = $"SELECT * FROM task_managment.workers WHERE user_name='{userName}' and password='{password}' ";
            Func<MySqlDataReader, List<Worker>> func = (reader) => {
                List<Worker> worker = new List<Worker>();
                while (reader.Read())
                {
                    worker.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        UserName = reader.GetString(2),
                        Password = "",
                        EMail = reader.GetString(4),
                        JobId = reader.GetInt32(5),
                        ManagerId = reader[6] as int?
                    });
                }
                return worker;
            };
            List<Worker> workers= DBAccess.RunReader(query, func);
            if (workers != null && workers.Count > 0)
                return workers[0];
            return null;
        }

        public static bool UpdatePassword(string userName, string oldpassword, string newPassord)
        {
            string query =$" UPDATE workers SET password = '{newPassord}' WHERE user_name = '{userName}' AND password = '{oldpassword}'";
            return DBAccess.RunNonQuery(query) == 1;
        }

        public static bool sendEmail(string sub, string body, string email)
        {
            MailMessage msg = new MailMessage();
            msg.From = new MailAddress("shtilimrishum2018@gmail.com");
            msg.To.Add(new MailAddress(email));
            msg.Subject = sub;
            msg.Body = body;
            SmtpClient client = new SmtpClient();
            client.UseDefaultCredentials = true;
            client.Host = "smtp.gmail.com";
            client.Port = 587;
            client.EnableSsl = true;
            client.DeliveryMethod = SmtpDeliveryMethod.Network;
            client.Credentials = new NetworkCredential("shtilimrishum2018@gmail.com", "0504190762");
            client.Timeout = 20000;
            try
            {
                client.Send(msg);
                return true;
            }
            catch (Exception ex)
            {
                return false;
            }
            finally
            {
                msg.Dispose();
            }
        }
   
        public static void OnStart(object sender, ElapsedEventArgs args)
        {
            string query = $"SELECT w.email, p.name ,p.end_date" +
            $" FROM workers w JOIN project_workers pw ON W.worker_id = pw.worker_id JOIN projects p" +
            $" ON p.project_id = pw.project_id" +
            $" WHERE p.end_date <= NOW() + INTERVAL 1 DAY" +
             $" UNION SELECT w.email, p.name ,p.end_date" +
            $" FROM workers w JOIN projects p ON p.team_leader = W.worker_id" +
             $" WHERE p.end_date <= NOW() + INTERVAL 1 DAY";

            Func<MySqlDataReader, List<object>> func = (reader) => {
                List<object> objects = new List<object>();
                while (reader.Read())
                {
                    var o = new
                    {
                        email = reader.GetString(0),
                        name = reader.GetString(1),
                        end_date = Convert.ToDateTime(reader.GetString(2)),
                    };
                    sendEmail("Notification do not end task", $"the project {o.name} end in {o.end_date}!!", o.email);
                }
                return objects;
            };
            List<object> objects2 = DBAccess.RunReader(query, func);
        }
        public static  void Notifications()
        {
            OnStart(null, null);
            Timer timer = new Timer();
            timer.Interval = 60000*60*24; // once a day
            timer.Elapsed += new ElapsedEventHandler(OnStart);
            timer.Start();
        }
    }
}
```
```csharp
using BOL;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace BLL
{
    public class ManagerLogic
    {
        public static bool AddProject(Project project)
        {
            string query = $"INSERT INTO task_managment.projects  " +
                $"(name,customer,team_leader,develop_houres,qa_houres,ui_ux_houres,start_date,end_date)" +
                $" VALUES ('{project.Name}','{project.Customer}'," +
                $"'{project.TeamLeaderId}',{project.DevelopHours},{project.QAHours},{project.UiUxHours}," +
                $"'{project.StartDate.Year}-{project.StartDate.Month}-{project.StartDate.Day}','{project.EndDate.Year}-{project.EndDate.Month}-{project.EndDate.Day}')";
            if (DBAccess.RunNonQuery(query) == 1)
            {
                return AddWorkersToProject(project);
            }
            return false;
        }

        private static bool AddWorkersToProject(Project project)
        {
            var query = $"SELECT project_id FROM projects WHERE name = '{project.Name}'";
            int idProject = (int)DBAccess.RunScalar(query);
            query = $"SELECT worker_id FROM workers" +
                    $" WHERE manager ={project.TeamLeaderId}";
            List<int> workersId = new List<int>();
            Func<MySqlDataReader, List<int>> func = (reader) =>
            {
                List<int> workers = new List<int>();
                while (reader.Read())
                {
                    //Add all the teamLeadr's worker to the project
                    workers.Add(reader.GetInt32(0));
                }
                return workers;
            };

            workersId = DBAccess.RunReader(query, func);
            workersId.ForEach(idWorker =>
            {
                var q = $"INSERT INTO project_workers (worker_id,project_id) VALUES({idWorker},{idProject})";
                DBAccess.RunNonQuery(q);
            });
            return true;
        }

        public static bool addWorkersToProject(int[] ids, string name)
        {
            string q = $"SELECT project_id FROM projects WHERE name = '{name}'";
            int idProject = (int)DBAccess.RunScalar(q);
            foreach (var item in ids)
            {
                string query = $"INSERT INTO task_managment.project_workers (worker_id,project_id) VALUES({item},{idProject})";
                if (DBAccess.RunNonQuery(query) == 0)
                    return false;
            }
            return true;
        }

        public static bool AddWorker(Worker worker)
        {
            string query = $"INSERT INTO task_managment.workers  " +
                $"(name,user_name,password,email,job,manager)" +
                $" VALUES ('{worker.Name}','{worker.UserName}'," +
                $"'{worker.Password}','{worker.EMail}',{worker.JobId},{worker.ManagerId})";
          if (DBAccess.RunNonQuery(query) == 1)
            {
                int id =(int)DBAccess.RunScalar( $" SELECT worker_id FROM workers WHERE user_name = '{worker.UserName}'");
                string query2 = $" SELECT project_id FROM task_managment.projects WHERE team_leader={worker.ManagerId}";
                Func<MySqlDataReader, List<int>> func = (reader) =>
                {
                    List<int> projectsId = new List<int>();
                    while (reader.Read())
                    {
                        projectsId.Add(reader.GetInt32(0));
                    }
                    return projectsId;
                };

                List<int> projectsIds= DBAccess.RunReader(query2, func);
                projectsIds.ForEach(p =>
                {
                    string query3 = $"INSERT INTO task_managment.project_workers (worker_id,project_id) VALUES ({id},{p})";
                    DBAccess.RunNonQuery(query3);
                });
                return true;
            }
            return false;
        }

        public static bool UpdateWorker(Worker worker)
        {
               string  query = $"UPDATE task_managment.workers SET name='{worker.Name}', user_name='{worker.UserName}'" +
                $", email='{worker.EMail}', job={worker.JobId}, manager={worker.ManagerId} WHERE worker_id={worker.Id}";
            return DBAccess.RunNonQuery(query) == 1;
        }

        public static List<Worker> GetAllWorkers()
        {
            string query = $"SELECT * FROM task_managment.workers";
            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        UserName = reader.GetString(2),
                        Password = "",
                        EMail = reader.GetString(4),
                        JobId = reader.GetInt32(5),
                        ManagerId = reader[6] as int?

                    });
                }
                return workers;
            };
            return DBAccess.RunReader(query, func);
        }
        public static List<Worker> GetAllManagers()
        {
            string query = $"SELECT * FROM task_managment.workers WHERE job<=2";
            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> managers = new List<Worker>();
                while (reader.Read())
                {
                    managers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        UserName = reader.GetString(2),
                        Password = "",
                        EMail = reader.GetString(4),
                        JobId = reader.GetInt32(5),
                        ManagerId = reader[6] as int?
                    });
                }
                return managers;
            };
            return DBAccess.RunReader(query, func);
        }

        public static List<Job> GetAllJobs()
        {
            string query = $"SELECT * FROM task_managment.jobs";
            Func<MySqlDataReader, List<Job>> func = (reader) =>
            {
                List<Job> jobs = new List<Job>();
                while (reader.Read())
                {
                    jobs.Add(new Job
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                    });
                }
                return jobs;
            };
            return DBAccess.RunReader(query, func);

        }
        public static bool RemoveWorker(int id)
        {
            //To delete an employee, it is necessary to delete it from all the tables in which it is located.
            var q = $"SELECT project_worker_id  FROM task_managment.project_workers  WHERE   worker_id={id}";
            List<int> projectsworkerId = new List<int>();
            Func<MySqlDataReader, List<int>> func = (reader) =>
            {
                List<int> projects = new List<int>();
                while (reader.Read())
                {
                    //Add all the teamLeadr's worker to the project
                    projects.Add(reader.GetInt32(0));
                }
                return projects;
            };

            projectsworkerId = DBAccess.RunReader(q, func);
            projectsworkerId.ForEach(idProject =>
            {
                var q2 = $"DELETE FROM task_managment.work_hours WHERE project_work_id={idProject}";
                DBAccess.RunNonQuery(q2);
            });
            q = $"DELETE FROM task_managment.project_workers WHERE worker_id={id}";
            DBAccess.RunNonQuery(q);
            q = $"DELETE FROM task_managment.workers WHERE worker_id={id}";
            return DBAccess.RunNonQuery(q) == 1;

        }

        public static List<Object> GetPresence()
        {
            string query = $"SELECT w.name, p.name, wh.date , wh.start , wh.end" +
$" FROM workers W JOIN project_workers pw ON w.worker_id = pw.worker_id" +
$" JOIN projects P ON pw.project_id = p.project_id JOIN work_hours wh" +
$" ON wh.project_work_id = pw.project_worker_id" +
$" ORDER BY w.name, p.name, wh.date , wh.start";

            Func<MySqlDataReader, List<Object>> func = (reader) =>
            {
                List<Object> Presence = new List<Object>();
                while (reader.Read())
                {
                    Presence.Add(new 
                    {
                        WorkerName = reader.GetString(0),
                        ProjectName=reader.GetString(1),
                        Date = reader.GetDateTime(2),
                       Start=reader.GetString(3),
                       End = reader.GetString(4),
                    });
                }
                return Presence;
            };

            return DBAccess.RunReader(query, func);
        }
       
        public static string getPassword(int workerId)
        {
            string query = $"select password from workers where worker_id={workerId}";
            return DBAccess.RunScalar(query).ToString();
        }

    }
}
```
```csharp
using BOL;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;
using System.Linq;

namespace BLL
{
    public static class TeamLeaderLogic
    {

        public static List<Project> GetProjectDeatails(int teamLeaderId)
        {
            string query = $"SELECT * FROM task_managment.projects WHERE team_leader={teamLeaderId}";

            Func<MySqlDataReader, List<Project>> func = (reader) =>
            {
                List<Project> projects = new List<Project>();
                while (reader.Read())
                {
                    projects.Add(new Project
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        Customer = reader.GetString(2),
                        TeamLeaderId = reader.GetInt32(3),
                        DevelopHours = reader.GetInt32(4),
                        QAHours = reader.GetInt32(5),
                        UiUxHours = reader.GetInt32(6),
                        StartDate = Convert.ToDateTime(reader.GetString(7)),
                        EndDate = Convert.ToDateTime(reader.GetString(8))
                    });
                }
                return projects;
            };
            return DBAccess.RunReader(query, func);
        }

        public static List<Worker> GetWorkersDeatails(int teamLeaderId)
        {
            string query = $"SELECT * FROM task_managment.workers WHERE manager={teamLeaderId}";

            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        UserName = reader.GetString(2),
                        Password = "",
                        EMail = reader.GetString(4),
                        JobId = reader.GetInt32(5),
                        ManagerId = reader[6] as int?
                    });
                }
                return workers;
            };
            return DBAccess.RunReader(query, func);
        }
      
        public static List<Object> getWorkersHours(int projectId)
        {
            string query = $"SELECT  name,SEC_TO_TIME(SUM(TIME_TO_SEC(end) - TIME_TO_SEC(start))) AS Time, allocated_hours" +
        $" FROM workers W JOIN project_workers PW ON W.worker_id=PW.worker_id LEFT JOIN work_hours WH ON PW.project_worker_id= WH.project_work_id" +
       $" WHERE PW.project_id= {projectId}" +
        $" GROUP BY name, allocated_hours ORDER BY name";
            Func<MySqlDataReader, List<Object>> func = (reader) =>
            {
                List<Object> unknowns = new List<Object>();
                while (reader.Read())
                {
                    string s = reader[2].ToString();
                    float.TryParse(s, out float x);
                       string s2;
                    try
                    {
                        TimeSpan t = reader.GetTimeSpan(1);
                        s2 = (t.Hours + t.Days * 24) + ":" + t.Minutes;
                    }
                    catch { s2 = 0 + ":" + 0; };
                    unknowns.Add(new 
                    {
                        Name = reader.GetString(0),
                       Hours = s2,
                       AllocatedHours = x
                    });
                }
                return unknowns;
            };
            return DBAccess.RunReader(query, func);
        }
        public static List<Object> getWorkerHours(int teamLeaderId, int workerId)
        {
            string query = $"SELECT pw.project_worker_id, p.name , allocated_hours , SEC_TO_TIME(SUM(TIME_TO_SEC(end) - TIME_TO_SEC(start)))" +
            $" FROM project_workers PW join projects p on p.project_id=pw.project_id"+
            $" LEFT join work_hours wh on wh.project_work_id= pw.project_worker_id" +
            $" where p.team_leader={teamLeaderId} and pw.worker_id= {workerId}"+
            $" group by pw.project_worker_id, p.name  ,  allocated_hours";

            Func<MySqlDataReader, List<Object>> func = (reader) =>
            {
                List<Object> unknowns = new List<Object>();
                while (reader.Read())
                {
                    string s = reader[2].ToString();
                    float.TryParse(s, out float x);
                    string s2 = reader[3].ToString();
                    unknowns.Add(new 
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        AllocatedHours = x,
                        Hours = s2
                    });
                }
                return unknowns;
            };
            return DBAccess.RunReader(query, func);

        }

        public static string GetRemainingHours(int projectId, int jobId)
        {
          string jobName= ManagerLogic.GetAllJobs().FirstOrDefault(j=>j.Id==jobId).Name;
            switch (jobName)
            {
                case "developer":
                    {
                        jobName = "develop_houres";
                        break;
                    }
                case "QA":
                    {
                        jobName = "qa_houres";
                        break;
                    }
                case "UxUi":
                    {
                        jobName = "ui_ux_houres";
                        break;
                    }
            }
 
            string query = $" SELECT {jobName} - SUM(allocated_hours)" +
$" FROM projects P JOIN  project_workers PW ON P.project_id = PW.project_id JOIN workers W ON W.worker_id = PW.worker_id" +
$" WHERE PW.project_worker_id = {projectId} AND W.job = {jobId}";
            return DBAccess.RunScalar(query).ToString();
        }

        public static bool UpdateWorkerHours(int projectWorkerId, int numHours)
        {
            string query = $"UPDATE task_managment.project_workers SET allocated_hours={numHours} WHERE project_worker_id={projectWorkerId}";
            return DBAccess.RunNonQuery(query) == 1;
        }

        public static string GetHours(int projectId)
        {
            string query = $"SELECT SEC_TO_TIME(SUM(TIME_TO_SEC(end) - TIME_TO_SEC(start))) FROM task_managment.work_hours WHERE project_work_id IN" +
                $"(SELECT project_worker_id FROM task_managment.project_workers" +
                $" WHERE project_id={projectId}) ";
            return DBAccess.RunScalar(query).ToString();
        }
    }
}

```
```csharp
using BOL;
using DAL;
using MySql.Data.MySqlClient;
using System;
using System.Collections.Generic;

namespace BLL
{
   public class WorkerLogic
    {
        public static bool UpdateStartHour(int idProjectWorker,DateTime hour )
        {
            string query = $"INSERT INTO task_managment.work_hours (project_work_id, date, start)"+
                $" VALUES({idProjectWorker}, '{DateTime.Now.Year}-{DateTime.Now.Month}-{DateTime.Now.Day}', '{hour.TimeOfDay}')";
            return DBAccess.RunNonQuery(query) == 1;
        }
        public static bool UpdateEndHour(int idProjectWorker, DateTime hour)
        {
            string query = $"UPDATE task_managment.work_hours  SET end='{hour.TimeOfDay}'  WHERE project_work_id={idProjectWorker}" +
                $" AND END IS NULL";
            return DBAccess.RunNonQuery(query) == 1;
        }
       
        public static bool SendMsg(string sub, string body,int id)
        {
            string query = $"SELECT email FROM task_managment.workers where worker_id = {id}" ;
           string email=(string) DBAccess.RunScalar(query);
            return HomeLogic.sendEmail(sub, body, email);
        }

        public static Worker GetWorkerDetails(int id)
        {
            string query = $"SELECT * FROM task_managment.workers WHERE worker_id={id}";
            Func<MySqlDataReader, List<Worker>> func = (reader) =>
            {
                List<Worker> workers = new List<Worker>();
                while (reader.Read())
                {
                    workers.Add(new Worker
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        UserName = reader.GetString(2),
                        Password = "",
                        EMail = reader.GetString(4),
                        JobId = reader.GetInt32(5),
                        ManagerId = reader[6] as int?
                    });
                }
                return workers;
            };
            List<Worker> workers2 = DBAccess.RunReader(query, func);
            if (workers2 != null && workers2.Count > 0)
                return workers2[0];
            return null;
        }
        public static List<object> GetProject(int id)
        {
            string query = $"SELECT PW.project_worker_id, p.name,  allocated_hours, SEC_TO_TIME(SUM(TIME_TO_SEC(end) - TIME_TO_SEC(start))) AS Time" +
        $" FROM project_workers PW JOIN projects P ON P.project_id = PW.project_id LEFT JOIN" +
        $" work_hours WH ON PW.project_worker_id = WH.project_work_id" +
        $" WHERE PW.worker_id = {id}" +
        $"    GROUP BY PW.project_worker_id,p.name,allocated_hours   ORDER BY project_worker_id";
      
            Func<MySqlDataReader, List<Object>> func = (reader) =>
            {
                List<Object> unknowns = new List<Object>();
                while (reader.Read())
                {
                     string s=reader[2].ToString();
                    float.TryParse(s,out float  x);
                    string s2;
                    try {
                        TimeSpan t = reader.GetTimeSpan(3);
                        s2 = (t.Hours + t.Days * 24) + ":" + t.Minutes;
                    }
                    catch { s2 = 0+ ":" +0; };
                        unknowns.Add(new 
                        {
                            Id = reader.GetInt32(0),
                            Name = reader.GetString(1),
                            AllocatedHours = x,
                            Hours = s2 
                    });
                }
                return unknowns;
            };
            return DBAccess.RunReader(query, func);
        }

        public static string GetAllHours(int id)
        {
            string query = $"SELECT SEC_TO_TIME(SUM(TIME_TO_SEC(end) - TIME_TO_SEC(start))) FROM task_managment.work_hours" +
                $" WHERE work_hours_id IN(SELECT project_worker_id" +
 $" FROM task_managment.project_workers WHERE worker_id = {id})";
            return DBAccess.RunScalar(query).ToString();
        }
    }
}





