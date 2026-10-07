# Agendador de Horários API

API REST para agendamento de horários de serviços (como corte de cabelo e barba), desenvolvida com **Java 21** e **Spring Boot**.

Com ela é possível criar, consultar, alterar e cancelar agendamentos. A API impede que o mesmo serviço seja marcado duas vezes no mesmo intervalo de horário.

## Funcionalidades

- **Criar agendamento**: recusa o pedido se já existir outro agendamento do mesmo serviço entre o horário pedido e 1 hora depois.
- **Listar agendamentos de um dia**: retorna todos os agendamentos de uma data.
- **Alterar agendamento**: localiza o agendamento pelo cliente e pela data/hora e atualiza seus dados.
- **Cancelar agendamento**: remove o agendamento pelo cliente e pela data/hora.
- **Data de inserção automática**: cada agendamento grava quando foi criado (`dataInsercao`), e essa data é mantida nas alterações.

## Tecnologias

- Java 21
- Spring Boot 4
- Spring Web MVC
- Spring Data JPA / Hibernate
- H2 Database (banco em memória)
- Lombok
- Maven

## Arquitetura

O projeto segue a separação em camadas:

```
src/main/java/com/javanauta/agendador_horarios
├── controller
│   └── AgendamentoController.java     # Endpoints REST
├── services
│   └── AgendamentoService.java        # Regras de negócio
└── infrastructure
    ├── entity
    │   └── Agendamento.java           # Entidade JPA
    └── repository
        └── AgendamentoRepository.java # Acesso ao banco (Spring Data JPA)
```

**Fluxo de uma requisição:** `Controller` → `Service` → `Repository` → banco H2

## Como executar

**Pré-requisito:** Java 21 instalado. O Maven não precisa ser instalado, porque o projeto usa o Maven Wrapper.

```bash
# Clonar o repositório
git clone https://github.com/danielperes1/agendador-horarios.git
cd agendador-horarios

# Linux / macOS
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run
```

A API sobe em `http://localhost:8080`.

> O banco H2 é em memória: os dados são apagados toda vez que a aplicação é reiniciada.

## Endpoints

Base URL: `http://localhost:8080/agendamentos`

| Método | Endpoint | Parâmetros (query) | Body | Resposta |
|---|---|---|---|---|
| `POST` | `/agendamentos` | — | Agendamento | `202 Accepted` |
| `GET` | `/agendamentos` | `data` | — | `200 OK` |
| `PUT` | `/agendamentos` | `cliente`, `dataHoraAgendamento` | Agendamento com os novos dados | `202 Accepted` |
| `DELETE` | `/agendamentos` | `cliente`, `dataHoraAgendamento` | — | `204 No Content` |

Formato das datas: `data` usa `AAAA-MM-DD` (ex.: `2026-10-08`) e `dataHoraAgendamento` usa `AAAA-MM-DDTHH:MM:SS` (ex.: `2026-10-08T14:00:00`).

### Criar agendamento

`POST /agendamentos`

```json
{
  "servico": "Corte de cabelo",
  "profissional": "Carlos",
  "dataHoraAgendamento": "2026-10-08T14:00:00",
  "cliente": "Joao",
  "telefoneCliente": "27999999999"
}
```

Resposta `202 Accepted`:

```json
{
  "id": 1,
  "servico": "Corte de cabelo",
  "profissional": "Carlos",
  "dataHoraAgendamento": "2026-10-08T14:00:00",
  "cliente": "Joao",
  "telefoneCliente": "27999999999",
  "dataInsercao": "2026-10-07T03:02:01.284303"
}
```

### Listar agendamentos do dia

`GET /agendamentos?data=2026-10-08`

Retorna `200 OK` com a lista de agendamentos da data, ou `[]` se não houver nenhum.

### Alterar agendamento

`PUT /agendamentos?cliente=Joao&dataHoraAgendamento=2026-10-08T14:00:00`

Os parâmetros da URL identificam **qual** agendamento alterar. O body traz os **novos dados**:

```json
{
  "servico": "Barba",
  "profissional": "Carlos",
  "dataHoraAgendamento": "2026-10-08T15:00:00",
  "cliente": "Joao",
  "telefoneCliente": "27999999999"
}
```

### Cancelar agendamento

`DELETE /agendamentos?cliente=Joao&dataHoraAgendamento=2026-10-08T15:00:00`

Retorna `204 No Content`.

## Testando com o Insomnia

A pasta [`insomnia/`](insomnia/) contém a collection com as requisições prontas para todos os endpoints.

1. No Insomnia, use **Import** e selecione o arquivo da pasta `insomnia/`.
2. Suba a aplicação e execute as requisições na ordem: criar → listar → alterar → cancelar.

## Banco de dados H2

Com a aplicação rodando, o console do banco fica disponível em `http://localhost:8080/h2-console`.

| Campo | Valor |
|---|---|
| JDBC URL | `jdbc:h2:mem:agendamentos-db` |
| User Name | `sa` |
| Password | *(vazio)* |

## Próximos passos

- [ ] Retornar status HTTP específicos nos erros (`409 Conflict` para horário ocupado e `404 Not Found` para agendamento inexistente), com a mensagem no corpo da resposta
- [ ] Validar os dados de entrada com Bean Validation
- [ ] Criar testes unitários e de integração
- [ ] Documentar a API com Swagger / OpenAPI
- [ ] Usar um banco de dados persistente, como o PostgreSQL
