-- Criação de database simples para teste 01:

create table students(
	id int,
	name text,
	enrolled_at date,
	course_id int
	);

insert into students values('0','Roberta','2024-08-12','1'),
							('1','Renato','2024-08-10','2'),
							('2','Carol','2024-08-12','4'),
							('3','Samanta','2024-08-11','3'),
							('4','Abner','2024-08-13','5'),
							('5','Rabi','2024-08-13','3'),
							('6','Silveira','2024-08-13','5'),
							('7','Paula','2024-08-11','4'),
							('8','Ana','2024-08-15','4'),
							('9','Leonidas','2024-08-15','3'),
							('10','Queila','2024-08-13','5'),
							('11','Rafael','2024-08-13','5'),
							('12','Patricia','2024-08-11','4'),
							('13','Ana Beatriz','2024-08-15','4'),
							('14','Henrique','2024-08-15','3'),
							('15','Damião','2024-08-13','5');



create table courses(
	c_id int,
	name text,
	price numeric,
	school_id int
	);

insert into courses values ('1','HTML','500','1'),
							('2','Python','800','1'),
							('3','Data Science','1000','2'),
							('4','Data Anallyst','900','2'),
							('5','Data Engineer','1210','2');

create table schools(
	id int,
	name text
	);

insert into schools values ('1','DB'),
						   ('2','LIA');







-- Consulta para retornar por dia e por escola: quantidade de alunos matriculados, valor total de matriculas, com uma restrição cursos que começam com palavra "Data", ordenando pleo dia mais rescente para o mais antigo.

select COUNT(s.name) as total_enrollment, SUM(c.price) as total_price, s.enrolled_at, c.name as course_name , s2.name as school_name 
	from students s 
	inner JOIN courses c 
	on s.course_id=c.c_id 
	inner JOIN schools s2 
	on c.school_id=s2.id
	where c.name like 'Data%'
	group by s.enrolled_at, c.name, s2.name
	order by s.enrolled_at desc;


-- Consulta para obter por escola e por dia, a soma acumulada:

with tbl_tmp_enrolleds as
	(select COUNT(s.name) as total_enrollment, SUM(c.price) as total_price, s.enrolled_at,  s2.name as school_name 
	from students s 
	inner JOIN courses c on s.course_id=c.c_id
	inner JOIN schools s2 on c.school_id=s2.id
	where c.name like 'Data%'
	group by s.enrolled_at, s2.name
	order by s.enrolled_at desc
)

select total_price, enrolled_at, school_name,
	SUM (total_price)
	over (partition by school_name order by enrolled_at
	rows between unbounded preceding and current ROW) as accumulated_sum
	from tbl_tmp_enrolleds
	order by enrolled_at;



-- Consulta para obter por escola e por dia, a média móvel de 7 dias:

with tbl_tmp_enrolleds as
	(select COUNT(s.name) as total_enrollment, SUM(c.price) as total_price, s.enrolled_at,  s2.name as school_name 
	from students s 
	inner JOIN courses c on s.course_id=c.c_id
	inner JOIN schools s2 on c.school_id=s2.id
	where c.name like 'Data%'
	group by s.enrolled_at, s2.name
	order by s.enrolled_at desc
)

select total_price, enrolled_at, school_name,
	AVG (total_price)
	over (partition by school_name order by enrolled_at
	rows between 6 preceding and current ROW) as moving_average
	from tbl_tmp_enrolleds
	order by enrolled_at;


-- Consulta para obter por escola e por dia, a média móvel de 30 dias:

with tbl_tmp_enrolleds as
	(select COUNT(s.name) as total_enrollment, SUM(c.price) as total_price, s.enrolled_at,  s2.name as school_name 
	from students s 
	inner JOIN courses c on s.course_id=c.c_id
	inner JOIN schools s2 on c.school_id=s2.id
	where c.name like 'Data%'
	group by s.enrolled_at, s2.name
	order by s.enrolled_at desc
)

select total_price, enrolled_at, school_name,
	AVG (total_price)
	over (partition by school_name order by enrolled_at
	rows between 29 preceding and current ROW) as moving_average
	from tbl_tmp_enrolleds
	order by enrolled_at;

