# G52_Fazliddin_Hoshimov_V1

CREATED BY @developer_f7 (Telegram)
## 1.  What is a Database?

**Database** — bu ma'lumotlarni tizimli, tartiblangan holatda saqlash uchun ishlatiladigan raqamli ombor. Ular real hayotdagi daftar yoki ro'yxatlarga o'xshaydi, lekin kompyuterda.

---

## 2. What is RDBMS?

**RDBMS (Relational Database Management System)** — munosabatli ma’lumotlar bazasi boshqaruv tizimi. Ma’lumotlar **jadval (table)** ko‘rinishida saqlanadi va ular o‘zaro bog‘langan bo‘ladi.

Misollar: PostgreSQL, MySQL, Oracle, MS SQL Server.

---

## 3.  DBMS va RDBMS Farqlari

| Xususiyat | DBMS | RDBMS |
|----------|------|--------|
| Saqlash usuli | Fayl yoki obyekt | Jadval ko‘rinishida |
| Bog‘liqlik | Kam yoki yo‘q | Jadval o‘rtasida kuchli aloqalar |
| Normalizatsiya | Shart emas | Muhim |
| Xavfsizlik | Oddiy | Yaxshi darajada |
| Misollar | XML DB, JSON DB | MySQL, PostgreSQL |

---

## 4.  What is SQL?

**SQL (Structured Query Language)** — bu ma’lumotlar bazasi bilan muloqot qilish uchun yaratilgan maxsus til.

### Ishlatish sohalari:

Ma’lumot kiritish (INSERT)

Ma’lumot o‘qish (SELECT)

O‘chirish (DELETE)

Yangilash (UPDATE)

Jadval va struktura yaratish yoki o‘zgartirish (CREATE, ALTER)

---

## 5.  Primary Key vs Foreign Key

**Primary Key** — jadvaldagi har bir satrni unikal aniqlovchi ustun. Takrorlanmas va bo‘sh bo‘lishi mumkin emas.

**Foreign Key** — boshqa jadvaldagi primary key’ga murojaat qiluvchi ustun. Jadval orasidagi **aloqani** ifodalaydi.

**Farqi:** Primary Key identifikatsiya uchun, Foreign Key esa bog‘lanish uchun ishlatiladi.

---

## 6.  What is PL/pgSQL?

**PL/pgSQL** — bu PostgreSQL uchun procedural til. SQL buyruqlarini `IF`, `LOOP`, `FUNCTION` kabi dasturlash konstruksiyalari bilan birlashtirish imkonini beradi.

Murakkab biznes mantiqni bazaning o‘zida avtomatlashtirish uchun ishlatiladi.

---

## 7.  What is an Assert Statement?

**Assert** — bu kod ishlayotganda ma'lum shartlar **haqiqat** ekanligini tekshiradi. Agar shart noto‘g‘ri bo‘lsa, xatolik chiqaradi.

Asosan testlar yoki debugging (nosozlik topish) vaqtida ishlatiladi.

---

## 8. What is a Trigger?

**Trigger** — bu bazadagi ma’lum voqealar sodir bo‘lganda **avtomatik tarzda ishga tushadigan** funksiya.

Masalan:
INSERT paytida log yozish

DELETE qilinsa, tarixga qo‘shish

Trigger’lar yordamida ma’lumotlar ustida avtomatik nazorat o‘rnatish mumkin.

---

## 9.  What are Transactions and ACID?

###  Transaction — bu bir nechta SQL operatsiyalarini **bir butun** sifatida bajarish.

###  ACID Principlari:
**Atomicity** — hammasi yoki hech narsa.

**Consistency** — tranzaktsiya oldi va keyin holat mantiqiy bo‘lishi kerak.

**Isolation** — tranzaktsiyalar bir-biriga xalaqit bermaydi.

**Durability** — yakunlangan tranzaktsiya natijasi doimiy saqlanadi.

---

## 10.  What is Indexing?

**Index** — bu ma’lumotlarni **tezroq izlash va olish** uchun maxsus struktura.

### Afzalliklari:

SELECT va WHERE so‘rovlarini sezilarli darajada tezlashtiradi

Ma’lumotga to‘g‘ridan-to‘g‘ri kirishni ta’minlaydi

### Kamchilik:

INSERT, UPDATE operatsiyalariga kichik sekinlik qo‘shadi

---


------------------------------- BU YERDA KODLAR -------------------------------

