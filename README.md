# DatabasesSession
Выполненное сессионное задание по предмету "Базы данных"

Задача 1.1

```sql
SELECT v.maker, m.model
FROM Motorcycle m
JOIN Vehicle v ON m.model = v.model
WHERE m.horsepower > 150
  AND m.price < 20000
  AND m.type = 'Sport'
ORDER BY m.horsepower DESC;
```

Задача 1.2

```sql
SELECT 
    v.maker,
    v.model,
    c.horsepower,
    c.engine_capacity,
    'Car' AS type
FROM Car c
JOIN Vehicle v ON c.model = v.model
WHERE c.horsepower > 150
  AND c.engine_capacity < 3.0
  AND c.price < 35000.00

UNION ALL

SELECT 
    v.maker,
    v.model,
    m.horsepower,
    m.engine_capacity,
    'Motorcycle' AS type
FROM Motorcycle m
JOIN Vehicle v ON m.model = v.model
WHERE m.horsepower > 150
  AND m.engine_capacity < 1.5
  AND m.price < 20000.00

UNION ALL

SELECT 
    v.maker,
    v.model,
    NULL AS horsepower,
    NULL AS engine_capacity,
    'Bicycle' AS type
FROM Bicycle b
JOIN Vehicle v ON b.model = v.model
WHERE b.gear_count > 18
  AND b.price < 4000.00

ORDER BY horsepower DESC NULLS LAST;
```

Задача 2.1

```sql
WITH car_stats AS (
    SELECT 
        c.name,
        c.class,
        AVG(r.position) AS avg_position,
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    GROUP BY c.name, c.class
),
min_avg_per_class AS (
    SELECT 
        class,
        MIN(avg_position) AS min_avg_position
    FROM car_stats
    GROUP BY class
)
SELECT 
    cs.name,
    cs.class,
    cs.avg_position,
    cs.race_count
FROM car_stats cs
JOIN min_avg_per_class mac ON cs.class = mac.class AND cs.avg_position = mac.min_avg_position
ORDER BY cs.avg_position;
```

Задача 2.2

```sql
WITH car_stats AS (
    SELECT 
        c.name,
        c.class,
        cl.country,
        AVG(r.position) AS avg_position,
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
)
SELECT 
    name,
    class,
    country,
    avg_position,
    race_count
FROM car_stats
WHERE avg_position = (SELECT MIN(avg_position) FROM car_stats)
ORDER BY name
LIMIT 1;
```

Задача 2.3

```sql
WITH car_stats AS (
    SELECT 
        c.name,
        c.class,
        cl.country,
        AVG(r.position) AS avg_position,
        COUNT(r.race) AS car_race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
),
class_avg AS (
    SELECT 
        class,
        AVG(avg_position) AS class_avg_position
    FROM car_stats
    GROUP BY class
),
min_class_avg AS (
    SELECT MIN(class_avg_position) AS min_avg
    FROM class_avg
),
selected_classes AS (
    SELECT class
    FROM class_avg
    WHERE class_avg_position = (SELECT min_avg FROM min_class_avg)
),
class_race_count AS (
    SELECT 
        c.class,
        COUNT(DISTINCT r.race) AS total_races
    FROM Cars c
    JOIN Results r ON c.name = r.car
    WHERE c.class IN (SELECT class FROM selected_classes)
    GROUP BY c.class
)
SELECT 
    cs.name,
    cs.class,
    cs.avg_position,
    cs.car_race_count,
    cs.country,
    crc.total_races
FROM car_stats cs
JOIN selected_classes sc ON cs.class = sc.class
JOIN class_race_count crc ON cs.class = crc.class
ORDER BY cs.class, cs.name;
```

Задача 2.4

```sql
WITH car_stats AS (
    SELECT 
        c.name,
        c.class,
        cl.country,
        AVG(r.position) AS avg_position,
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
),
class_stats AS (
    SELECT 
        class,
        AVG(avg_position) AS class_avg_position,
        COUNT(name) AS car_count
    FROM car_stats
    GROUP BY class
    HAVING COUNT(name) >= 2
)
SELECT 
    cs.name,
    cs.class,
    cs.avg_position,
    cs.race_count,
    cs.country
FROM car_stats cs
JOIN class_stats cls ON cs.class = cls.class
WHERE cs.avg_position < cls.class_avg_position
ORDER BY cs.class, cs.avg_position;
```

Задача 2.5

