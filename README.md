## 🧠 Entendimento do Projeto

### 🧩 1. Relacionamento entre `FORNECEDOR` e `ENQUADRAMENTO_FORNECEDOR`

A tabela `FORNECEDOR` representa a entidade central que contém todos os dados cadastrais dos fornecedores participantes da política de antecipação. Cada fornecedor é identificado unicamente pelo campo técnico `id_fornecedor`, utilizado como **chave primária**. Embora o `radical_cnpj` seja uma forma de identificação empresarial, a adoção de uma chave técnica assegura a integridade dos dados, evitando problemas com formatações, duplicidades ou alterações legais no CNPJ.

A tabela `ENQUADRAMENTO_FORNECEDOR` atua como uma **tabela de fato** que registra a classificação do fornecedor dentro de uma **faixa de performance** em momentos distintos no tempo. O campo `id_fornecedor` aqui é uma **chave estrangeira** que referencia a tabela `FORNECEDOR`.

- **Relacionamento:** `1:N` → Um fornecedor pode ter múltiplos registros de enquadramento ao longo do tempo.
- **Benefícios:** Permite consultar o histórico completo de enquadramentos, acompanhar alterações de performance e taxas, conforme a política vigente.

---

### 🧩 2. Relacionamento entre `FAIXA_PERFORMANCE` e `ENQUADRAMENTO_FORNECEDOR`

A tabela `FAIXA_PERFORMANCE` define os critérios de classificação dos fornecedores com base no `share_percentual`. Cada faixa contém:

- `taxa_min_cdi_percent`
- `share_min_percent`
- `share_max_percent`

O campo `id_faixa` é a **chave primária** e é referenciado na tabela `ENQUADRAMENTO_FORNECEDOR` como **foreign key**, configurando outro relacionamento do tipo `1:N`.

- **Relacionamento:** `1:N` → Uma faixa pode ser associada a múltiplos enquadramentos.
- **Importância:** Permite padronizar e parametrizar a política de risco, além de preservar o histórico de faixas mesmo em caso de alteração das regras.

---

### 🧩 3. Relacionamento entre `COMITE_TAXAS` e `ENQUADRAMENTO_FORNECEDOR`

A tabela `COMITE_TAXAS` registra eventos de reuniões com definições econômicas, como:

- `meta_selic_aa` (Selic anual)
- `meta_selic_am` (Selic mensal)
- `vigencia_inicio` e `vigencia_fim` (vigência das decisões)

O campo `id_reuniao` é chave primária e também aparece em `ENQUADRAMENTO_FORNECEDOR` como chave estrangeira.

- **Relacionamento:** `1:N` → Uma reunião pode impactar diversos enquadramentos de fornecedores.
- **Função:** Vincula o enquadramento não só à performance do fornecedor, mas também ao contexto econômico do período, permitindo o cálculo da `taxa_aplicada_mes`.

---

### 🔍 Considerações Técnicas

A modelagem privilegia a **normalização dos dados**, resultando em:

- Menor redundância;
- Maior integridade referencial;
- Auditabilidade de decisões;
- Flexibilidade para alterações futuras;
- Suporte a análises históricas e preditivas.

A tabela intermediária `ENQUADRAMENTO_FORNECEDOR` viabiliza a implementação de lógicas do tipo **SCD Tipo 2 (Slowly Changing Dimensions)**, comum em arquiteturas analíticas que demandam rastreabilidade e histórico.

---

### 📊 Exemplos de Análises Possíveis

- Qual foi a **taxa média aplicada** aos fornecedores da faixa _"MÉDIA-ALTA"_ no último semestre?
- Como o **share** dos fornecedores evoluiu ao longo do tempo?
- Que impacto uma mudança na meta Selic teve sobre a taxa final aplicada?
  
---
---

## 🛠️ Scripts de Criação das Tabelas

Abaixo estão os scripts SQL utilizados para criar as tabelas do modelo relacional descrito acima.

### 📄 Tabela `FORNECEDOR`

```CREATE TABLE FORNECEDOR (
    id_fornecedor INT PRIMARY KEY,
    radical_cnpj VARCHAR(255),
    razao_social VARCHAR(255),
    cod_depto VARCHAR(255),
    nome_depto VARCHAR(255),
    share_percentual DECIMAL(10, 4),
    data_base_share DATE
);

CREATE TABLE FAIXA_PERFORMANCE (
    id_faixa INT PRIMARY KEY,
    nome_faixa VARCHAR(255),
    taxa_min_cdi_percent DECIMAL(10, 4),
    share_min_percent DECIMAL(10, 4),
    share_max_percent DECIMAL(10, 4)
);

CREATE TABLE COMITE_TAXAS (
    id_reuniao INT PRIMARY KEY,
    numero_reuniao VARCHAR(255),
    data_reuniao DATE,
    vigencia_inicio DATE,
    vigencia_fim DATE,
    meta_selic_aa DECIMAL(10, 4),
    meta_selic_am DECIMAL(10, 4)
);

CREATE TABLE ENQUADRAMENTO_FORNECEDOR (
    id_enquadramento INT PRIMARY KEY,
    id_fornecedor INT,
    id_faixa INT,
    id_reuniao INT,
    data_enquadramento DATE,
    taxa_aplicada_mes DECIMAL(10, 4),
    FOREIGN KEY (id_fornecedor) REFERENCES FORNECEDOR(id_fornecedor),
    FOREIGN KEY (id_faixa) REFERENCES FAIXA_PERFORMANCE(id_faixa),
    FOREIGN KEY (id_reuniao) REFERENCES COMITE_TAXAS(id_reuniao)
);