(mackoroo bn qoshdim ishladi lekn uni bu yerga yozib otrmadm)

CREATE TABLE roles (
 id SERIAL PRIMARY KEY,
 roleName VARCHAR UNIQUE NOT NULL CHECK (roleName IN ('STUDENT', 'MENTOR'))
);


CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  fullName VARCHAR CHECK (char_length(fullName) >= 5),

  phone VARCHAR(9) UNIQUE CHECK (char_length(phone) = 9),
  role_id INT,
  CONSTRAINT fk_role FOREIGN KEY (role_id)
      REFERENCES roles(id)
      ON DELETE SET NULL
);


CREATE TABLE groups (
  id SERIAL PRIMARY KEY,
  groupName VARCHAR CHECK (char_length(groupName) >= 3),
  mentor_id INT,
  createdAt DATE NOT NULL DEFAULT CURRENT_DATE,
  CONSTRAINT fk_mentor FOREIGN KEY (mentor_id)
      REFERENCES users(id)
      ON DELETE SET NULL
);


CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL,
  group_id INT NOT NULL,
  createdAt DATE NOT NULL DEFAULT CURRENT_DATE,
  active BOOLEAN NOT NULL DEFAULT FALSE,
  CONSTRAINT fk_student_user FOREIGN KEY (user_id)
      REFERENCES users(id)
      ON DELETE CASCADE,
  CONSTRAINT fk_student_group FOREIGN KEY (group_id)
      REFERENCES groups(id)
      ON DELETE CASCADE
);

CREATE OR REPLACE VIEW latest_10_students_view AS
SELECT
    s.id AS student_id,
    u.fullName AS student_fullName,
    s.createdAt AS student_createdAt,
    g.groupName,
    m.fullName AS mentor_fullName
FROM students s
         JOIN users u ON u.id = s.user_id
         JOIN groups g ON g.id = s.group_id
         LEFT JOIN users m ON g.mentor_id = m.id
ORDER BY s.createdAt DESC
LIMIT 10;

SELECT * FROM latest_10_students_view;

CREATE MATERIALIZED VIEW top_3_groups_mv AS
SELECT
    g.id AS group_id,
    g.groupName,
    g.createdAt AS group_createdAt,
    COUNT(s.id) AS studentCount
FROM groups g
         LEFT JOIN students s ON s.group_id = g.id
GROUP BY g.id
ORDER BY COUNT(s.id) DESC
LIMIT 3;


REFRESH MATERIALIZED VIEW top_3_groups_mv;

SELECT * FROM top_3_groups_mv;

CREATE OR REPLACE FUNCTION groupOfMentor(mentor_id_param INT)
    RETURNS TABLE (
                      mentor_id INT,
                      mentor_fullName VARCHAR,
                      groupName VARCHAR,
                      group_createdAt DATE,
                      studentCount INT
                  )
AS $$
BEGIN
    RETURN QUERY
        SELECT
            u.id,
            u.fullName,
            g.groupName,
            g.createdAt,
            COUNT(s.id)
        FROM users u
                 JOIN groups g ON g.mentor_id = u.id
                 LEFT JOIN students s ON s.group_id = g.id
        WHERE u.id = mentor_id_param
        GROUP BY u.id, u.fullName, g.groupName, g.createdAt;
END;
$$ LANGUAGE plpgsql;

SELECT * FROM groupOfMentor(1);

CREATE OR REPLACE PROCEDURE studentActivitor(group_id_param INT)
    LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE students
    SET active = true
    WHERE group_id = group_id_param AND active = false;
END;
$$;

CALL studentActivitor(3);


CREATE TABLE update_logs (
                             id SERIAL PRIMARY KEY,
                             table_name VARCHAR,
                             operation VARCHAR,
                             old_data JSONB,
                             new_data JSONB,
                             changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE OR REPLACE FUNCTION log_update_changes()
    RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO update_logs(table_name, operation, old_data, new_data)
    VALUES (
               TG_TABLE_NAME,
               TG_OP,
               row_to_json(OLD),
               row_to_json(NEW)
           );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_update_students
    AFTER UPDATE ON students
    FOR EACH ROW
EXECUTE FUNCTION log_update_changes();

CREATE TRIGGER log_update_groups
    AFTER UPDATE ON groups
    FOR EACH ROW
EXECUTE FUNCTION log_update_changes();

SELECT * FROM update_logs;










