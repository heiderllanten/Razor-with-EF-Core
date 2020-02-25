# Razor Pages with Entity Framework Core in ASP.NET Core

## The data model

### Relationships

A relationship defines how two entities relate to each other. In a relational database, this is represented by a foreign key constraint.

**Navigation property:** Navigation properties hold other entities that are related to this entity. A property defined on the principal and/or dependent entity that references the related entity.

The Grade property is an 'enum'. The question mark after the Grade type declaration indicates that the Grade property is nullable. A grade that's null is different from a zero grade—null means a grade isn't known or hasn't been assigned yet.

The 'DatabaseGenerated' attribute allows the app to specify the primary key rather than having the database generate it.

### Scaffold

The scaffolding process:

- Creates Razor pages in the Pages/Students folder:
    - Create.cshtml and Create.cshtml.cs
    - Delete.cshtml and Delete.cshtml.cs
    - Details.cshtml and Details.cshtml.cs
    - Edit.cshtml and Edit.cshtml.cs
    - Index.cshtml and Index.cshtml.cs
- Creates Data/SchoolContext.cs.
- Adds the context to dependency injection in Startup.cs.
- Adds a database connection string to appsettings.json.

### Database Context Class

The highlighted code creates a DbSet<TEntity> property for each entity set. In EF Core terminology:

- An entity set typically corresponds to a database table.
- An entity corresponds to a row in the table.

Since an entity set contains multiple entities, the DBSet properties should be plural names. Since the scaffolding tool created aStudent DBSet, this step changes it to plural Students.

### Startup.cs

ASP.NET Core is built with dependency injection. Services (such as the EF Core database context) are registered with dependency injection during application startup. Components that require these services (such as Razor Pages) are provided these services via constructor parameters. The constructor code that gets a database context instance is shown later in the tutorial.

### Create the Database

The EnsureCreated method takes no action if a database for the context exists. If no database exists, it creates the database and schema. EnsureCreated enables the following workflow for handling data model changes:

Delete the database. Any existing data is lost.
Change the data model. For example, add an EmailAddress field.
Run the app.
EnsureCreated creates a database with the new schema.
This workflow works well early in development when the schema is rapidly evolving, as long as you don't need to preserve data. The situation is different when data that has been entered into the database needs to be preserved. When that is the case, use migrations.

Later in the tutorial series, you delete the database that was created by EnsureCreated and use migrations instead. A database that is created by EnsureCreated can't be updated by using migrations.

### Seed the database

'Drop-Database'

### Asynchronous code

Asynchronous programming is the default mode for ASP.NET Core and EF Core.

A web server has a limited number of threads available, and in high load situations all of the available threads might be in use. When that happens, the server can't process new requests until the threads are freed up. With synchronous code, many threads may be tied up while they aren't actually doing any work because they're waiting for I/O to complete. With asynchronous code, when a process is waiting for I/O to complete, its thread is freed up for the server to use for processing other requests. As a result, asynchronous code enables server resources to be used more efficiently, and the server can handle more traffic without delays.

In the following code, the async keyword, Task<T> return value, await keyword, and ToListAsync method make the code execute asynchronously.

- The async keyword tells the compiler to:
    - Generate callbacks for parts of the method body.
    - Create the Task object that's returned.
- The Task<T> return type represents ongoing work.
- The await keyword causes the compiler to split the method into two parts. The first part ends with the operation that's started asynchronously. The second part is put into a callback method that's called when the operation completes.
- ToListAsync is the asynchronous version of the ToList extension method.

Some things to be aware of when writing asynchronous code that uses EF Core:

- Only statements that cause queries or commands to be sent to the database are executed asynchronously. That includes ToListAsync, SingleOrDefaultAsync, FirstOrDefaultAsync, and SaveChangesAsync. It doesn't include statements that just change an IQueryable, such as var students = context.Students.Where(s => s.LastName == "Davolio").
- An EF Core context isn't thread safe: don't try to do multiple operations in parallel.
- To take advantage of the performance benefits of async code, verify that library packages (such as for paging) use async if they call EF Core methods that send queries to the database.