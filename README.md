# data-join-sql-and-linear-algebra

## 1. Dados de Exemplo

### Tabela Professores

| ID | Nome           | Departamento |
|----|----------------|--------------|
| 1  | Prof. Silva    | Matemática   |
| 2  | Prof. Souza    | Física       |
| 3  | Prof. Pereira  | Química      |

### Tabela Cursos

| ID | Nome            | Mensalidade | ProfessorID |
|----|-----------------|-------------|-------------|
| 1  | Cálculo I       | 500.00      | 1           |
| 2  | Física I        | 550.00      | 2           |
| 3  | Álgebra Linear  | 600.00      | 1           |
| 4  | Física II       | 600.00      | 2           |

> **Observação:** O professor “Prof. Pereira” (ID=3) não ministra nenhum curso. Além disso, o curso “Física II” (ID=4) não possui matrículas.

### Tabela Alunos

| ID | Nome   | Idade |
|----|--------|-------|
| 1  | Alice  | 20    |
| 2  | Bruno  | 21    |
| 3  | Carla  | 22    |
| 4  | Daniel | 23    |

> **Observação:** O aluno “Daniel” (ID=4) não está matriculado em nenhum curso.

### Tabela Matrículas

| CursoID | AlunoID | DataMatricula |
|---------|---------|---------------|
| 1       | 1       | 2025-01-10    |
| 1       | 2       | 2025-01-12    |
| 2       | 3       | 2025-01-15    |
| 3       | 2       | 2025-01-16    |

> **Observação:** Não há matrícula para o curso “Física II” (ID=4).

---

## 2. Consulta SQL Utilizada

Utilizando os dados acima, a consulta que une as tabelas com três tipos de JOIN (LEFT, FULL OUTER e RIGHT) foi definida assim:

```sql
SELECT 
    P.Nome AS 'Nome do Professor',
    C.Nome AS 'Nome do Curso',
    A.Nome AS 'Nome do Aluno'
FROM Professores P
LEFT JOIN Cursos C ON P.ID = C.ProfessorID
FULL OUTER JOIN Matrículas M ON C.ID = M.CursoID
RIGHT JOIN Alunos A ON M.AlunoID = A.ID
WHERE P.ID IS NOT NULL;
```

**Breve explicação da lógica da consulta:**

1. **Professores LEFT JOIN Cursos:**  
   - Retorna todos os professores, mesmo aqueles que não ministram cursos (nesse caso, os campos referentes ao curso ficam com valor _NULL_).

2. **FULL OUTER JOIN com Matrículas:**  
   - Garante que sejam incluídas as linhas de cursos mesmo que não possuam matrículas, bem como eventuais registros de matrículas “órfãos” (a título ilustrativo).

3. **RIGHT JOIN com Alunos:**  
   - Assegura que todos os alunos que estão vinculados a uma matrícula apareçam na consulta. Se um aluno não tiver matrícula associada, essa linha ficaria com os dados das tabelas à esquerda como _NULL_ e seria, em seguida, filtrada pela condição.

4. **WHERE P.ID IS NOT NULL:**  
   - Essa condição garante que, no resultado final, apareçam apenas os registros com dados do professor (a “primeira entidade”).

---

## 3. Simulação da Junção das Tabelas e Resultado

### Passo a Passo do JOIN

1. **Professores com Cursos (LEFT JOIN):**

   - **Prof. Silva (ID=1):**  
     - Cursos: *Cálculo I* (ID=1) e *Álgebra Linear* (ID=3)

   - **Prof. Souza (ID=2):**  
     - Cursos: *Física I* (ID=2) e *Física II* (ID=4)

   - **Prof. Pereira (ID=3):**  
     - Não ministra nenhum curso – resulta em linha com os campos de curso nulos.

2. **Inclusão de Matrículas (FULL OUTER JOIN):**

   - Para *Cálculo I* (ID=1):  
     - Matrículas encontradas:  
       - (CursoID=1, AlunoID=1, 2025-01-10)  
       - (CursoID=1, AlunoID=2, 2025-01-12)

   - Para *Álgebra Linear* (ID=3):  
     - Matrícula encontrada:  
       - (CursoID=3, AlunoID=2, 2025-01-16)

   - Para *Física I* (ID=2):  
     - Matrícula encontrada:  
       - (CursoID=2, AlunoID=3, 2025-01-15)

   - Para *Física II* (ID=4):  
     - Nenhuma matrícula – os campos da matrícula ficam _NULL_.

   - Para o professor sem curso (Prof. Pereira):  
     - Não há informação de matrícula, logo os campos permanecem _NULL_.

3. **Inclusão de Alunos (RIGHT JOIN):**

   - A partir da tabela de Matrículas, associamos os alunos:
     - **Aluno 1 (Alice):** Associado a *Cálculo I* (registro com AlunoID=1).
     - **Aluno 2 (Bruno):** Aparece em dois registros: um vinculado a *Cálculo I* e outro a *Álgebra Linear*.
     - **Aluno 3 (Carla):** Associado a *Física I*.
     - **Aluno 4 (Daniel):** Não possui matrícula – a linha resultante terá os dados à esquerda (_Professor_ e _Curso_) como _NULL_.

4. **Aplicando o filtro WHERE P.ID IS NOT NULL:**

   - Elimina o registro onde não há dados de professor (por exemplo, a linha com aluno Daniel).

### Resultado Final da Consulta

Após todas as junções e a aplicação do filtro, o resultado exibido será:

| Nome do Professor | Nome do Curso   | Nome do Aluno |
|-------------------|-----------------|---------------|
| Prof. Silva       | Cálculo I       | Alice         |
| Prof. Silva       | Cálculo I       | Bruno         |
| Prof. Silva       | Álgebra Linear  | Bruno         |
| Prof. Souza       | Física I        | Carla         |

> **Notas sobre o Resultado:**  
> - O professor “Prof. Pereira” não aparece, pois não há curso (e consequentemente nenhuma matrícula) associada a ele.  
> - O aluno “Daniel” também é excluído por não estar vinculado a nenhuma matrícula de um curso ministrado por um professor.

---

## 4. Equação em Álgebra Relacional (Relembrando)

A mesma consulta pode ser representada em álgebra relacional da seguinte forma:

$$ 
\pi_{P.\text{Nome},\; C.\text{Nome},\; A.\text{Nome}} \Bigl(
( \text{Professores} \; ⟕_{P.ID = C.ProfessorID} \; \text{Cursos} )
\; ⟗_{C.ID = M.CursoID} \; \text{Matrículas}
\; ⟖_{M.AlunoID = A.ID} \; \text{Alunos}
\Bigr)
$$


Onde:
- \( ⟕ \) representa o **LEFT OUTER JOIN**;
- \( ⟗ \) representa o **FULL OUTER JOIN**;
- \( ⟖ \) representa o **RIGHT OUTER JOIN**;
- \( \pi \) é o operador de projeção, selecionando as colunas desejadas.

---

Com esses dados e o detalhamento do processo de junção, temos um exemplo completo que ilustra como o modelo de dados se comporta e qual o resultado da consulta SQL proposta.