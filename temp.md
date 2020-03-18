Table of content

- [Pagination in oracle 11g](#pagination-in-oracle-11g)
- [Pagination in oracle 12g](#pagination-in-oracle-12g)
- [Pagination using spring data jpa](#pagination-using-spring-data-jpa)
  - [Static pagination](#static-pagination)
  - [Dynamic pagination](#dynamic-pagination)

# Pagination in oracle 11g 
```
based on 
1. pageno. and 
2. no of records
```

```sql
pageno: 0 (offset 0), no of records (limit) : 4
select m.* from (select * from (select t.*, rownum rnum from (select * from emp order by ename) t) where rnum>4*0 /* page no. 3 means 4*3=12 i.e., offset 12 records means after removing first 12 records*/) m where rownum<=4 /* no. of rows*/;
pageno: 1 (offset 4), no of records (limit) : 4
select m.* from (select * from (select t.*, rownum rnum from (select * from emp order by ename) t) where rnum>4*1 /* page no. 3 means 4*3=12 i.e., offset 12 records means after removing first 12 records*/) m where rownum<=4 /* no. of rows*/;
pageno: 2 (offset 8), no of records (limit) : 4
select m.* from (select * from (select t.*, rownum rnum from (select * from emp order by ename) t) where rnum>4*2 /* page no. 3 means 4*3=12 i.e., offset 12 records means after removing first 12 records*/) m where rownum<=4 /* no. of rows*/;
pageno: 3 (offset 12), no of records (limit) : 4
select m.* from (select * from (select t.*, rownum rnum from (select * from emp order by ename) t) where rnum>4*3 /* page no. 3 means 4*3=12 i.e., offset 12 records means after removing first 12 records*/) m where rownum<=4 /* no. of rows*/;
```
formatted sql query
```sql 
select m.* from (                                        -- 4th query
    select * from                                         -- 3rd inner query
        (
            select t.*, rownum rnum from                 -- 2nd inner query
                (select * from emp order by ename) t     -- 1st inner query
        )
    where rnum>4*3) m 
where rownum<=4;
```

Steps to create above query
1. 1st inner most query (to sort employees based on ename)
2. 2nd inner most query (to add column with sequence no. using rownum)
3. 3rd inner most query (removing rows using sequnece no. (offset specified records))
4. 4th outer query (first 4 rows)

# Pagination in oracle 12g
```sql 
select * from emp 
    offset 4*1 rows 
    fetch next 4 rows only;
```

# Pagination using spring data jpa 

## Static pagination
entity class
```java    
@Entity 
class Employee { -- - }
```
repository
```java
interface MyRepo extends JPARepository<Employee, Long>{}
```
In service
```java
Page<Employee> pagedResult = repo.findAll(pageable);
if(pagedResult.hasContent()) { return pagedResult.getContent(); } else { return new ArrayList<EmployeeEntity>(); }
```

**pageable object**
```java
Pageable pageable = PageRequest.of(pageNo, pageSize, Sort.by(sortBy)); // pageNo=1, pageSize=4, sortBy=ename
```
or 
```java
//another to ge pageable object
    Pageable pageable = PageRequest.of(0, 20, Sort.by("firstName"));
    Pageable pageable = PageRequest.of(0, 20, Sort.by("fistName").ascending().and(Sort.by("lastName").descending());
``` 
or 
```java   
    new PageRequest(1, 10,  new Sort(Sort.Direction.ASC, "ename") .and(new Sort(Sort.Direction.DESC, "job"));
```   

## Dynamic pagination
(when to use: when query is complex to create using spring data jpa then use native query)
```java
@Autowired
private EntityManager em;

String query = "complex native query which ca includes inner joins also"; // but it return Employee object fields only

Query nativeQuery = em.createNativeQuery(query);
nativeQuery.setParameter("collonAbcParameter", 1234);

//Paginering
nativeQuery.setFirstResult(offset);
nativeQuery.setMaxResults(limit);

final List<Object[]> resultList = nativeQuery.getResultList();
```
