# Introducción
Damos un salto hacia el mercado de trabajo en data! Concentrandonos en roles de Analista de Datos, este proyecto explora los trabajos mejor pagos, las habilidades en demanda y el lugar de encuentro donde salarios altos coinciden con alta demanda, en relación a Analisis de Datos.

Consultas SQL? Exploralas aqui: [project_sql folder](/proyect_sql/)
# Bases
Los fundamentos del proyecto buscan el navegar el mercado de trabajos de un Analista de Datos mas eficientemente, nacen de un deseo de identificar salarios bien remunerados y las habilidades que buscan los mismos. Asi agilizando la busqueda de oportunidades optimas!

Los datos surgen de [Curso de SQL](https://lukebarousse.com/sql) de Luke Barousse. Conglomera miles de entradas de datos, incluyendo titulos de tranajos, slarios, ubicaciones y habilidades esenciales.

### Las preguntas que este analisis busca explorar:

1. ¿Cuales son los trabajos mejor remunerados de Analista de Datos?
2. ¿Que herramientas requieren dichos trabajos?
3. Dentro de esas herramientas, ¿Cuales son las mas buscadas?
4. ¿Que herramientas estan asociadas con mayor remuneración?
5. ¿Cuales son las herramientas mas improtantes en las cuales enfocarse?
# Herramientas Utilizadas
Para la zambullida en el mercado de trabajo de Analista de Datos hay varias herramientas claves que me ayudaron a navegarlo:

- **SQL:** La base del analisis, lo que me permite consultar la base de datos para encontrar esas conclusiones clave.
- **PostgreSQL:** El sistema de manejo de datos elegido, ideal para contener las miles de entradas.
- **Visual Studio Code:** Estupenda herramienta para interactuar con la base de datos ejecutando y visualizando las consultas SQL.
- **Git y GitHub:** Esencial para iterar versiones del analisis, junto con compartir el proceso asegurando colaboracion y un seguimiento del proyecto.
# El Analisis
Cada consulta SQL apunta a investigar distintos aspectos del mercado de trabajo de Analista de Datos.

Asi aborde cada pregunta:

### 1. Trabajos de Analista de Datos mejor remunerados.
Para identificar los trabajos mejor remunerados filtre las posiciones de Analista de datos por el promedio de salario anual y la ubicación, en este caso, haciendo enfasis en trabajos remotos.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    company_dim.name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    salary_year_avg IS NOT NULL AND
    job_work_from_home = TRUE AND
    job_title_short = 'Data Analyst'
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```
#### ¿Que podemos extraer?
- **Rangos Variados:** El top 10 de las remuneraciones varian desde $184,000 hasta $650,000, indicando un significativo potencial de salario en el campo. (cabe remarcar que el valor de $650,000 es el unico de su indole en los datos, lo que podria clasificarlo como un caso atípico).

- **Diversos Origenes:** Compañias como SmartAsset, Meta y AT&T aparecen entre aquellos que ofrecen esa alta remuneracion, indicando un amplio interes entre distintas industrias.

- **Variacion de titulos y responsabilidades:** Tambien podemos encontrar gran variedad en los titulos de los trabajos, reflejando roles expansivos y oportunidades de especializacion dentro del ambito.

![Salarios Mejor Remunerados](assets\Salarios_mejor_remunerados.png)
*Grafico de barras visualizando el salario para el top 10 salarios de Analista de Datos; grafico generado a traves de DeepSeek usando los resultados de la consulta SQL*
### 2. Herramientas y habilidades especializadas.
Para generar un mayor entendimiento de lo que uno debe manejar para llegar a estas posiciones, usando una SubQuery de la consulta anterior, unimos las tablas de datos de trabajos y la de habilidades para analizar que herramientas son mas valoradas en dichas posiciones.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        company_dim.name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        salary_year_avg IS NOT NULL AND
        job_work_from_home = TRUE AND
        job_title_short = 'Data Analyst'
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
    )

SELECT
    top_paying_jobs.*,
    skills
FROM
    top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC
```
#### ¿Que podemos extraer?
- **SQL** lidera la demanda apareciendo en 8 de los 10 trabajos en analisis.
- **Python** es un cercano segundo apareciendo en 7.
- **Tableau** tambien surge en gran demanda como herramienta para visualizacion de los datos. Otras habilidades como R, Snowflake, Pandas y Excel muestran distintos grados de demanda.

![Herramientas en demanda](assets\top_habilidades_demandadas.png)
*Grafico de barras visualizando la demanda de las habilidades en el top 10; grafico generado a traves de DeepSeek usando la segunda consulta SQL*
### 3. Habilidades con mayor demanda.
Esta consulta ayuda a identificar las habilidades con mayor demanda teniendo en cuenta la totalidad de los datos.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM
    job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND job_location = 'Anywhere'
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```
#### ¿Que podemos extraer?
- SQL y Excel son fundamentales, demostrando la necesidad de una fuerte base en habilidades relacionadas al procesamiento de datos y manipulacion de hojas de calculo.
- Herramientas de programacion y visualizacion como Python, Tableau y Power BI son esenciales, apuntando hacia la importancia de habilidades tecnicas a la hora de analizar datos.

|Habilidades | Demanda Cont.|
|------------|--------------|
|  SQL       |  7291        |
|  Excel     |  4611        |
|  Python    |  4330        |
|  Tableau   |  3745        |
|  Power BI  |  2609        |

*Tabla del top 5 de las habilidades con matoy demanda*
### 4. Habilidades basadas en Salario.
Explorar los salarios promedio referenciandolos a las distintas habilidades reveló que habilidades son las mejor pagas.

```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) as avg_salary
FROM
    job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    salary_year_avg is NOT NULL AND job_location = 'Anywhere'
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```
#### ¿Que podemos extraer?
- **Big data y procesamiento distribuido dominan el top:** PySpark lidera con $208,172, confirmando que el manejo de datos a gran escala es la habilidad más valorada. Le siguen Couchbase ($160,515) y Databricks ($141,907), completando un ecosistema de tecnologías para big data que consistentemente pagan por encima del promedio.
- **DevOps y colaboracion:** Bitbucket ($189,155) y GitLab ($154,500) aparecen en posiciones sorprendentemente altas, junto con Jenkins ($125,436). Esto revela una tendencia clave: las empresas buscan analistas que se integren con flujos de desarrollo y automatización.
- **El ecosistema Python mantiene su poder:** Jupyter ($152,777), Pandas ($151,821) y NumPy ($143,513) demuestran que el stack tradicional de Python para datos sigue siendo muy bien remunerado. Sin embargo, el verdadero valor está en las herramientas específicas del ecosistema, no en Python "básico".

|Habilidades  | Salario Prom.|
|-------------|--------------|
|  pyspark    |  208172      |
|  bitbucket  |  189155      |
|  couchbase  |  160515      |
|  watson     |  160515      |
|  datarobot  |  155486      |
|  gitlab     |  154500      |
|  swift      |  153750      |
|  jupyter    |  152777      |
|  pandas     |  151821      |
|elasticsearch|  145000      |

*Tabla del Top 10 habilidades mejor remuneradas.*
### 5. Donde ejercer mayor enfoque.
Combinando datos de los salarios y la demanda, esta consulta busca encontrar las habilidades en la interseccion entre ambos y ofrecer un enfoque estrategico de desarrollo de las habilidades.
```sql
SELECT
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM
    job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
        AND salary_year_avg is NOT NULL
        AND job_work_from_home = TRUE
