---
name: laravel-best-practices
description: >
  Diretrizes para o uso do framework Laravel priorizando as ferramentas nativas (O "Laravel Way"),
  sem over-engineering (arquitetura hexagonal/limpa complexa), focando em performance, Octane, ORM e filas.
---

# Laravel Best Practices — The "Laravel Way"

Esta skill define as melhores práticas para trabalhar com Laravel, focando em extrair o máximo do ecossistema e ferramentas nativas, priorizando a performance sem sacrificar a produtividade introduzindo abstrações complexas e desnecessárias como Arquitetura Limpa ou Hexagonal.

---

## 1. Ferramentas Nativas (Não reinvente a roda)

O Laravel possui ferramentas prontas para quase todos os problemas de backend.
- **Form Requests:** **NUNCA** valide requests diretamente no Controller. Crie classes de `FormRequest` (`php artisan make:request`) e delegue toda a validação de input para elas.
- **Policies & Gates:** A autorização de acesso a entidades (ex: "um usuário pode deletar este post?") deve ser definida em **Policies**, não em ifs espalhados nos controllers ou services.
- **API Resources:** Ao retornar dados via API, **NUNCA** retorne o Model diretamente (`return User::all();`). Use `JsonResource` (`php artisan make:resource`) para formatar e mascarar os dados que vão para o cliente.
- **Jobs & Queues:** Qualquer operação que leve mais de 1 segundo (envio de emails, processamento de imagem, chamadas a APIs externas) **DEVE** ser despachada para uma fila (`Queue::push(new ProcessVideo($video))`).

---

## 2. Otimizações de ORM (Eloquent)

O Eloquent é produtivo, mas pode ser lento se usado de forma descuidada. 
Siga rigorosamente estas micro-otimizações de ORM:

1. **Evite N+1 (Eager Loading):**
   - **Incorreto:** `$users = User::all(); foreach($users as $u) { echo $u->profile->bio; }` (Gera 1 query para buscar users + 1 query por perfil).
   - **Correto:** `$users = User::with('profile')->get();`
2. **Use `select()` para poupar memória:**
   - Evite `User::all()` ou `User::get()` se precisar apenas de `id` e `name`.
   - **Correto:** `User::select('id', 'name')->get();` ou `User::pluck('name', 'id');`
3. **Chunking em Grandes Volumes:**
   - Nunca use `all()` ou `get()` em tabelas com milhares de registros.
   - Use `chunk(1000)` ou `lazy()` para processar com uso mínimo de memória RAM.
4. **Bulk Inserts e Upserts:**
   - Não faça loop com `save()`:
     ```php
     // INCORRETO
     foreach($data as $item) { Model::create($item); }
     ```
   - Use inserção em massa:
     ```php
     // CORRETO
     Model::insert($data); // ou upsert()
     ```

---

## 3. Caching e Otimização de Servidor

- **Queries repetitivas:** Se uma query de catálogo ou configuração que muda raramente for chamada, faça um wrap no cache:
  ```php
  $settings = Cache::remember('site_settings', 3600, fn() => Setting::all());
  ```
- **Processamento no Banco:** Evite manipular dados no PHP (ex: `filter()`, `map()`, calculos complexos em `Collection`) se o banco de dados puder fazer nativamente no SQL (`where`, `sum`, `groupBy`).

### Preparação para Produção
Antes do deploy, **SEMPRE** execute as otimizações nativas:
- `php artisan config:cache`
- `php artisan route:cache`
- `php artisan view:cache`

---

## 4. Prioridade de Performance: Laravel Octane

Para servidores e APIs de altíssima performance, o padrão tradicional (Nginx + PHP-FPM) gera overhead pelo "boot" do framework em cada requisição.
- **Regra:** Sempre priorize a configuração de servidores usando **Laravel Octane** (com Swoole ou RoadRunner) para manter o framework carregado na RAM.
- **Cuidados com Octane (Memory Leaks):**
  - O estado de classes Singleton e propriedades estáticas **NÃO** é limpo entre requisições no Octane.
  - Evite injetar objetos de estado globalmente ou inicializar propriedades estáticas que variam por usuário.
  - O contêiner de Injeção de Dependência persiste; tome cuidado ao fazer `app()->singleton()`.

---

## Checklist de Validação Laravel

- [ ] A lógica de validação está separada do Controller (usando `FormRequest`)?
- [ ] A autorização está encapsulada em `Policies`?
- [ ] O Controller usa `API Resources` para ditar o formato do payload de saída?
- [ ] O Eloquent está evitando o N+1 com `with()`?
- [ ] Grandes volumes de dados usam `chunk()` ou `lazy()` ao invés de `get()`?
- [ ] Consultas trazem apenas as colunas necessárias (`select`)?
- [ ] Tarefas lentas foram passadas para `Jobs` na fila em vez de aguardar síncrono?
- [ ] O ambiente suporta Laravel Octane (Swoole/RoadRunner) para alta performance?
