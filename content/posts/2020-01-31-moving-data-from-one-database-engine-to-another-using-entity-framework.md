---
title: Moving Data From One Database Engine to Another Using Entity Framework Core
date: 2021-03-03 22:08 -500
draft: false
---

Sometimes a project's requirements change and you need to change the
database engine in which you store your data. In my case, I needed to
switch from PostgreSQL to Microsoft SQL Server. Changing some 
configuration is easy, but sometimes moving data can be a challenge.

The main challenge I have faced when moving data between data stores, 
whether it be from MySQL to PostgreSQL or PostgreSQL to Microsoft SQL
Server is the encoding between databases. This is how characters are
handled, in strings. It can cause issues when moving data, using
conventional methods. In addition to it being easier, I find it more
enjoyable to use code, rather than spending time manually working with
the data. 

Now, let's get down to business looking over a code sample. The code
sample below is a simple .NET Core Console that connects to one 
database via EntityFrameworkCore and stores the data in a List<> data
type, to then write it back to the new database.


```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace DataConverter
{
    class Program
    {
        static string NpgsqlConnectionString => Environment.GetEnvironmentVariable("PSQL_CONN");

        static string SqlServerConnectionString => Environment.GetEnvironmentVariable("MSSQL_CONN");

        static async Task Main(string[] args)
        {
            Console.WriteLine("Converting Data");

            List<User> users = await RetrieveUsers();
            await WriteUsers(users);
        }

        static async Task<List<User>> RetrieveUsers()
        {
            DbContextOptions opts = new DbContextOptionsBuilder()
                .UseNpgsql(NpgsqlConnectionString)
                .Options;

            List<User> users;

            using (var context = new DataConverterContext(opts))
            {
                users = await context.Users.ToListAsync();
            }

            return users;
        }

        static async Task<int> WriteUsers(List<User> users)
        {
            DbContextOptions opts = new DbContextOptionsBuilder()
                .UseSqlServer(SqlServerConnectionString)
                .Options;

            using (var context = new DataConverterContext(opts))            
            {
                context.AddRange(users);
                await context.SaveChangesAsync();
            }
            
            return 0;
        }
    }
}
```

The DbContext to configure this setup is simple as well. It is just a
context with a single property and the entity configuration.

```cs

  
using Microsoft.EntityFrameworkCore;

namespace DataConverter
{
    public class DataConverterContext : DbContext
    {
        public DbSet<User> Users { get; set; }

        public DataConverterContext(DbContextOptions opts) : base(opts) { }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.ApplyConfiguration(new UserConfiguration());
        }
    }
}
```

Overall, if you are facing issues migrating data from one database
engine to another, give your ORM (Object Relational Mapper) a try.

If you want to see the working solution, you can see the full project
here,  https://github.com/rebelweb/DataConverter 