GROUP BY
    skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```
| # | Habilidad | Salario Promedio | Demanda (empleos) |
|---|-----------|-----------------:|------------------:|
| 1 | Go | $115,320 | 27 |
| 2 | Confluence | $114,210 | 11 |
| 3 | Hadoop | $113,193 | 22 |
| 4 | Snowflake | $112,948 | 37 |
| 5 | Azure | $111,225 | 34 |
| 6 | BigQuery | $109,654 | 13 |
| 7 | AWS | $108,317 | 32 |
| 8 | Java | $106,906 | 17 |
| 9 | SSIS | $106,683 | 12 |
| 10 | Jira | $104,918 | 20 |

*Tabla de habilidades optimas ordenadas por salario*
| # | Habilidad | Demanda (empleos) | Salario Promedio |
|---|-----------|------------------:|-----------------:|
| 1 | Python | 236 | $101,397 |
| 2 | Tableau | 230 | $99,288 |
| 3 | R | 148 | $100,499 |
| 4 | SAS | 63 | $98,902 |
| 5 | Looker | 49 | $103,795 |
| 6 | Snowflake | 37 | $112,948 |
| 7 | Oracle | 37 | $104,534 |
| 8 | Azure | 34 | $111,225 |
| 9 | AWS | 32 | $108,317 |
| 10 | SQL Server | 35 | $97,786 |

*Tabla de habilidades optimas ordenadas por demanda*
#### ¿Que podemos extraer?
- **Lenguajes de programacion en alta demanda:** Python y R resaltan por su gran demanda con contabilizaciones de 236 y 148 respectivamente. A pesar de eso, sus salarios rondan en $101,397 para Python y $100,499 para R, lo que indica que proficiencia en estos leguajes es muy valorado pero tambien hay gran disponibilidad.
- **Herramientas y tecnologias en la nube:** habilidades en tecpnologias especializadas como Snowflake, Azure, AWS y BigQuery muestran significativa demanda con slarios relativamente altos, apuntando a la importancia creciente de las plataformas de la nube en Analisis de Datos.
- **Herramientas de Visualizacion e Inteligencia Empresarial:** Tableau y Looker, con contabilizaciones de demanda de 230 y 49 respectivamente, y salarios promedio de $99,288 y $103,795, remarcan el rol vital de poder derivar conclusiones accionables de los datos.
- **Tecnologias de Bases de datos:** la demanda por habilidades en tradicionales y bases de datos NoSQL (Oracle, SQL Server, NoSQL) con promedio de salarios desde $99,786 a $104,534, refleja la fundamental necesidad de experiencia para almacenar, consultar y controlar datos. 
# Lo que aprendí
A traves de esta experiencia, pude expandir tanto mi entendimiento como experiencia de los sistemas que fundan el ambiente de SQL:

- **Avance el creado de  una Query:** ya sea utilizando JOIN para expandir la pileta de datos o usando WITH para acelerar la iteracion apoyandome en tablas temporales.
- **Agregacion de Datos:** me puse mas comodo a la hora de utilizar GROUP BY para sacar mas potencial de funciones de agregacion como COUNT() o AVG.
-  **Resolucion de problemas:** puse en practica mis habilidades de resolucion de problemas con un enfoque en soluciones reales para convertir preguntas en consultas accionables de SQL.
# Conclusiones
### Perspectivas
Basandonos en el analisis, surgen varias perspectivas:
1. **Trabajos de Analista de Datos mejor Pagos:** los trabajos mejor remunerados para el rol ofrecen un gran rango de salarios, $650,000 siendo el mas alto.
2. **Herramientas y habilidades especializadas:** trabajos con mayor remuneracion requieren avanzada proficiencia en SQL, indicando que es una habilidad fundamental para asegurar esas oportunidades.
3. **Habilidades con mayor demanda:** SQL es tambien la habilidad con mayor demanda en el mercado, convirtiendola en la base para las demas.
4. **Habilidades basadas en Salario:** habilidades especificas, como SVN y Solidity, son asociadas a los salarios mas altos, indicando un plus en nichos especializados.
5. **Donde ejercer mayor enfoque:** SQL lidera en demanda y ofrece un alto salario promedio, convirtiendola en la habilidad mejor valorada para maximizar tu valor en este ambito.
### Closing Thoughts
Este proyecto mejoro mis hablidades con SQL y me dejo con valiosas perspectivas dentro del mundo de Analisis de Datos y su mercado laboral.
Los resultados apuntan a servir como una guia a la hora de priorizar el tiempo y enfoque cuando uno aspira a entrar a este ecosistema. Y lograr un primer paso, confiado tanto como informado, dentro de un mundo de cambiantes tendencias, que continuamente evoluciona.