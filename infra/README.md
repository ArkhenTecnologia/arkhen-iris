# Infra — ÍRIS (Fase 0: Setup)

Runbook para subir a stack num VPS **Ubuntu Server LTS** (alvo: 4 vCPU / 8 GB / NVMe).
Critério de pronto da Fase 0: **a stack sobe, o Nginx serve HTTPS e `/health` responde.**

> **Antes de começar — pendências que afetam a execução:**
> - `[A CONFIRMAR]` o plano **HostGator** dá **root/SSH + Docker livres** e **datacenter no Brasil**. Se o plano for um n8n gerenciado fechado, contrate um VPS Cloud Ubuntu LTS comum.
> - Dois subdomínios no DNS apontando para o IP do VPS: **`iris.SEU_DOMINIO`** (n8n) e **`bi.SEU_DOMINIO`** (Metabase), preservando o MX do Titan.

## Arquivos

| Arquivo | O que é |
|---|---|
| `docker-compose.yml` | 3 PostgreSQL (iris, n8n, metabase) + n8n + Metabase + Nginx |
| `nginx/iris.conf` | reverse proxy TLS; só 443 público; 80 só redirect + ACME |
| `../.env.example` | modelo das variáveis (copie para `../.env` e preencha) |

## Passos

### 1. Acesso e atualização
```bash
ssh usuario@IP_DO_VPS
sudo apt update && sudo apt upgrade -y
```

### 2. Docker + Compose plugin
```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
# saia e entre de novo na sessão SSH para o grupo valer
docker compose version
```

### 3. Firewall (UFW) — negar por padrão, abrir só o necessário
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 80/tcp        # ACME + redirect
sudo ufw allow 443/tcp       # único acesso público à aplicação
sudo ufw allow from SEU_IP_FIXO to any port 22 proto tcp   # SSH só do seu IP (allowlist)
sudo ufw enable
sudo ufw status verbose
```
> Troque `SEU_IP_FIXO`. Se não tiver IP fixo, use VPN. **Nunca** deixe o 22 aberto para todos.

### 4. Clonar o repositório
```bash
git clone https://github.com/ArkhenTecnologia/arkhen-iris.git
cd arkhen-iris
```

### 5. Variáveis de ambiente
```bash
cp .env.example .env
nano .env            # preencha TODAS as senhas (fortes e únicas) e a N8N_ENCRYPTION_KEY
# dica para a chave:  openssl rand -hex 32
```

### 6. Ajustar o domínio no Nginx
```bash
sed -i 's/SEU_DOMINIO/arkhentecnologia.com.br/g' infra/nginx/iris.conf
```

### 7. Certificados TLS (Let's Encrypt / Certbot)
```bash
sudo apt install -y certbot
# com o DNS já apontando para o VPS e a porta 80 livre:
sudo certbot certonly --standalone -d iris.arkhentecnologia.com.br -d bi.arkhentecnologia.com.br
# certificados ficam em /etc/letsencrypt — o compose monta essa pasta em ./certs
sudo ln -s /etc/letsencrypt infra/certs   # ou ajuste o volume no compose para /etc/letsencrypt
```
> Renovação automática: o Certbot já instala um timer. Teste com `sudo certbot renew --dry-run`.

### 8. Subir a stack
```bash
cd infra
docker compose --env-file ../.env up -d
docker compose ps        # todos "healthy"/"running"
docker compose logs -f nginx
```

### 9. Verificar o critério de pronto
```bash
curl -I https://iris.arkhentecnologia.com.br/health    # deve responder 200
```
Abra no navegador: `https://iris.arkhentecnologia.com.br` (n8n) e `https://bi.arkhentecnologia.com.br` (Metabase). Conclua o primeiro acesso de cada um e **ative MFA**.

## Segurança (checar ao final da Fase 0)
- [ ] Só 80/443 públicos; SSH restrito por IP/VPN (UFW).
- [ ] `.env` preenchido, **fora do Git** (já está no `.gitignore`).
- [ ] `N8N_ENCRYPTION_KEY` definida e guardada.
- [ ] HTTPS válido nos dois subdomínios; HSTS ativo.
- [ ] MFA no n8n e no Metabase.
- [ ] Backup: a configurar na Fase 9 (dump diário dos bancos + teste de restauração).

## Não construir ainda (fora da Fase 0)
Banco/DDL e RLS (Fase 1), captura IMAP e workflows (Fase 2+), IA (Fase 3), dashboards (Fase 5). Esta fase entrega só a **casca de infra** rodando com HTTPS.
