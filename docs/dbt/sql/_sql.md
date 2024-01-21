## SQL `SELECT` statement

It's the first SQL Command most developers encounter, and the most they'll ever use.  Easy to learn, hard to master, but with DBT you can go pretty far by just knowing:

* **The Big Six**
* **LEFT JOINs**
* **Set Operations**
* **CTEs**

### The **Big Six**
Consider an example SQL SELECT statement which returns an ordered count (starting with the most frequent) for first names of persons with 'Smith' as their last name:

!!! info inline end "SELECT ***(required)***"

    Declares the **columnar shape** (width) of the returned dataset. The columns, their calculations, and their names.

``` sql

select 

       count('x')         as total_count,
       spriden_first_name as first_name


from     ...

```
!!! info inline end "FROM ***(required)***"

    Identifies what **dataset(s)** to use for columns and logic. Can be tables, views, or other `SELECT` statements.

``` sql

select 

         ...

from saturn.spriden
where    ...

```

!!! info inline end "WHERE"

    Filtering-logic to **reduce** returned records.  Multiple logical components are commonly combined using `AND`, `OR`,  and usually where business logic is captured. 

``` sql

select 

         ...

from     ...
where spriden_change_ind is null
  and spriden_last_name = 'Smith'
  and spriden_entity_ind = 'P'
group by ...

```

!!! info inline end "GROUP BY"

    Grouping-logic to support columns in `SELECT` using aggregate-calculations, like `COUNT`. 
    
    When used, all non-aggregate columns in `SELECT` must be referenced in `GROUP BY` .

``` sql

select 


       count('x')         as total_count,
       spriden_first_name as first_name
       

from     ...
where    ...
group by spriden_last_name

```

!!! info inline end "HAVING"

    Filtering-logic which applies specifically to **aggregate-calculations** to reduce returned records.  Similar to `WHERE`, which applies to non-aggregates.

``` sql

select 

         ...

from     ...
where    ...
group by ...
having count('x') > 1

```
!!! info inline end "ORDER BY"

    Lastly, `ORDER BY` dictates how the `SELECT` list of columns are to be used in ordering rows of the returned datases, in precedent, and in order from 'high to low', datatype-depending.  While `SELECT` and `FROM` are required and all others optional, `ORDER BY` isn't dependant on other items like `GROUP BY` and `HAVING`.  

``` sql

select 

       count('x')         as total_count,
       spriden_first_name as first_name

from saturn.spriden
where spriden_change_ind is null
  and spriden_last_name = 'Smith'
  and spriden_entity_ind = 'P'
group by spriden_last_name
having count('x') > 1
order by count('x') desc

```
###LEFT JOINs###


###Set Operations###

###CTEs###

## Chaining SQL statements

Data transformations rarely need only ***one step***.  If a transformation only occurs in one step, then either you're a proud owner of a :unicorn:...

... or it's a transformation occuring in a **single** SQL statement that encapsulates many different steps and logic.  DBT is a tool that is built around a **highly opinionted, but also highly flexible** data transformation paradigm.

Consider the following as an example of such a SQL transformation to return a list of names and email addresses:

``` SQL
--SELECT COUNT('X'),spriden_pidm FROM (--(1)
SELECT spriden_pidm,
       NVL(spbpers.SPBPERS_PREF_FIRST_NAME, spriden_first_name) first_name,--(2)
       spriden_last_name,
       spbpers.SPBPERS_CONFID_IND,
       goremal.GOREMAL_EMAIL_ADDRESS
from saturn.spriden
left join saturn.spbpers
  on spbpers_pidm = spriden_pidm
left join general.goremal
  on goremal_pidm = spriden_pidm
and goremal_emal_code = 'UO'--(3)
and goremal_status_ind = 'A'
where spriden_change_ind is null
  and spriden_entity_ind = 'P'
--)GROUP BY spriden_pidm
--HAVING COUNT('X') > 1
```

1. This commented line, in conjunction with the two other commented lines at the end, represent artifacts of a **test for grain**.  That is, the maintainers of this code thought it important to test the returned dataset to be unique by `spriden_pidm`.
If SQL has these fragments left ***around***, it's usually an indicator this is something worth checking more than once.  This is a foreshadowing of **DBT Tests**...
2. This attempts to use a 'preferred first name' name over a different first name, implementing the `NVL` function.  This function works in Oracle, but not Microsoft SQL Server.  The `COALESCE` function works on most databases and acheives the same result.  Keeping SQL as platform-agnostic as can be has an array of benefits from multi-platform support to wider audience for adoption.  Note also the mixed-usage of upper/lowercases and explicit/lack-of table owners for fields used in the calculation. 
3. Here we see some business-specific logic used in `JOIN` criteria
