# PostgreSQL db_normalized
## Оглавление
1. [Описание структуры базы данных](#описание-структуры-базы-данных)
2. [Запрос 1. Получение всех имен и фамилий людей в определенном кабинете](#запрос-1-получение-всех-имен-и-фамилий-людей-в-определенном-кабинете)
3. [Запрос 2. Добавление нового телефона сотруднику](#запрос-2-добавление-нового-телефона-сотруднику)
4. [Запрос 3. Получение списка телефонов всех сотрудников определенной должности в одном из отделов](#запрос-3-получение-списка-телефонов-всех-сотрудников-определенной-должности-в-одном-из-отделов)
5. [Запрос 4. Добавление нового сотрудника](#запрос-4-добавление-нового-сотрудника)
6. [Запрос 5. Перемещение сотрудника в другой отдел со сменой комнаты](#запрос-5-перемещение-сотрудника-в-другой-отдел-со-сменой-комнаты)
## Описание структуры базы данных
С целью оптимизации времени работы запросов было принято решение исходную таблицу db_normalized разбить на 4 таблицы:
1. Contacts
2. Departments
3. Employees
4. Positions

В таблице Contacts содержатся контактные данные сотрудников (номера их телефонов). У одного сотрудника может быть несколько номеров телефонов.</br>
В то же время, из таблицы Employee был удален столбец Phone.</br>
В таблице Departments содержатся данные об отделах организации (id и наименование отдела).</br>
В таблице Employee все текстовые значения поля Department были заменены на числовые (id).</br>
В таблице Positions содержатся данные о должностях внутри организации (id и наименование должности).</br>
В таблице Employee все текстовые значения поля Position были заменены на числовые (id).</br>
В исходной таблице также были удалены дубликаты строк. Таким образом, количество записей в таблице Employees сократилось до 67143.</br>
Для каждого запроса из задания была создана процедура или функция. Это позволит быстро получить доступ к желаемому результату.

## Запрос 1. Получение всех имен и фамилий людей в определенном кабинете
Запрос (функция) выглядит следующим образом:
```
drop function if exists get_name_in_room;
create or replace function get_name_in_room (
  current_room int
) 
	returns table (
		f_name varchar(15),
		l_name varchar(15)
	) 
	language plpgsql
as $$
begin
	return query 
		select first_name, last_name
		from employees
		where room = current_room;
end;$$
```
Пример вызова функции:
```
select * from get_name_in_room(100);
```

## Запрос 2. Добавление нового телефона сотруднику
Запрос (процедура) выглядит следующим образом:
```
drop procedure if exists add_phone;
create procedure add_phone(emp_id int, phone_number varchar(12))
LANGUAGE SQL
AS $BODY$
    INSERT INTO contacts VALUES (emp_id, phone_number);
$BODY$;
```
Пример вызова процедуры:
```
call add_phone(0, '+79999999999');
```
## Запрос 3. Получение списка телефонов всех сотрудников определенной должности в одном из отделов
Запрос (функция) выглядит следующим образом:
```
drop function if exists get_employee_list;
create or replace function get_employee_list (
  pos_name varchar(15), dep_name varchar(20)
) 
	returns table (
		phone_number varchar(12)
	) 
	language plpgsql
as $$
begin
	return query 
		select c.phone from employees e 
		join contacts c on c.employee_id = e.id
		join departments d on d.id = e.department_id
		join positions p on p.id = e.position_id
		where position_name = pos_name and department_name = dep_name;
end;$$
```
Пример вызова функции:
```
select * from get_employee_list('Intern', 'IT');
```
## Запрос 4. Добавление нового сотрудника
Запрос (процедура) выглядит следующим образом:
```
drop procedure if exists add_new_employee;
create procedure add_new_employee
(
	emp_id int, 
	dep_id int,
	emp_email varchar(50),
	f_name VARCHAR(15),
	l_name VARCHAR(15),
	pos_id INT,
	room_number INT
)
LANGUAGE SQL
AS $BODY$
    INSERT INTO employees VALUES
	(emp_id, dep_id, emp_email, f_name, l_name, pos_id, room_number);
$BODY$;
```
Пример вызова процедуры:
```
call add_new_employee(100000, 1, 'alex@yandex.ru', 'Alex', 'Ingrosso', 1, 100);
```
## Запрос 5. Перемещение сотрудника в другой отдел со сменой комнаты
Запрос (процедура) выглядит следующим образом:
```
drop procedure if exists move_employee_to_department;
create procedure move_employee_to_department
(
	emp_id int, 
	dep_name varchar(20),
	room_number INT
)
LANGUAGE SQL
AS $BODY$
    update employees set 
	department_id = (select id from departments where department_name = dep_name), 
	room = room_number
	where id = emp_id
$BODY$;
```
Пример вызова процедуры:
```
call move_employee_to_department(100000, 'IT', 110);
```
