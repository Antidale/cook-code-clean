+++
title = "Variable like queries in Postgres"
date = "2025-01-01"
+++

I'll start off with an acknowledgment that the title of this post probably isn't quite right. It took a few tries at searching out the information that I actually wanted, which was a way to write a query like the following, but have it adhere to Postgres' standards/synatax

```sql

    Declare @someVariable varchar(max) = 'My Value'

    select * 
    from SomeTable
    where SomeColumn = @someVariable

```

Postgres doesn't handle variables that way, and if you want to use them, you're using PLPGSQL and doing your work within a DO/END block, which carries some restrictions on what you can do with it. [Documentation.](https://www.postgresql.org/docs/current/plpgsql.html) (or you're doing something in psql)

So instead, general suggestions are to use either a CTE or a temp table to store what would be your variables and use them in your query. This [stack overflow](https://stackoverflow.com/questions/1490942/declare-a-variable-in-a-postgresql-query) post has examples of all the above methods (PLPGSQL, CTEs, temp tables, and a couple of other options besides).

I liked the CTE approach for the specific use-case I had, and it made it work more nicely in my brain when I went from a single value to updating a number of rows in the database. In the end, I had something like this to going:

```sql

    WITH data (userId, displayName, tournamentName) AS (
    values 
        ('472321239864705034', 'somename', 'test updates'),
        ('someId_1', 'somename_1', 'test updates'),
        ('someId_2', 'somename_2', 'test updates')
    )

    update tournament.registrations r
    set user_name_alias = d.displayName
    from tournament.tournament_registrations tr
    join data d on d. userId = tr.user_id 
    where tr.tournament_id = r.tournament_id 
    and tr.user_id = d.userId
    and tr.entrant_id = r.entrant_id
    and (d.tournamentName = '' 
        or tr.tournament_name = d.tournamentName)

```

The downside of using a common table expression is that you can't re-use it later on if you're doing multiple queries. IN case you do need to re-use your setup, you'd change to inserting the data into a temp table, and just move on from there. Temp tables have their own issues, so, you know, pick what's right for you.

All of this was needed because I give myself lots of room in my schema design to shoot myslef in the foot just this way. I probably should have just designed the system so that the display name is just per user, but instead I have it per tournament the user is registered for.

Also could have avoided figuring this out in production with better testing, either automated or manual. That's for a different day, though.