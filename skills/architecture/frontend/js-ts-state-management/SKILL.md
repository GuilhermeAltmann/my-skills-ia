---
name: frontend-js-ts-state-management
description: >
  Define padrões para o gerenciamento de estado no frontend (Server State vs Client State).
  Use esta skill para decidir entre estado local, global ou cache de requisições, evitando
  complexidade desnecessária com Redux/Zustand em dados que pertencem ao servidor.
---

# Frontend JS/TS State Management — Gerenciamento de Estado

No frontend, a principal complexidade não é o estado em si, mas sim a confusão entre o estado que pertence à UI e o estado que pertence ao Servidor. Esta skill define regras para lidar com isso de forma eficiente.

---

## 1. As 3 Categorias de Estado

O estado deve ser categorizado antes de ser gerenciado. Nunca coloque tudo em uma única Store (ex: Redux global) indiscriminadamente.

### A. Estado Local (Local State)
Estado que pertence apenas a um componente e seus filhos diretos.
- **Exemplos:** Modais abertos/fechados, formulários (input texts), abas selecionadas.
- **Como gerenciar:** `useState`, `useReducer`, `ref`.
- **Regra:** Tente manter o estado o mais próximo possível de onde ele é usado.

### B. Estado do Servidor (Server State)
Dados assíncronos que residem no banco de dados, mas que o frontend faz um "cache" para exibição.
- **Exemplos:** Lista de usuários, perfil logado, produtos no e-commerce.
- **Como gerenciar:** Ferramentas de Data Fetching / Async Cache (React Query, SWR, Apollo Client).
- **Regra:** **NÃO use Redux ou Zustand para salvar respostas de API.** Deixe o React Query/SWR lidar com loading, erro, cache, deduplicação e stale-while-revalidate.

### C. Estado Global da UI (Client Global State)
Estado estritamente da interface que precisa ser acessado por vários componentes distantes na árvore.
- **Exemplos:** Tema (Dark/Light Mode), idioma atual, carrinho de compras (antes do sync com API), toasts de notificação.
- **Como gerenciar:** Zustand, Redux Toolkit, Vuex/Pinia, ou Context API.
- **Regra:** Use parcimoniosamente. Se o estado for apenas "Client Global", aí sim uma store faz sentido.

---

## 2. Regras de Ouro do Estado

1. **Evite Estados Derivados / Duplicados**
   Se um estado pode ser calculado a partir de outro, calcule-o durante o render. Não crie um novo `useState`.
   
   **Incorreto:**
   ```tsx
   const [items, setItems] = useState([1, 2, 3]);
   const [count, setCount] = useState(3); // Estado derivado manual
   ```
   **Correto:**
   ```tsx
   const [items, setItems] = useState([1, 2, 3]);
   const count = items.length; // Calculado no render
   ```

2. **Evite Prop Drilling Extremo**
   Passar props por mais de 3 níveis de componentes indica que a estrutura precisa ser refatorada (uso de Component Composition, Context, ou Zustand).

3. **Imutabilidade Sempre**
   Nunca mute o estado diretamente. No React/Zustand isso quebra a re-renderização.
   - Use spread operator `[...arr, newItem]` ou `{ ...obj, newKey: val }`.
   - Para estados muito aninhados, considere usar bibliotecas como `Immer`.

---

## 3. Padrão de Contexto (Context API)

Para evitar re-renderizações desnecessárias em Context APIs nativas (no React), separe o Estado das Funções de Mutação.

```tsx
// Exemplo: Tema
const ThemeStateContext = createContext<'light'|'dark'>('light');
const ThemeDispatchContext = createContext<(theme: 'light'|'dark') => void>(() => {});

// Componente não re-renderiza se usar apenas o Dispatcher
```

---

## 4. Checklist de Decisão de Estado

Sempre que precisar criar um estado, faça estas perguntas em ordem:

1. [ ] **Esse dado pode ser calculado a partir de outra variável existente?**
   👉 *Sim:* Calcule durante a renderização. Fim.
2. [ ] **Esse dado veio de uma requisição HTTP e precisa de Loading/Error state?**
   👉 *Sim:* Use Server State (React Query / SWR / Fetch Hook). Fim.
3. [ ] **Esse dado é usado apenas neste componente e em seus filhos diretos?**
   👉 *Sim:* Use Local State (`useState` / `useReducer`). Fim.
4. [ ] **Esse estado da UI é necessário em partes completamente diferentes do app?**
   👉 *Sim:* Use Global Client State (Zustand, Context, Pinia). Fim.