```sql
WITH car_stats AS (
    SELECT 
        c.name,
        c.class,
        cl.country,
        AVG(r.position) AS avg_position,
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
),
class_low_performance AS (
    SELECT 
        class,
        COUNT(*) AS low_performance_count
    FROM car_stats
    WHERE avg_position > 3.0
    GROUP BY class
),
max_low_performance AS (
    SELECT MAX(low_performance_count) AS max_count
    FROM class_low_performance
),
selected_classes AS (
    SELECT class
    FROM class_low_performance
    WHERE low_performance_count = (SELECT max_count FROM max_low_performance)
),
class_total_races AS (
    SELECT 
        c.class,
        COUNT(DISTINCT r.race) AS total_races
    FROM Cars c
    JOIN Results r ON c.name = r.car
    WHERE c.class IN (SELECT class FROM selected_classes)
    GROUP BY c.class
)
SELECT 
    cs.name,
    cs.class,
    cs.avg_position,
    cs.race_count,
    cs.country,
    ctr.total_races
FROM car_stats cs
JOIN selected_classes sc ON cs.class = sc.class
JOIN class_total_races ctr ON cs.class = ctr.class
WHERE cs.avg_position > 3.0
ORDER BY 
    (SELECT low_performance_count FROM class_low_performance WHERE class = cs.class) DESC,
    cs.class,
    cs.avg_position;
```

Задача 3.1

```sql
WITH customer_bookings AS (
    SELECT 
        c.ID_customer,
        c.name,
        c.email,
        c.phone,
        COUNT(DISTINCT b.ID_booking) AS total_bookings,
        AVG(b.check_out_date - b.check_in_date) AS avg_stay_duration_days,
        STRING_AGG(DISTINCT h.name, ', ' ORDER BY h.name) AS hotels
    FROM Customer c
    JOIN Booking b ON c.ID_customer = b.ID_customer
    JOIN Room r ON b.ID_room = r.ID_room
    JOIN Hotel h ON r.ID_hotel = h.ID_hotel
    GROUP BY c.ID_customer, c.name, c.email, c.phone
    HAVING COUNT(DISTINCT b.ID_booking) > 2
        AND COUNT(DISTINCT r.ID_hotel) > 1
)
SELECT 
    name,
    email,
    phone,
    total_bookings,
    hotels,
    ROUND(avg_stay_duration_days, 2) AS avg_stay_duration
FROM customer_bookings
ORDER BY total_bookings DESC;
```

Задача 3.2

```sql
WITH customer_spending AS (
    SELECT 
        c.ID_customer,
        c.name,
        COUNT(DISTINCT b.ID_booking) AS total_bookings,
        SUM(r.price * (b.check_out_date - b.check_in_date)) AS total_spent,
        COUNT(DISTINCT h.ID_hotel) AS unique_hotels
    FROM Customer c
    JOIN Booking b ON c.ID_customer = b.ID_customer
    JOIN Room r ON b.ID_room = r.ID_room
    JOIN Hotel h ON r.ID_hotel = h.ID_hotel
    GROUP BY c.ID_customer, c.name
),
qualified_customers AS (
    SELECT 
        ID_customer,
        name,
        total_bookings,
        total_spent,
        unique_hotels
    FROM customer_spending
    WHERE total_bookings > 2
      AND unique_hotels > 1
)
SELECT 
    ID_customer,
    name,
    total_bookings,
    total_spent,
    unique_hotels
FROM qualified_customers
WHERE total_spent > 500
ORDER BY total_spent ASC;
```

Задача 3.3

```sql
WITH hotel_avg_prices AS (
    SELECT 
        h.ID_hotel,
        h.name,
        h.location,
        AVG(r.price) AS avg_price,
        CASE 
            WHEN AVG(r.price) < 175 THEN 'Дешевый'
            WHEN AVG(r.price) BETWEEN 175 AND 300 THEN 'Средний'
            ELSE 'Дорогой'
        END AS hotel_category
    FROM Hotel h
    JOIN Room r ON h.ID_hotel = r.ID_hotel
    GROUP BY h.ID_hotel, h.name, h.location
),
customer_hotels AS (
    SELECT DISTINCT
        c.ID_customer,
        c.name,
        h.ID_hotel,
        h.name AS hotel_name,
        hp.hotel_category
    FROM Customer c
    JOIN Booking b ON c.ID_customer = b.ID_customer
    JOIN Room r ON b.ID_room = r.ID_room
    JOIN Hotel h ON r.ID_hotel = h.ID_hotel
    JOIN hotel_avg_prices hp ON h.ID_hotel = hp.ID_hotel
),
customer_category AS (
    SELECT 
        ID_customer,
        name,
        MAX(CASE 
            WHEN hotel_category = 'Дорогой' THEN 3
            WHEN hotel_category = 'Средний' THEN 2
            WHEN hotel_category = 'Дешевый' THEN 1
            ELSE 0
        END) AS category_rank,
        STRING_AGG(DISTINCT hotel_name, ', ' ORDER BY hotel_name) AS visited_hotels
    FROM customer_hotels
    GROUP BY ID_customer, name
)
SELECT 
    ID_customer,
    name,
    CASE 
        WHEN category_rank = 3 THEN 'дорогой'
        WHEN category_rank = 2 THEN 'средний'
        WHEN category_rank = 1 THEN 'дешевый'
    END AS preferred_hotel_type,
    visited_hotels
FROM customer_category
WHERE category_rank > 0
ORDER BY category_rank ASC;
```

Задача 4.1

