# Реалізація інформаційного та програмного забезпечення

## SQL-скрипт для створення початкового наповнення бази даних

```sql
CREATE SCHEMA IF NOT EXISTS ProjectManagement;
USE ProjectManagement;

CREATE TABLE IF NOT EXISTS Project (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    description TEXT
);

CREATE TABLE IF NOT EXISTS Board (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    project_id BIGINT,
    FOREIGN KEY (project_id) REFERENCES Project(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS Task (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    deadline DATETIME,
    status VARCHAR(50),
    board_id BIGINT,
    FOREIGN KEY (board_id) REFERENCES Board(id) ON DELETE SET NULL
);

CREATE TABLE IF NOT EXISTS Attachment (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    format VARCHAR(50),
    content BLOB,
    task_id BIGINT,
    FOREIGN KEY (task_id) REFERENCES Task(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS Label (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    color VARCHAR(20)
);

CREATE TABLE IF NOT EXISTS Tag (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    task_id BIGINT,
    label_id BIGINT,
    FOREIGN KEY (task_id) REFERENCES Task(id) ON DELETE CASCADE,
    FOREIGN KEY (label_id) REFERENCES Label(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS User (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    role VARCHAR(50),
    isBanned TINYINT DEFAULT 0
);

CREATE TABLE IF NOT EXISTS Member (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT,
    project_id BIGINT,
    FOREIGN KEY (user_id) REFERENCES User(id) ON DELETE CASCADE,
    FOREIGN KEY (project_id) REFERENCES Project(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS Assignee (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    task_id BIGINT,
    member_id BIGINT,
    FOREIGN KEY (task_id) REFERENCES Task(id) ON DELETE CASCADE,
    FOREIGN KEY (member_id) REFERENCES Member(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS Permission (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    action VARCHAR(255) NOT NULL
);

CREATE TABLE IF NOT EXISTS AccessGrant (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    member_id BIGINT,
    permission_id BIGINT,
    FOREIGN KEY (member_id) REFERENCES Member(id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES Permission(id) ON DELETE CASCADE
);


INSERT INTO Project (title, description) VALUES
('Project Alpha', 'Опис проекту Alpha'),
('Project Beta', 'Опис проекту Beta');

INSERT INTO Board (title, project_id) VALUES
('Board 1', 1),
('Board 2', 1),
('Board 3', 2);

INSERT INTO Task (title, description, deadline, status, board_id) VALUES
('Task 1', 'Опис Task 1', '2024-12-31 23:59:59', 'open', 1),
('Task 2', 'Опис Task 2', '2024-11-30 23:59:59', 'in progress', 1),
('Task 3', 'Опис Task 3', '2024-10-15 23:59:59', 'completed', 2);

INSERT INTO Attachment (format, content, task_id) VALUES
('pdf', 0x1234567890ABCDEF, 1),
('jpg', 0xABCDEF1234567890, 2),
('docx', 0x7890ABCDEF123456, 3);

INSERT INTO Label (name, color) VALUES
('Urgent', 'red'),
('Review', 'blue'),
('Low Priority', 'green');

INSERT INTO Tag (task_id, label_id) VALUES
(1, 1),
(2, 2),
(3, 3);

INSERT INTO User (username, password, email, role, isBanned) VALUES
('john_doe', 'password123', 'john@example.com', 'admin', 0),
('jane_smith', 'password456', 'jane@example.com', 'member', 0),
('alice_jones', 'password789', 'alice@example.com', 'member', 0);

INSERT INTO Member (user_id, project_id) VALUES
(1, 1),
(2, 1),
(3, 2);

INSERT INTO Assignee (task_id, member_id) VALUES
(1, 1),
(2, 2),
(3, 3);

INSERT INTO Permission (action) VALUES
('view'),
('edit'),
('delete');

INSERT INTO AccessGrant (member_id, permission_id) VALUES
(1, 1),
(1, 2),
(2, 1),
(2, 3),
(3, 1);
```


## RESTfull сервіс для управління даними

Цей RESTful сервіс розроблений з використанням .NET 7 та забезпечує функціонал для управління проектами. Основними компонентами цього сервісу є:

<ul>
  <li><strong>ASP.NET Core Web API</strong> — дозволяє створювати RESTful API з підтримкою контролерів, маршрутизації та моделі-вид-контролер (MVC).</li>
  <li><strong>Entity Framework Core</strong> — використовується для взаємодії з базою даних через ORM (Object-Relational Mapping). Забезпечує легке створення моделей та автоматизацію більшості CRUD операцій.</li>
  <li><strong>MySQL</strong> — реляційна база даних для зберігання даних проектів.</li>
  <li><strong>Swagger (Swashbuckle)</strong> — використовується для документування та тестування API за допомогою OpenAPI Specification.</li>
</ul>

## Діаграма класів
![image](https://github.com/user-attachments/assets/c1c8f0be-1b95-4460-bac7-a81023e93441)

## Project Management API Specification

## Entity

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace ProjectManagementAPI.Models
{
    [Table("project")]
    public class Project
    {
        [Key]
        public long Id { get; set; }

        [Required]
        [MaxLength(255)]
        public string Title { get; set; }

        public string? Description { get; set; }
    }
}
```

## Repository

```csharp
using Microsoft.EntityFrameworkCore;
using ProjectManagementAPI.Models;

namespace ProjectManagementAPI.Data
{
    public class ProjectManagementContext : DbContext
    {
        public ProjectManagementContext(DbContextOptions<ProjectManagementContext> options)
            : base(options)
        {
        }

        public DbSet<Project> Projects { get; set; }
    }
}
```

## Controller

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using ProjectManagementAPI.Data;
using ProjectManagementAPI.Models;

namespace ProjectManagementAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ProjectsController : ControllerBase
    {
        private readonly ProjectManagementContext _context;

        public ProjectsController(ProjectManagementContext context)
        {
            _context = context;
        }

        [HttpGet]
        public async Task<ActionResult<IEnumerable<Project>>> GetProjects()
        {
            return await _context.Projects.ToListAsync();
        }

        [HttpGet("{id}")]
        public async Task<ActionResult<Project>> GetProject(long id)
        {
            var project = await _context.Projects.FindAsync(id);
            if (project == null) return NotFound();

            return project;
        }

        [HttpPost]
        public async Task<ActionResult<Project>> PostProject(Project project)
        {
            _context.Projects.Add(project);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetProject), new { id = project.Id }, project);
        }

        [HttpPut("{id}")]
        public async Task<IActionResult> PutProject(long id, Project project)
        {
            if (id != project.Id) return BadRequest();

            _context.Entry(project).State = EntityState.Modified;

            try
            {
                await _context.SaveChangesAsync();
            }
            catch (DbUpdateConcurrencyException)
            {
                if (!_context.Projects.Any(e => e.Id == id)) return NotFound();
                throw;
            }

            return NoContent();
        }

        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteProject(long id)
        {
            var project = await _context.Projects.FindAsync(id);
            if (project == null) return NotFound();

            _context.Projects.Remove(project);
            await _context.SaveChangesAsync();

            return NoContent();
        }
    }
}
```

## Main Class for Application Launch

```csharp
using Microsoft.EntityFrameworkCore;
using ProjectManagementAPI.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
builder.Services.AddDbContext<ProjectManagementContext>(options =>
    options.UseMySQL(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapControllers();

app.Run();
```

