# 🍯 SSH/Telnet Honeypot Deployment & Real-Time Threat Intel

<p align="center">
  <a href="#-english-version">🇺🇸 English</a> •
  <a href="#-versão-em-português">🇧🇷 Português</a>
</p>

---

## 🇺🇸 English Version

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Security](https://img.shields.io/badge/Focus-Threat%20Intel%20%7C%20Cyber%20Deception-red.svg)
![Engine](https://img.shields.io/badge/Honeypot-Cowrie-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

This is my **SSH/Telnet Honeypot Lab**, built on top of **Cowrie**. The idea is simple: instead of only protecting a real server, I set up a fake "bait server" inside an isolated environment that pretends to be a real machine with SSH and Telnet exposed. Anyone trying to break in falls into this trap — and while they do, I get to watch everything: where the attacks come from, what they try to do, and how they behave, all without putting any real workload at risk.

---

### 🛡️ Why this matters for security

Most of the time, cyber defense is reactive — you wait for the attack to happen and then respond. A Honeypot flips that a bit. It sits there, exposed on purpose, acting like an early-warning sensor whenever someone tries something:

- **Understanding attacker behavior:** you get to see exactly which credentials, dictionary words, and commands intruders actually try.
- **Real-time threat intel:** every brute-force IP becomes a fresh Indicator of Compromise (IoC) you can feed into other defenses.
- **Forensics practice:** it's a great way to get hands-on with reading raw logs using everyday Linux tools like `grep` and `awk`.

---

### 🚀 Step-by-Step Implementation Guide

#### 🎛️ Step 1: Isolate the environment first

Before touching any code, get the sandbox right — this is the part you really don't want to skip. Everything runs inside a virtual machine so that whatever happens to the honeypot stays contained.

- **Hypervisor:** VirtualBox
- **OS:** Ubuntu Server 22.04 LTS
- **Network mode:** NAT (the VM can reach the internet to attract attackers, but your host network stays isolated)

#### 🛠️ Step 2: Install the dependencies

Now let's get the shell ready and pull in everything Cowrie needs to run:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip git libssl-dev libffi-dev build-essential
```

#### 📦 Step 3: Set up Cowrie itself

Create a separate, low-privilege user just for the honeypot (never run it as root) and grab the Cowrie source:

```bash
sudo adduser --disabled-password cowrie
sudo su - cowrie
git clone https://github.com/cowrie/cowrie.git
cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
python -m pip install --upgrade pip
python -m pip install -e '.[dev]'
```

#### ⚙️ Step 4: Make it convincing

Copy the default config so you have your own working copy to edit:

```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
nano etc/cowrie.cfg
```

A few tweaks here go a long way in making the fake server feel real:

| Parameter | Example Value | Purpose |
|---|---|---|
| `hostname` | `srv04` | Spoofs a realistic enterprise directory naming convention |
| `listen_port` | `2222` | Sets the internal listening port for the Cowrie backend |
| `download_limit_size` | `10485760` | Mitigates storage exhaustion attacks (bytes) |

#### 🔄 Step 5: Set the trap

Log out of the cowrie user and set up the firewall rules that actually redirect traffic. This is what quietly sends anyone knocking on the standard SSH/Telnet ports straight into the honeypot instead:

```bash
exit
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
```

#### 👁️ Step 6: Turn it on and watch

Back into the cowrie user, start the service, and just... watch what happens:

```bash
sudo su - cowrie
cd cowrie
source cowrie-env/bin/activate
bin/cowrie start
tail -f var/log/cowrie/cowrie.log
```

---

### 📊 Digging through the logs

After letting it run for a while, this is where the fun part starts — pulling out actual intelligence from the raw logs using nothing but `grep` and `awk`:

**Isolate Top 20 Threat Actor IPs:**

```bash
grep 'login attempt' var/log/cowrie/cowrie.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -20
```

**Identify Top 20 Password Dictionary Targets:**

```bash
grep 'login attempt' var/log/cowrie/cowrie.log | awk -F'/' '{print $NF}' | sort | uniq -c | sort -rn | head -20
```

**Audit Executed Attacker Commands (Malicious Shell Commands):**

```bash
grep 'CMD:' var/log/cowrie/cowrie.log
```

---

## 🇧🇷 Versão em Português

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Security](https://img.shields.io/badge/Foco-Threat%20Intel%20%7C%20Cyber%20Deception-red.svg)
![Engine](https://img.shields.io/badge/Honeypot-Cowrie-orange.svg)
![License](https://img.shields.io/badge/Licença-MIT-green.svg)

Esse é o meu laboratório de **Honeypot SSH/Telnet**, montado em cima do **Cowrie**. A ideia é bem simples: em vez de só proteger um servidor de verdade, eu monto uma "isca" dentro de um ambiente isolado, que finge ser uma máquina real com SSH e Telnet abertos. Quem tenta invadir cai direto nessa armadilha — e enquanto isso, eu fico observando tudo: de onde vêm os ataques, o que tentam fazer e como se comportam, sem colocar nada de produção em risco.

---

### 🛡️ Por que isso importa

Na maior parte do tempo, a segurança é reativa: você espera o ataque acontecer pra depois reagir. O Honeypot inverte um pouco esse jogo. Ele fica ali, exposto de propósito, funcionando como um sensor de alerta precoce sempre que alguém tenta algo:

- **Entender o comportamento do atacante:** dá pra ver exatamente quais credenciais, palavras de dicionário e comandos os invasores realmente tentam usar.
- **Inteligência de ameaças em tempo real:** cada IP de força bruta vira um Indicador de Comprometimento (IoC) fresquinho, que dá pra usar em outras defesas.
- **Prática de forense:** é uma ótima forma de treinar a leitura de logs brutos com as ferramentas do dia a dia do Linux, tipo `grep` e `awk`.

---

### 🚀 Guia de Implementação Passo a Passo

#### 🎛️ Passo 1: Primeiro, isole o ambiente

Antes de mexer em qualquer código, deixe o sandbox certinho — essa é a parte que você realmente não quer pular. Tudo roda dentro de uma máquina virtual, pra garantir que o que acontecer com o honeypot fique contido ali.

- **Hypervisor:** VirtualBox
- **Sistema:** Ubuntu Server 22.04 LTS
- **Modo de rede:** NAT (a VM consegue acessar a internet pra atrair atacantes, mas sua rede local fica protegida)

#### 🛠️ Passo 2: Instale as dependências

Agora vamos deixar o shell pronto e baixar tudo que o Cowrie precisa pra funcionar:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip git libssl-dev libffi-dev build-essential
```

#### 📦 Passo 3: Instale o Cowrie

Crie um usuário separado, com privilégios bem restritos, só pra rodar o honeypot (nunca rode como root) e baixe o código do Cowrie:

```bash
sudo adduser --disabled-password cowrie
sudo su - cowrie
git clone https://github.com/cowrie/cowrie.git
cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
python -m pip install --upgrade pip
python -m pip install -e '.[dev]'
```

#### ⚙️ Passo 4: Deixe a isca convincente

Copie o config padrão pra ter sua própria versão editável:

```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
nano etc/cowrie.cfg
```

Alguns ajustes simples aqui fazem bastante diferença pra deixar o servidor falso mais crível:

| Parâmetro | Valor de Exemplo | Finalidade |
|---|---|---|
| `hostname` | `srv04` | Muda o nome visível do servidor para parecer um ativo corporativo real |
| `listen_port` | `2222` | Define a porta interna onde o Cowrie aguardará as conexões |
| `download_limit_size` | `10485760` | Limita o tamanho dos arquivos baixados por invasores, evitando estouro de disco (bytes) |

#### 🔄 Passo 5: Agora sim, arme a armadilha

Saia do usuário cowrie e configure as regras de firewall que realmente redirecionam o tráfego. É isso que manda, discretamente, quem bater nas portas padrão de SSH/Telnet direto pro honeypot:

```bash
exit
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
```

#### 👁️ Passo 6: Ligue e observe

Volte pro usuário cowrie, suba o serviço e simplesmente... acompanhe o que acontece:

```bash
sudo su - cowrie
cd cowrie
source cowrie-env/bin/activate
bin/cowrie start
tail -f var/log/cowrie/cowrie.log
```

---

### 📊 Garimpando os logs

Depois de deixar rodando por um tempo, é aqui que começa a parte boa — tirar inteligência de verdade dos logs brutos usando só `grep` e `awk`:

**Isolar os Top 20 IPs Atacantes:**

```bash
grep 'login attempt' var/log/cowrie/cowrie.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -20
```

**Identificar as Top 20 Senhas Mais Testadas (Dicionário):**

```bash
grep 'login attempt' var/log/cowrie/cowrie.log | awk -F'/' '{print $NF}' | sort | uniq -c | sort -rn | head -20
```

**Auditar Comandos Executados por Invasores (Shell Commands):**

```bash
grep 'CMD:' var/log/cowrie/cowrie.log
```

---

<p align="center">
  Desenvolvido por um entusiasta de Cibersegurança focado em construir software seguro, resiliente e de alta performance.<br>
  <em>Developed by a Cybersecurity enthusiast focused on building secure, resilient, and high-performance software.</em>
</p>


