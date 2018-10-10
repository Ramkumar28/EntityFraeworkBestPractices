# EntityFraeworkBestPractices

Entity Framework Best Practices.
1. AsNoTracking()

When a code is simple returning without modifying or adding, we can avoid tracking cost.

Bad

    var person = dbContext.Persons.where(x=>x.PersonId == Person.Id).SingleOrDefault();

Good

    var person = dbContext.Persons.AsNoTracking().where(x=>x.PersonId == Person.Id).SingleOrDefault();

2. Queries at Database and not at memory

BAD

Suppose we want to count how many counties are there in Texas:

    dbContext.States.Single(s => s.Code == “tx”).Counties.Count();

The query is correct, but inefficient. The line States.Single() loads states from the Database, next the counties loads all the items with all of their fields as a next query, and Count() is performed on the results.

So, we have lot of data which are not required, and here is a good solution.

GOOD

    dbContext.Counties.Count(c => c.State.Code == “tx”);

this is translated into SQL to a count and join rather.
3. Load only Required Data

One problem often we are encountering is loading all unnecessary data.

Let’s say I have a model named Person which has fields like name,address,LocationId,City,State etc. Now I want only address,

BAD

    var person= dbContext.Persons.AsNoTracking().Where(p=>p.person.name == person_name).SingleOrDefault();

return person.Address;

GOOD

    var address= dbContext.Persons.Where(p=>p.name == person_name).Select(x=>x.Address).SingleOrDefault();

    return address;

4. Queries in Async and in parallel

When you want to query in asynchrounous manner, you want to execute it as multiple queries at same time and not in same context.

BAD

    using(var context = new DbContext())

    {

    var result1 = await context.Set<TResult1>().ToListAsync().ConfigureAwait(false);

    var result2 = await context.Set<TResult1>().ToListAsync().ConfigureAwait(false);

    }
