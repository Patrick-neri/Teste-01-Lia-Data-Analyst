# 📊 Teste 01: Data Analyst - Patrick Neri

## 🖋️ Narrativa:
Dadas as 3 tabelas:

- students: (id int, name text, enrolled_at date, course_id text)
- courses: (id int, name text, price numeric, school_id text)
- schools: (id int, name text)

Considere que todos alunos se matricularam nos respectivos cursos e que price é o valor da matrícula, pago por cada aluno.

a. Escreva uma consulta PostgreSQL para obter, por nome da escola e por dia, a quantidade de alunos matriculados e o valor total das matrículas, 
tendo como restrição os cursos que começam com a palavra “data”. Ordene o resultado do dia mais recente para o mais antigo.

b.Utilizando a resposta do item a, escreva uma consulta para obter, por escola e por dia, a soma acumulada, a média móvel 7 dias
e a média móvel 30 dias da quantidade de alunos.


## 🗂️ Objetivo
Este projeto simula um cenário de análise de dados para avaliar habilidades em SQL. Foi criada uma estrutura de banco de dados e realizadas consultas para gerar insights como: quantidade de matrículas, valores acumulados, e médias móveis.
Observação: Dados simples foram inseridos para testar os códigos.

---

## 🛠️ Estrutura do Banco de Dados


```sql

🧑‍🎓 Tabela de Alunos (`students`)

create table students(
    id int,
    name text,
    enrolled_at date,
    course_id int
);

insert into students values
    (0, 'Roberta', '2024-08-12', 1),
    (1, 'Renato', '2024-08-10', 2),
    (2, 'Carol', '2024-08-12', 4),
    (3, 'Samanta', '2024-08-11', 3),
    (4, 'Abner', '2024-08-13', 5),
    (5, 'Rabi', '2024-08-13', 3),
    (6, 'Silveira', '2024-08-13', 5),
    (7, 'Paula', '2024-08-11', 4),
    (8, 'Ana', '2024-08-15', 4),
    (9, 'Leonidas', '2024-08-15', 3),
    (10, 'Queila', '2024-08-13', 5),
    (11, 'Rafael', '2024-08-13', 5),
    (12, 'Patricia', '2024-08-11', 4),
    (13, 'Ana Beatriz', '2024-08-15', 4),
    (14, 'Henrique', '2024-08-15', 3),
    (15, 'Damião', '2024-08-13', 5);

---

📚 Tabela de Cursos (courses)

create table courses(
    c_id int,
    name text,
    price numeric,
    school_id int
);

insert into courses values
    (1, 'HTML', 500, 1),
    (2, 'Python', 800, 1),
    (3, 'Data Science', 1000, 2),
    (4, 'Data Analyst', 900, 2),
    (5, 'Data Engineer', 1210, 2);

---

🏫 Tabela de Escolas (schools)

create table schools(
    id int,
    name text
);

insert into schools values
    (1, 'DB'),
    (2, 'LIA');

---

💡 Consultas Realizadas

📅 1. Quantidade de Matrículas e Total de Matrículas Por Dia e Escola
Retorna a quantidade de alunos matriculados, o valor total das matrículas por dia e por escola,
apenas para cursos que começam com a palavra "Data". Ordenado do mais recente para o mais antigo.

select 
    COUNT(s.name) as total_enrollment,
    SUM(c.price) as total_price,
    s.enrolled_at,
    c.name as course_name,
    s2.name as school_name
from students s
inner join courses c on s.course_id = c.c_id
inner join schools s2 on c.school_id = s2.id
where c.name like 'Data%'
group by s.enrolled_at, c.name, s2.name
order by s.enrolled_at desc;

---

📈 2. Soma Acumulada Por Escola e Por Dia
Descrição: Retorna a soma acumulada do valor das matrículas por escola e por dia.

with tbl_tmp_enrolleds as (
    select 
        COUNT(s.name) as total_enrollment,
        SUM(c.price) as total_price,
        s.enrolled_at,
        s2.name as school_name
    from students s
    inner join courses c on s.course_id = c.c_id
    inner join schools s2 on c.school_id = s2.id
    where c.name like 'Data%'
    group by s.enrolled_at, s2.name
    order by s.enrolled_at desc
)
select 
    total_price,
    enrolled_at,
    school_name,
    SUM(total_price) over (partition by school_name order by enrolled_at rows between unbounded preceding and current row) as accumulated_sum
from tbl_tmp_enrolleds
order by enrolled_at;

---

📊 3. Média Móvel de 7 Dias Por Escola e Por Dia
Descrição: Retorna a média móvel dos valores das matrículas em uma janela de 7 dias.

with tbl_tmp_enrolleds as (
    select 
        COUNT(s.name) as total_enrollment,
        SUM(c.price) as total_price,
        s.enrolled_at,
        s2.name as school_name
    from students s
    inner join courses c on s.course_id = c.c_id
    inner join schools s2 on c.school_id = s2.id
    where c.name like 'Data%'
    group by s.enrolled_at, s2.name
    order by s.enrolled_at desc
)
select 
    total_price,
    enrolled_at,
    school_name,
    AVG(total_price) over (partition by school_name order by enrolled_at rows between 6 preceding and current row) as moving_average
from tbl_tmp_enrolleds
order by enrolled_at;

---

📊 4. Média Móvel de 30 Dias Por Escola e Por Dia
Descrição: Retorna a média móvel dos valores das matrículas em uma janela de 30 dias.

with tbl_tmp_enrolleds as (
    select 
        COUNT(s.name) as total_enrollment,
        SUM(c.price) as total_price,
        s.enrolled_at,
        s2.name as school_name
    from students s
    inner join courses c on s.course_id = c.c_id
    inner join schools s2 on c.school_id = s2.id
    where c.name like 'Data%'
    group by s.enrolled_at, s2.name
    order by s.enrolled_at desc
)
select 
    total_price,
    enrolled_at,
    school_name,
    AVG(total_price) over (partition by school_name order by enrolled_at rows between 29 preceding and current row) as moving_average
from tbl_tmp_enrolleds
order by enrolled_at;
