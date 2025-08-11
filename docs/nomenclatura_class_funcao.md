## 🧠 Padrões de Nomenclatura – Classes e Funções

### 🏛️ Classes

- **Estilo:** PascalCase (também conhecido como UpperCamelCase)
- **Regra:** Nome deve ser um substantivo ou representar uma entidade
- **Exemplos:**
  - `UserService`
  - `EmailClient`
  - `PermissionRepository`
  - `CapacityMonitorTasks`
  - `AuthSchema`

> 🔎 Dica: Classes que herdam de frameworks (como `BaseModel`, `APIView`, etc.) devem manter o padrão do framework.

---

### ⚙️ Funções e Métodos

- **Estilo:** snake_case
- **Regra:** Nome deve ser um verbo ou frase verbal que descreve claramente a ação
- **Exemplos:**
  - `send_email`
  - `get_user_permissions`
  - `validate_token`
  - `schedule_monitoring_job`
  - `convert_date_to_iso`

> 🧼 Dica: Evite nomes genéricos como `do_task` ou `handle_data`. Prefira nomes específicos e descritivos.

---

### 📌 Convenções Adicionais

| Tipo de Função/Método        | Prefixo sugerido     | Exemplo                     |
|------------------------------|----------------------|-----------------------------|
| Métodos privados             | `_` (underscore)     | `_connect_to_smtp`          |
| Métodos assíncronos          | `async def`          | `async def fetch_data()`    |
| Funções de validação         | `validate_`          | `validate_email_format`     |
| Funções de transformação     | `convert_`, `map_`   | `convert_to_json`, `map_roles_to_ids` |
| Funções de acesso            | `get_`, `fetch_`     | `get_user_by_id`, `fetch_logs` |

---

### 🧪 Testes

- **Funções de teste:** `test_` + nome da funcionalidade
- **Exemplo:** `test_send_email_success`, `test_validate_token_expired`

---

### 📚 Sugestão de Leitura

- [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/)
