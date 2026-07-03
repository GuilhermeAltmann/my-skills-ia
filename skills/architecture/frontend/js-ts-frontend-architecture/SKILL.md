---
name: frontend-js-ts-architecture
description: >
  Define padrões arquiteturais para aplicações frontend (Client-side, SPA, SSR) utilizando
  JavaScript ou TypeScript (React, Vue, Angular, etc). Use esta skill ao estruturar o lado
  do cliente, decidir onde alocar componentes, criar hooks/composables ou separar
  responsabilidades entre UI e lógica de negócio.
---

# Frontend JS/TS Architecture — Arquitetura de Componentes

Esta skill define regras arquiteturais modernas para o frontend, focando na
**separação de preocupações (Separation of Concerns)** entre a Apresentação (UI) e a Lógica de Negócios.

---

## Princípio Fundamental: UI é Reflexo do Estado

O Frontend moderno é orientado a estado. A arquitetura deve garantir que:
1. **Lógica de negócio não fique acoplada ao ciclo de vida do DOM.**
2. **Componentes visuais sejam o mais puros (dumb) possível.**
3. **Integrações com API fiquem isoladas.**

---

## Padrão Smart vs Dumb Components

Para manter os componentes testáveis e reusáveis, dividimos a UI em duas categorias:

### 1. Dumb Components (Presentational)
São puramente visuais. Eles não sabem de onde os dados vêm.

- **O que fazem:** Recebem dados via `props` e emitem eventos (ex: `onClick`).
- **O que NÃO fazem:** Não buscam dados em APIs, não têm estado global e não acessam o roteador.

```tsx
// Exemplo (React): Um Dumb Component
export const Button = ({ label, onClick, disabled }) => (
  <button onClick={onClick} disabled={disabled} className="btn-primary">
    {label}
  </button>
);
```

### 2. Smart Components (Containers)
São conectados ao estado da aplicação ou serviços externos.

- **O que fazem:** Chamam hooks de busca de dados, leem o estado global (Zustand/Redux), lidam com lógica complexa e passam esses dados para os Dumb Components.
- **O que NÃO fazem:** Não devem ter estilos complexos ou muita marcação HTML; eles delegam a renderização para os Dumb.

```tsx
// Exemplo (React): Um Smart Component
export const UserProfileContainer = ({ userId }) => {
  const { data: user, isLoading } = useUser(userId); // Fetch logic isolada em um hook
  
  if (isLoading) return <Spinner />;
  
  // Delega a UI para um componente puramente visual
  return <UserProfileDisplay user={user} onEdit={() => navigate('/edit')} />;
};
```

---

## Estrutura de Pastas Recomendada

```
src/
├── components/          ← Dumb Components globais (Button, Modal, Input)
├── features/            ← Feature-Sliced Design (Agrupamento por domínio)
│   └── user-profile/
│       ├── components/  ← Dumb Components específicos da feature
│       ├── hooks/       ← Lógica de estado e fetching
│       ├── services/    ← Integração com a API
│       └── utils/
├── hooks/               ← Hooks globais (useWindowSize, useAuth)
├── services/            ← Clientes HTTP genéricos (axios instance, auth interceptor)
├── pages/ (ou views/)   ← Componentes de rota (geralmente Smart Components)
├── utils/               ← Funções puras utilitárias genéricas
└── types/               ← Definições de tipos TS globais
```

---

## Regras de Integração de API (Services)

O Frontend **nunca** deve fazer requisições HTTP espalhadas pelos componentes de UI.

**Incorreto (Acoplamento Alto):**
```tsx
const UserList = () => {
  const [users, setUsers] = useState([]);
  useEffect(() => {
    fetch('/api/users').then(res => res.json()).then(setUsers);
  }, []);
  // ...
};
```

**Correto (Service Isolado + Custom Hook):**
1. **Service (Contrato HTTP):**
```ts
// src/features/users/services/user.service.ts
export const fetchUsers = async (): Promise<User[]> => {
  const { data } = await api.get('/users');
  return data;
};
```

2. **Hook (Gestão de Estado/Cache):**
```tsx
// src/features/users/hooks/useUsers.ts
export const useUsers = () => {
  return useQuery(['users'], fetchUsers); // Usando React Query, SWR, ou custom wrapper
};
```

3. **Component (Consumidor):**
```tsx
// src/features/users/components/UserList.tsx
export const UserList = () => {
  const { data: users, isLoading } = useUsers();
  // UI logic...
};
```

---

## Anti-padrões no Frontend

| Anti-padrão | Problema | Solução |
|-------------|----------|---------|
| Prop Drilling excessivo | Componentes intermediários conhecem props que não usam | Use Context API ou State Management para estados profundos |
| God Components (Componentes gigantes) | Difícil de manter, testar e estilizar | Quebre em componentes menores e extraia a lógica para Hooks/Composables |
| Dados derivados duplicados no Estado | Causa bugs de dessincronização | Derive dados durante o render (ex: `const fullName = firstName + lastName;`) |
| Imports absolutos longos e cruzados | Código "espaguete" entre features | Mantenha features encapsuladas; exponha apenas o necessário via `index.ts` |

---

## Checklist de Arquitetura Frontend

- [ ] A lógica de busca de dados está isolada da árvore de componentes (em hooks ou stores)?
- [ ] Os componentes visuais reutilizáveis dependem apenas de props (dumb components)?
- [ ] As URLs literais de APIs estão encapsuladas em `/services` e não "hardcoded" na UI?
- [ ] A complexidade condicional do template foi simplificada com extração de subcomponentes?
