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

## Razor Pages with EF Core in ASP.NET Core - CRUD

### Update the Details page

The Include and ThenInclude methods cause the context to load the Student.Enrollments navigation property, and within each enrollment the Enrollment.Course navigation property. These methods are examined in detail in the Reading related data tutorial.

The AsNoTracking method improves performance in scenarios where the entities returned are not updated in the current context. AsNoTracking is discussed later in this tutorial.

### Ways to read one entity

The generated code uses FirstOrDefaultAsync to read one entity. This method returns null if nothing is found; otherwise, it returns the first row found that satisfies the query filter criteria. 'FirstOrDefaultAsync' is generally a better choice than the following alternatives:

- SingleOrDefaultAsync - Throws an exception if there's more than one entity that satisfies the query filter. To determine if more than one row could be returned by the query, 'SingleOrDefaultAsync' tries to fetch multiple rows. This extra work is unnecessary if the query can only return one entity, as when it searches on a unique key.
- FindAsync - Finds an entity with the primary key (PK). If an entity with the PK is being tracked by the context, it's returned without a request to the database. This method is optimized to look up a single entity, but you can't call 'Include' with FindAsync. So if related data is needed, 'FirstOrDefaultAsync' is the better choice.

### Update the Create page

#### TryUpdateModelAsync

The preceding code creates a Student object and then uses posted form fields to update the Student object's properties. The TryUpdateModelAsync method:

- Uses the posted form values from the PageContext property in the PageModel.
- Updates only the properties listed (s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate).
- Looks for form fields with a "student" prefix. For example, Student.FirstMidName. It's not case sensitive.
- Uses the model binding system to convert form values from strings to the types in the Student model. For example, EnrollmentDate has to be converted to DateTime.

### Overposting

Using 'TryUpdateModel' to update fields with posted values is a security best practice because it prevents overposting. For example, suppose the Student entity includes a 'Secret' property that this web page shouldn't update or add:

Even if the app doesn't have a 'Secret' field on the create or update Razor Page, a hacker could set the 'Secret' value by overposting. A hacker could use a tool such as Fiddler, or write some JavaScript, to post a 'Secret' form value. The original code doesn't limit the fields that the model binder uses when it creates a Student instance.

Whatever value the hacker specified for the 'Secret' form field is updated in the database. The following image shows the Fiddler tool adding the 'Secret' field (with the value "OverPost") to the posted form values.

#### View model

View models provide an alternative way to prevent overposting.

The application model is often called the domain model. The domain model typically contains all the properties required by the corresponding entity in the database. The view model contains only the properties needed for the UI that it is used for (for example, the Create page).

In addition to the view model, some apps use a binding model or input model to pass data between the Razor Pages page model class and the browser.

### Update the Edit page

The code changes are similar to the Create page with a few exceptions:

- 'FirstOrDefaultAsync' has been replaced with FindAsync. When you don't have to include related data, FindAsync is more efficient.
- 'OnPostAsync' has an id parameter.
- The current student is fetched from the database, rather than creating an empty student.

### Entity States

The database context keeps track of whether entities in memory are in sync with their corresponding rows in the database. This tracking information determines what happens when SaveChangesAsync is called. For example, when a new entity is passed to the AddAsync method, that entity's state is set to Added. When SaveChangesAsync is called, the database context issues a SQL INSERT command.

An entity may be in one of the following states:

Added: The entity doesn't yet exist in the database. The SaveChanges method issues an INSERT statement.

Unchanged: No changes need to be saved with this entity. An entity has this status when it's read from the database.

Modified: Some or all of the entity's property values have been modified. The SaveChanges method issues an UPDATE statement.

Deleted: The entity has been marked for deletion. The SaveChanges method issues a DELETE statement.

Detached: The entity isn't being tracked by the database context.

In a desktop app, state changes are typically set automatically. An entity is read, changes are made, and the entity state is automatically changed to Modified. Calling SaveChanges generates a SQL UPDATE statement that updates only the changed properties.

In a web app, the DbContext that reads an entity and displays the data is disposed after a page is rendered. When a page's OnPostAsync method is called, a new web request is made and with a new instance of the DbContext. Rereading the entity in that new context simulates desktop processing.

## Sort, filter, page and group

### Use dynamic LINQ to simplify code

The third tutorial in this series shows how to write LINQ code by hard-coding column names in a switch statement. With two columns to choose from, this works fine, but if you have many columns the code could get verbose. To solve that problem, you can use the 'EF.Property' method to specify the name of the property as a string. To try out this approach, replace the Index method in the StudentsController with the following code.