```sql
WITH RECURSIVE EmployeeHierarchy AS (
    SELECT 
        EmployeeID,
        Name,
        ManagerID,
        DepartmentID,
        RoleID
    FROM Employees
    WHERE EmployeeID = 1
    
    UNION ALL
	
    SELECT 
        e.EmployeeID,
        e.Name,
        e.ManagerID,
        e.DepartmentID,
        e.RoleID
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT 
    eh.EmployeeID,
    eh.Name,
    eh.ManagerID,
    d.DepartmentName AS DepartmentName,
    r.RoleName AS RoleName,
    (
        SELECT STRING_AGG(p.ProjectName, ', ' ORDER BY p.ProjectName)
        FROM Projects p
        WHERE p.DepartmentID = eh.DepartmentID
    ) AS ProjectNames,
    (
        SELECT STRING_AGG(t.TaskName, ', ' ORDER BY t.TaskName)
        FROM Tasks t
        WHERE t.AssignedTo = eh.EmployeeID
    ) AS TaskNames
FROM EmployeeHierarchy eh
LEFT JOIN Departments d ON eh.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON eh.RoleID = r.RoleID
ORDER BY eh.Name;
```

Задача 4.2

```sql
WITH RECURSIVE EmployeeHierarchy AS (
    SELECT 
        EmployeeID,
        Name,
        ManagerID,
        DepartmentID,
        RoleID
    FROM Employees
    WHERE EmployeeID = 1
    
    UNION ALL
    
    SELECT 
        e.EmployeeID,
        e.Name,
        e.ManagerID,
        e.DepartmentID,
        e.RoleID
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
),
TaskCounts AS (
    SELECT 
        AssignedTo,
        COUNT(*) AS total_tasks
    FROM Tasks
    WHERE AssignedTo IS NOT NULL
    GROUP BY AssignedTo
),
SubordinateCounts AS (
    SELECT 
        ManagerID,
        COUNT(*) AS total_subordinates
    FROM Employees
    WHERE ManagerID IS NOT NULL
    GROUP BY ManagerID
)
SELECT 
    eh.EmployeeID,
    eh.Name,
    eh.ManagerID,
    d.DepartmentName AS DepartmentName,
    r.RoleName AS RoleName,
    (
        SELECT STRING_AGG(p.ProjectName, ', ' ORDER BY p.ProjectName)
        FROM Projects p
        WHERE p.DepartmentID = eh.DepartmentID
    ) AS ProjectNames,
    (
        SELECT STRING_AGG(t.TaskName, ', ' ORDER BY t.TaskName)
        FROM Tasks t
        WHERE t.AssignedTo = eh.EmployeeID
    ) AS TaskNames,
    COALESCE(tc.total_tasks, 0) AS TotalTasks,
    COALESCE(sc.total_subordinates, 0) AS TotalSubordinates
FROM EmployeeHierarchy eh
LEFT JOIN Departments d ON eh.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON eh.RoleID = r.RoleID
LEFT JOIN TaskCounts tc ON eh.EmployeeID = tc.AssignedTo
LEFT JOIN SubordinateCounts sc ON eh.EmployeeID = sc.ManagerID
ORDER BY eh.Name;
```

Задача 4.3

```sql
WITH RECURSIVE SubordinateHierarchy AS (
    SELECT 
        EmployeeID,
        ManagerID
    FROM Employees
    WHERE ManagerID IS NOT NULL
    
    UNION ALL
    
    SELECT 
        e.EmployeeID,
        sh.ManagerID
    FROM Employees e
    INNER JOIN SubordinateHierarchy sh ON e.ManagerID = sh.EmployeeID
),
SubordinateCounts AS (
    SELECT 
        ManagerID,
        COUNT(DISTINCT EmployeeID) AS total_subordinates
    FROM SubordinateHierarchy
    GROUP BY ManagerID
),
Managers AS (
    SELECT DISTINCT
        ManagerID AS EmployeeID
    FROM SubordinateHierarchy
)
SELECT 
    e.EmployeeID,
    e.Name,
    e.ManagerID,
    d.DepartmentName AS DepartmentName,
    r.RoleName AS RoleName,
    (
        SELECT STRING_AGG(p.ProjectName, ', ' ORDER BY p.ProjectName)
        FROM Projects p
        WHERE p.DepartmentID = e.DepartmentID
    ) AS ProjectNames,
    (
        SELECT STRING_AGG(t.TaskName, ', ' ORDER BY t.TaskName)
        FROM Tasks t
        WHERE t.AssignedTo = e.EmployeeID
    ) AS TaskNames,
    COALESCE(sc.total_subordinates, 0) AS TotalSubordinates
FROM Managers m
JOIN Employees e ON m.EmployeeID = e.EmployeeID
LEFT JOIN Departments d ON e.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON e.RoleID = r.RoleID
LEFT JOIN SubordinateCounts sc ON e.EmployeeID = sc.ManagerID
WHERE r.RoleName = 'Менеджер'
ORDER BY e.Name;
```
