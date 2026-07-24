# TD-MPC2 TinyHouse WebApp — Google Apps Script

Este projecto contém uma arquitectura completa em MVC (Model-View-Controller) para ser executada no Google Apps Script (`script.google.com`). 

Foi concebido para gerir simulações do modelo TD-MPC2 TinyHouse, centralizando todo o CRUD (Create, Read, Update, Delete) numa única Google Sheet, conforme solicitado.

Além do CRUD de treino, o webapp é o painel de **preferências de comportamento e percepção** do "cérebro" (TD-MPC2) desta tiny house: os 2 perfis de utilizador (`administrador` e `morador`) registam, em painéis sanfona (`preferences.html`), como o modelo deve interpretar alerta de segurança, modo de água, rodízio de irrigação e percepção de fauna — ver README.md §"Recalibrando com dados de campo".

## 📦 Estrutura do Projecto

A arquitectura foi dividida em múltiplos componentes (25 ficheiros `.gs` e 16 ficheiros `.html`) para garantir boas práticas, manutenibilidade e separação de responsabilidades.

### Lógica de Servidor (`.gs`)
- **Code.gs**: Ponto de entrada (`doGet`, `doPost`, `setupSpreadsheet`).
- **Router.gs**: Roteamento de requisições.
- **Config.gs**: Leitura da variável de ambiente (`SPREADSHEETS_ID`).
- **Constants.gs**: Papéis, nomes de abas e constantes de domínio (zonas/espécies/linhas de irrigação, espelhando `gerar_modelo_3d.py`).
- **Auth.gs** e **SessionManager.gs**: Autenticação em texto plano (2 perfis: `administrador`, `morador`) e controlo de sessões.
- ***CRUD.gs**: Módulos de persistência para cada entidade (Users, Config, Episodes, States, Actions, Checkpoints, **Preferences**, **ZoneAffinity**).
- **DashboardController.gs**: Agregação de estatísticas.
- **ErrorHandler.gs** e **Logging.gs**: Tratamento de excepções e registo de eventos.
- **Validation.gs**, **DataFormatter.gs**, **Utils.gs**, **ExportUtils.gs**, **EmailService.gs**: Utilitários.
- **Proxy.gs**: Ponte entre o cliente (`google.script.run`) e o servidor.

### Interface do Utilizador (`.html`)
- **_Styles.html**, **_Scripts.html**, **_Icons.html**, **_Navbar.html**, **_Sidebar.html**: Componentes partilhados (tema grafite escuro, mobile-first, ícones em contorno).
- **login.html**: Ecrã de autenticação.
- **dashboard.html**: Painel principal.
- **preferences.html**: Painéis sanfona de preferências (morador + administrador).
- **config.html**, **users.html**, **episodes.html**, **states.html**, **actions.html**, **checkpoints.html**, **logs.html**, **export.html**, **profile.html**: Ecrãs de gestão.

## 🚀 Guia de Implantação (Deployment)

Para colocar este sistema em produção no Google Apps Script, siga os passos abaixo:

### Passo 1: Preparar a Google Sheet
1. Aceda a [sheets.google.com](https://sheets.google.com) e crie uma nova folha de cálculo vazia.
2. Copie o **ID da folha de cálculo** do URL. (O ID é a parte alfanumérica longa entre `/d/` e `/edit`).

### Passo 2: Criar o Projecto no Apps Script
1. Aceda a [script.google.com](https://script.google.com) e clique em **Novo projecto**.
2. Dê um nome ao projecto (ex: "TD-MPC2 TinyHouse WebApp").
3. Pode usar a ferramenta de linha de comandos **clasp** (`npm install -g @google/clasp`) para fazer upload de todos os ficheiros desta pasta, ou criar os ficheiros manualmente no editor online copiando e colando o conteúdo.
   - *Dica*: Se usar o `clasp`, basta fazer `clasp login`, depois `clasp create --type webapp` e finalmente `clasp push`.

### Passo 3: Configurar a Variável de Ambiente
O sistema exige que o ID da folha de cálculo seja configurado como uma Propriedade do Script, para não ficar exposto no código.
1. No editor do Apps Script, clique no ícone da **Engrenagem (Project Settings / Definições do projecto)** na barra lateral esquerda.
2. Desça até à secção **Script Properties (Propriedades do script)**.
3. Clique em **Add script property (Adicionar propriedade de script)**.
4. Em *Property (Propriedade)* escreva exactamente: `SPREADSHEETS_ID`
5. Em *Value (Valor)* cole o ID da folha de cálculo que copiou no Passo 1.
6. Clique em **Save script properties (Guardar propriedades do script)**.

### Passo 4: Inicializar o Sistema
Antes de usar o webapp, é necessário criar as abas na folha de cálculo.
1. Volte ao editor de código (`Code.gs`).
2. Na barra de ferramentas superior, seleccione a função `setupSpreadsheet` no menu pendente de funções.
3. Clique em **Run (Executar)**.
4. O Google pedirá permissões de acesso (Autorização necessária). Clique em **Rever permissões**, escolha a sua conta, clique em **Avançadas** e depois em **Ir para o projecto (não seguro)**.
5. Permita o acesso. A função irá criar todas as abas necessárias (incluindo `Preferences` e `ZoneAffinity`, esta última já semeada com os mesmos valores de `ZONE_AFFINITY` do notebook) e 2 utilizadores padrão — um de cada perfil.

### Passo 5: Implantar o WebApp
1. No canto superior direito, clique em **Deploy (Implantar)** > **New deployment (Nova implantação)**.
2. Clique no ícone de engrenagem ao lado de "Select type" e escolha **Web app**.
3. Em *Description*, escreva "Versão 1.0".
4. Em *Execute as*, escolha **Me (o seu email)**.
5. Em *Who has access*, escolha **Anyone (Qualquer pessoa)**.
   *(A segurança é garantida pelo sistema de login do próprio código).*
6. Clique em **Deploy (Implantar)**.
7. Copie o **Web app URL**. É este o link que usará para aceder ao sistema.

## 🔐 Acesso Inicial

- **URL**: (O link copiado no Passo 5)
- **Administrador**: `admin` / `admin123` — gere utilizadores, calibração ZONE_AFFINITY, config. de treino TD-MPC2, CRUD de simulação.
- **Morador (demonstração)**: `morador` / `morador123` — regista preferências de alerta, água, irrigação e percepção de fauna em `preferences.html`.

> **Aviso de Segurança**: Por requisito explícito do projecto, as senhas são armazenadas em texto plano na aba `Users`. Recomenda-se alterar as senhas por omissão no primeiro acesso, através da página "Perfil".

## 📡 Integração de API (Python/Python)

Se pretender enviar dados do seu script Python (`gerar_modelo_3d.py` ou do ambiente de simulação TD-MPC2) directamente para este WebApp, faça um POST para o URL do WebApp com a seguinte estrutura:

```python
import requests
import json

WEBAPP_URL = "https://script.google.com/macros/s/.../exec"

# 1. Autenticar para obter token
auth_res = requests.post(WEBAPP_URL, json={"action": "login", "username": "admin", "password": "sua_senha"})
token = auth_res.json()["data"]["token"]

# 2. Registar um vector de estado
payload = {
    "action": "logState",
    "token": token,
    "episodeId": "EP-20231024-153000-1234",
    "step": 1,
    "stateArray": [0.1, 0.2, 0.3, ...], # Array com 118 posições
    "source": "tdmpc2_env"
}
requests.post(WEBAPP_URL, json=payload)
```
