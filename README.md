# RPG Idle — Front-end (React + Vite + TypeScript + Tailwind)

Interface do **RPG Idle** (Jala University – Matemática Discreta), integrada ao
[backend em NestJS](../rpg-idle-backend). Segue a stack definida no TDD: React,
TypeScript, Vite, Tailwind CSS, e estado global com Context API + hooks (sem Redux/Zustand).

## Como rodar

1. Rode o backend primeiro (em outra pasta/terminal):
   ```bash
   cd rpg-idle-backend
   npm install
   npm run start:dev      # http://localhost:3000/api
   ```
2. Rode o front:
   ```bash
   cd rpg-idle-frontend
   npm install
   npm run dev             # http://localhost:5173
   ```

Em desenvolvimento, o Vite já encaminha `/api` para `http://localhost:3000` (veja
`vite.config.ts`), então não é preciso configurar CORS nem escrever a URL completa
nas chamadas. Se o backend rodar em outra porta, defina `VITE_API_PROXY_TARGET`
antes de `npm run dev`.

Para build de produção com o backend em outro domínio, copie `.env.example` para
`.env` e defina `VITE_API_URL` com a URL completa da API (ex.: `https://minha-api.com/api`).

```bash
npm run build      # gera dist/ (checa os tipos com tsc antes de empacotar)
npm run preview    # serve o build de produção localmente
```

## Como o jogo funciona (game loop)

O tempo é controlado pelo **servidor**, não pelo front. `GameProvider`
(`src/context/GameContext.tsx`) só faz:

1. Carrega o estado do jogo ao montar (`GET /games/:id`).
2. A cada 1s, chama `POST /games/:id/tick` e substitui o estado pelo que a API
   devolve — o front nunca calcula produção, XP ou probabilidade, só exibe.
3. Pausa o tick quando a aba está em segundo plano (`document.visibilityState`)
   e ignora ticks sobrepostos, para nunca ter "dois relógios" rodando ao mesmo tempo.
4. Ao reabrir a página, o primeiro tick já recupera o tempo que passou enquanto
   ela estava fechada (até 8h, limite definido no backend).

O `id` do jogo fica salvo no `localStorage` (`src/hooks/useIdDoJogo.ts`), então
recarregar a página continua a mesma partida.

## Estrutura

```
src/
  types/jogo.ts             tipos espelhando o modelo de dados do backend
  api/                       cliente HTTP (client.ts) e funções por rota (gameApi.ts)
  context/GameContext.tsx    estado global do jogo + game loop (Context API + useReducer)
  hooks/useIdDoJogo.ts       persistência do id do jogo no localStorage
  components/
    icons/Icones.tsx         ícones de linha (sem emoji)
    ui/                      componentes de interface base: Botao, Painel, BarraProgresso, DiscoNivel
    game/                    componentes de domínio: BarraRecursos, CartaoAtividade, ListaAtividades,
                              PainelPersonagem, SeletorEquipamentos, CentralMatematica, DiarioEventos
  screens/
    CriacaoPersonagem.tsx    escolha de classe (busca stats reais em /catalogo/classes) e nome
    Jogo.tsx                 tela principal — monta o layout com os componentes de domínio
  App.tsx                    decide entre CriacaoPersonagem e Jogo, conforme o localStorage
```

Nenhum componente calcula produção, XP ou probabilidade — eles só leem o `EstadoJogo`
devolvido pela API e disparam ações (`iniciar`, `parar`, `equipar`), exatamente como o
TDD pede ("componente apenas lê o estado atual e informa que algo aconteceu").

## Design

Paleta escura tipo "ficha de personagem" (tons de tinta/ardósia), com um acento
dourado para nível e ouro, violeta para experiência e verde-azulado para itens.
Tipografia: **Fraunces** para títulos e números de destaque, **Inter** para texto
de interface (com `tabular-nums` nos contadores, para os números não "pularem" a
cada tick). O elemento de destaque é o disco circular de nível no painel do
personagem; o resto do layout é discreto, com painéis de borda fina.

## Ajustando a URL da API

Se o backend estiver em outra máquina/porta durante a apresentação, edite
`.env` (copiado de `.env.example`) com `VITE_API_URL`, ou ajuste o proxy em
`vite.config.ts` via `VITE_API_PROXY_TARGET`.
