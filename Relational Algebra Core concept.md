###### @princebillyGK

# Relational Algebra Core concept:

In relational algebra when you select multiple rows the duplicate rows are automatically eliminated.  As relational algebra is based on set theory.

but when using SQL all rows are selected included duplicate rows by default. we use 'DISTINCT' keyword to select unique rows in MYSQL.

## Sample scheme:

![example scheme](https://imgur.com/nOwafQw.png)

#### σ (sigma) - select row (WHERE*)

*<sup>(1)</sup>select all columns from students table who have GPA greater than or equal to 4.00* 

σ <sub>GPA ≥ 4.00</sub> Student

```mysql
SELECT * FROM STUDENT WHERE GPA >= 4.00;
```



#### Π (projection) - select column (SELECT)

*<sup>(2)</sup>select Id and name columns from students table who have GPA greater than or equal to 4.00* 

Π<sub>sID, sName</sub> (σ <sub>GPA ≥ 4.00</sub> Student)

```mysql
SELECT sID, sName FROM STUDENT WHERE GPA >= 4.00;
```



#### x Cross Product ( ≈ ⋈ <sub> **θ**</sub>Theta join) (INNER JOIN)

*<sup>(3)</sup>select id, name and GPA who has high school  > 1000  applied for cs major and got rejected.* 

Π<sub>Student.sID, sName, GPA (</sub>σ <sub>Student.sID = Apply.sID ^ HS > 1000 ^ major = 'cs' ^ dec = false </sub>(Student  x Apply))

**Alternatives:**
$$
\pi_{sName, GPA}(\sigma_{Student.sID=Apply.sID}(\sigma_{HS\gt1000}(Student) \times \sigma_{major=`CS` \wedge dec=`R`}(Apply)))
$$

$$
\pi_{sName, GPA}(\sigma_{Student.sID=Apply.sID \wedge HS \gt 1000 \wedge major=`CS` \wedge dec=`R`}(Student \times \pi_{sID, major, dec}Apply))
$$



```mysql
SELECT Student.sID, sName, GPA FROM Student JOIN Apply ON Student.sID = Apply.sID WHERE HS > 1000 AND MAJOR = 'CS' AND dec = false;
```



#### ⋈ Natural join (natural join)

- Eliminate duplicate column/attributes
- Enforce equality on the value of attribute/column value.

*<sup>(3)</sup>select id, name and GPA who has high school  > 1000  applied for cs major and got rejected.* 

Π<sub>sID, sName, GPA (</sub>σ <sub> HS > 1000 ^ major = 'cs' ^ dec = false </sub>(Student ⋈ Apply))

```mysql
SELECT sID, sName, GPA FROM Student NATURAL JOIN Apply WHERE HS > 1000 AND MAJOR = 'CS' AND dec = false;
```

<sup>(4)</sup>select id, name and GPA who has high school > 1000  applied for cs major and got rejected and paid 200000 for college enrollment.* 

Π<sub>sID, sName, GPA (</sub>σ <sub> HS > 1000 ^ major = 'cs' ^ dec = false^enr>20000 </sub>(Student ⋈ Apply  ⋈ College))

```mysql
SELECT sID, sName, GPA FROM Student NATURAL JOIN Apply NATURAL JOIN College WHERE HS > 1000 AND MAJOR = 'CS' AND dec = false and enr > 20000;
```



### Set operators

- **Union** operator (U) (UNION) **[When we want to keep duplicate UNION ALL in mysql]**
- **Difference** operator (-) (MINUS)
- **Intersection** operator (**∩**) (INTERSECT)
  - **Formula:** A**∩**B = A - (A-B)

*<sup>(5)</sup>Find the names of all students who did not apply to major in CS or EE*
$$
\pi_{sName}(Student \Join (\pi_{sID}Student - (\pi_{sID}(\sigma_{major=`CS`}Apply) \cup \pi_{sID}(\sigma_{major=`EE`}Apply))))
$$

$$
\pi_{sName}(Student \Join (\pi_{sID}Student - \pi_{sID}(\sigma_{major=`CS` \vee major=`EE`}Apply)))
$$



```mysql
SELECT 
    sName
FROM
    (SELECT 
        sName
    FROM
        Student
    NATURAL JOIN 
    (SELECT 
        sID
    FROM
        Student
    NATURAL JOIN Apply
    WHERE
        major = 'CS' OR major = 'ee'));
```



### ρ - Rename Operator(AS):

<sup>(3)</sup>select id, name and GPA who has high school  > 1000  applied for cs major and got rejected.* 

Π<sub>s.sID, sName, GPA (</sub>σ <sub>Student.sID = Apply.sID ^ HS > 1000 ^ major = 'cs' ^ dec = false </sub>(ρ<sub>s</sub>(Student)  x ρ<sub>a</sub>(Apply))

```mysql
SELECT s.sID, sName, GPA FROM Student s JOIN Apply a ON s.sID = a.sID WHERE HS > 1000 AND MAJOR = 'CS' AND dec = false;
```



*<sup>(6)</sup>Select pairs of college in same state*

> **incorrect :** <span style="color:red"> **σ (ρ<sub>c1(n1,s,e1)</sub>(College) ⋈ ρ<sub>c2(n2,s,e2)</sub>(College))**</span> 
>
> <span style="color:orange">⚠️ *will include pair containing same values  eg. (Berkly, Berkly)*</span>
>
> <span style="color:orange">⚠️ *will contain duplicate pairs eg. {(Berkly, Torrento),(Torrento, Berkly)}*</span>
>
> **incorrect :** **<span style="color:red"> σ<sub>n1≠n2</sub> (ρ<sub>c1(n1,s,e1)</sub>(College) ⋈ ρ<sub>c2(n2,s,e2)</sub>(College))</span>** 
>
> <span style="color:orange">-⚠️ *will contain duplicate pairs {(Berkly, Torrento),(Torrento, Berkly)}*</span>



<span style="color:green">**Correct:**</span> σ**<sub>n1<n2</sub>** (ρ<sub>c1(n1,s,e1)</sub>(College) ⋈ ρ<sub>c2(n2,s,e2)</sub>(College))

### Modularizing Expression:

Consider the following Expression:



we can extract the relation like this:

**c1:=** ρ<sub>c1(n1,s,e1)</sub>(College)

**c2:=** ρ<sub>c2(n2,s,e2)</sub>(College)

**c3:=**  σ**<sub>n1<n2</sub>** (c1 ⋈ c2)