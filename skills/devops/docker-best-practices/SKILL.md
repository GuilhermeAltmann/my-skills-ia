---
name: docker-best-practices
description: >
  Define as melhores práticas para criação de ambientes Docker (Dockerfile e docker-compose.yml),
  incluindo otimização de imagens, segurança, redes e separação de responsabilidades.
---

# Docker & DevOps — Melhores Práticas

Esta skill define as diretrizes obrigatórias ao configurar containers e ambientes Docker, garantindo imagens leves, comunicação resiliente e arquitetura adequada.

---

## 1. Otimização de Imagens e Build

Ao escrever um `Dockerfile`, as regras de ouro são o uso de cache e a redução do tamanho final da imagem.

### Do's:
- **Use imagens base leves:** Sempre que possível, utilize versões `alpine` ou `slim` (ex: `node:20-alpine`, `golang:1.21-alpine`).
- **Aproveite o cache do Docker:** Copie os arquivos de dependência (como `package.json` e `package-lock.json`, ou `go.mod` e `go.sum`) ANTES de copiar o código-fonte. Instale as dependências e só depois copie o restante. Isso evita que as dependências sejam reinstaladas toda vez que o código muda.
- **Use Multi-stage Builds:** Para linguagens compiladas (Go, Rust) ou assets construídos (React, Vue), use um estágio de "builder" pesado e copie apenas o binário ou a pasta `dist` para um estágio final baseado em `alpine` ou `scratch`.
- **Crie o `.dockerignore`:** SEMPRE gere um arquivo `.dockerignore` ignorando pastas como `node_modules`, `vendor`, `.git`, e arquivos de ambiente (`.env`).

### Don'ts:
- **NÃO instale pacotes de desenvolvimento na imagem final.**
- **NÃO rode a aplicação como `root`.** Crie e utilize um usuário não privilegiado (`USER appuser`).

---

## 2. Comunicação e Redes (Docker Compose)

O `docker-compose.yml` deve conectar os containers de forma segura e resiliente.

### Nomes de Serviço vs IPs
- **Nunca** utilize `localhost` ou `127.0.0.1` dentro de um container para tentar falar com outro.
- **Sempre** utilize o nome do serviço definido no compose (ex: `http://db:5432` ou `http://redis:6379`). O DNS interno do Docker resolverá esse nome automaticamente.

### Redes Personalizadas
- Sempre declare uma rede customizada (bridge) em vez de usar a rede `default`, para melhor isolamento.

### Resiliência e Wait-for-it
- A diretiva `depends_on` por si só apenas aguarda o container iniciar, mas NÃO aguarda o serviço estar pronto para receber conexões (ex: o banco de dados ainda está rodando scripts de boot).
- **Regra:** Para depender de bancos de dados, utilize `depends_on` com `condition: service_healthy` e defina um `healthcheck` no container do banco. Alternativamente, utilize scripts de wait (como `wait-for-it.sh`) no entrypoint da aplicação.

---

## 3. Separação de Responsabilidades (Nginx + App)

Ao expor uma API ou aplicação Web, é antipadrão expor diretamente o processo da aplicação (como PHP-FPM, Uvicorn, ou Go server) para a internet.

- **Regra:** Separe o servidor web (proxy reverso/assets) do processo da aplicação.
- Use um container Nginx na frente para servir arquivos estáticos, lidar com SSL/TLS e fazer proxy para a aplicação.

### Exemplo: Nginx + Go (ou PHP-FPM)
No `docker-compose.yml`:
```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - api
    networks:
      - my_network

  api:
    build: .
    expose:
      - "8080" # Não exposto para o host, apenas para o Nginx
    networks:
      - my_network

networks:
  my_network:
    driver: bridge
```

Arquivo `nginx.conf` de exemplo para Proxy Reverso:
```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://api:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## Checklist de Validação de DevOps

- [ ] A imagem base é leve (`alpine`/`slim`)?
- [ ] Os arquivos de dependência são copiados e instalados antes do código-fonte para aproveitar o cache?
- [ ] Multi-stage build foi utilizado (quando aplicável)?
- [ ] O arquivo `.dockerignore` foi criado?
- [ ] Os containers se comunicam via nome do serviço na rede interna (e não por IP/localhost)?
- [ ] O `depends_on` garante que o banco de dados está realmente pronto (via healthcheck)?
- [ ] Há um Nginx ou proxy reverso na frente da aplicação servindo como Gateway?
