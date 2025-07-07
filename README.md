# Documentação da API (back-end)

Este documento descreve como configurar e testar a API do sistema escolar.

---

## Requisitos

- **Node.js:** versão 22.16.0 ou superior  
- **PostgreSQL:** psql 14.18 ou superior

---

## Configuração

Antes de rodar o projeto, siga os passos abaixo:

### 1. Instalar Dependências

Na pasta raiz do projeto, execute o comando no seu terminal:

```bash
npm install
```

### 2. Configurar Variáveis de Ambiente

Crie um arquivo chamado `.env` na raiz do projeto e adicione as seguintes variáveis:

```env
DB_USER=nome_do_seu_usuario_postgres
DB_HOST=localhost
DB_NAME=nome_do_seu_banco
DB_PASSWORD=senha_do_seu_banco
DB_PORT=5432
JWT_SECRET=48fsdjf3r9fjsd!@FDKJ34rjrf93rjfk3fjrk!@fdk
JWT_EXPIRATION=1h
PORT=3000
```

### 3. Criar as Tabelas no Banco

Com as variáveis configuradas, execute o seguinte comando no terminal para criar as tabelas no seu banco de dados:

```bash
node src/utils/run-migrations.js
```

---

## Endpoints da API

### `/api/auth`

- **POST /login:** Realiza o login de um usuário com email e senha.

- **POST /register:** Cadastra um novo usuário no sistema.

### `/api/users`

- **GET /**: Lista todos os usuários (protegido, somente admins).

- **GET /:id**: Pega um usuário específico (protegido).

- **PUT /:id**: Atualiza um usuário (protegido).

- **DELETE /:id**: Deleta um usuário (protegido, somente admins).

### `/api/classes`

- **GET /**: Retorna turmas (se admin, todas; se professor, apenas as vinculadas).

- **GET /:id**: Retorna uma turma específica pelo ID.

- **POST /**: Cria uma nova turma (protegido, somente admins).

- **PUT /:id**: Edita uma turma (protegido, somente admins).

- **DELETE /:id**: Deleta uma turma (protegido, somente admins).

### `/api/students`

- **GET /**: Retorna estudantes (se admin, todos; se professor, apenas os de suas turmas).

- **GET /:id**: Retorna um estudante específico pelo ID.

- **POST /**: Cria um novo estudante (protegido, somente admins).

- **PUT /:id**: Edita um estudante (protegido, somente admins).

- **DELETE /:id**: Deleta um estudante (protegido, somente admins).

---

## Guia de Testes no Postman (ou similar)

Siga os passos abaixo para testar a funcionalidade completa da API.

### 1. Cadastro e Login

#### 1.1. Registrar Usuário Administrador

- Método: POST  
- URL: `http://localhost:3000/api/auth/register`

Body (JSON):

```json
{
    "name": "admin",
    "username": "admin",
    "email": "admin@gmail.com",
    "password": "admin@admin",
    "role": "admin",
    "phone": "49999999999"
}
```

Retorno Esperado: Mensagem de sucesso e os dados do usuário criado.

#### 1.2. Registrar Usuário Professor

- Método: POST  
- URL: `http://localhost:3000/api/auth/register`

Body (JSON):

```json
{
    "name": "Professor Carlos",
    "username": "carlos.prof",
    "email": "carlos@escola.com",
    "password": "senhaforte123",
    "role": "professor",
    "phone": "49999990002"
}
```

#### 1.3. Efetuar Login

- Método: POST  
- URL: `http://localhost:3000/api/auth/login`

Body (JSON):

```json
{
    "email": "admin@gmail.com",
    "password": "admin@admin"
}
```

Ação: Copie o token JWT retornado. Ele será usado para autenticar as próximas requisições.

---

### 2. Rotas de Usuários

Use o token de admin para os testes abaixo.

- **Listar todos os usuários:**  
  - Método: GET  
  - URL: `http://localhost:3000/api/users`  
  - Retorno Esperado: Um objeto com a chave users contendo um array com os dois usuários criados.

- **Buscar usuário por ID:**  
  - Método: GET  
  - URL: `http://localhost:3000/api/users/1`  
  - Retorno Esperado: Os dados do usuário "admin".

- **Atualizar usuário:**  
  - Método: PUT  
  - URL: `http://localhost:3000/api/users/1`  

Body (JSON):

```json
{
    "name": "Admin Atualizado",
    "email": "admin@gmail.com",
    "role": "admin",
    "phone": "49111112222"
}
```

Retorno Esperado: Mensagem de sucesso e os dados atualizados.

- **Deletar usuário:**  
  - Método: DELETE  
  - URL: `http://localhost:3000/api/users/2` (Deletando o professor)  
  - Retorno Esperado: Mensagem de sucesso.

---

### 3. Rotas de Turmas

Use o token de admin para os testes abaixo.

#### 3.1. Criar Turmas

- Método: POST  
- URL: `http://localhost:3000/api/classes`

Body (JSON) - Turma 1:

```json
{
    "name": "Oficina de Robótica",
    "shift": "Vespertino",
    "time": "14:00 - 16:00",
    "number_of_vacancies": 20
}
```

Body (JSON) - Turma 2:

```json
{
    "name": "Oficina de Informática",
    "shift": "Matutino",
    "time": "08:00 - 10:00",
    "number_of_vacancies": 15
}
```

#### 3.2. Vincular Professor a uma Turma

Recrie o professor se você o deletou no passo anterior.

- Método: PUT  
- URL: `http://localhost:3000/api/users/3` (ID do Professor Carlos, utilize o ID 2 se você não o apagou nos passos anteriores)

Body (JSON):

```json
{
    "name": "Professor Carlos Silva",
    "email": "carlos@escola.com",
    "role": "professor",
    "phone": "49988887777",
    "classIds": [1]
}
```

#### 3.3. Verificar Turmas (Como Professor)

Use o token do professor para este teste.

- Método: GET  
- URL: `http://localhost:3000/api/classes`

Retorno Esperado: Um array contendo apenas a "Oficina de Robótica" (ID 1).

---

### 4. Rotas de Estudantes

Use o token de admin para os testes abaixo.

#### 4.1. Criar Estudante

- Método: POST  
- URL: `http://localhost:3000/api/students`

Body (JSON):

```json
{
    "name": "Mariana Alves",
    "birth_date": "2012-02-10",
    "age": 13,
    "institution": "Escola Modelo",
    "grade": "7º Ano",
    "nationality": "Brasileira",
    "hometown": "Chapecó",
    "state": "SC",
    "marital_status": "Solteira",
    "profession": "Estudante",
    "sex": "feminino",
    "responsible_name": "Juliana Alves",
    "responsible_contact": "49977776666",
    "cpf": "123.456.789-00",
    "rg": "1234567",
    "uf": "SC",
    "address": "Rua Teste, 123",
    "has_health_plan": false,
    "uses_medication": false,
    "has_allergy": false,
    "has_special_needs": false,
    "image_authorization": true,
    "classIds": [1, 2]
}
```

#### 4.2. Listar Estudantes (Como Professor)

Use o token do professor para este teste.

- Método: GET  
- URL: `http://localhost:3000/api/students`

Retorno Esperado: Um array contendo apenas os dados da estudante "Mariana Alves".
