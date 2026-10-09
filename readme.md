# Explorando Práticas de Teste: stenciljs/core

## Passo 1: Repositório selecionado

**Repositório:** [stenciljs/core](https://github.com/stenciljs/core)

O Stencil é um compilador mantido pela equipe do Ionic que gera Web Components a partir de código TypeScript/JSX. Escolhi esse projeto porque ele é grande e maduro. Além disso, como o código que ele gera roda no navegador, o projeto precisa de vários tipos de teste diferentes, o que deixa a análise mais interessante.

## Passo 2: Exploração no TestMiner

Página analisada: https://andrehora.github.io/testminer/#stenciljs/core

### Onde ficam os testes

| Pasta | Arquivos de teste |
|---|---|
| `test/` | 1007 |
| `src/` | 391 |
| `.github/` | 9 |
| `scripts/` | 8 |
| `.` (raiz) | 1 |
| `docs/` | 1 |
| `screenshot/` | 1 |

A maior parte dos arquivos está na pasta dedicada `test/`. Mesmo assim, há muitos (391) dentro de `src/`, junto do código de produção.

### Evolução dos testes

| Versão | Testes (aprox.) |
|---|---|
| 0.0.0 | ~250 |
| 1.11.0 | ~235 |
| 4.45.1 | ~470 |

Da versão 0.0.0 para a 1.11.0, o número ficou quase estável, com uma leve queda. Da 1.11.0 até a 4.45.1, ele praticamente dobrou.

### Dependências de teste (npm)

| Papel | Bibliotecas |
|---|---|
| Executor de testes de unidade | `jest`, `jest-circus`, `jest-runner`, `jest-runtime`, `jest-cli`, `jest-config` e demais módulos `jest-*` |
| DOM simulado | `jsdom`, `jest-environment-jsdom`, `jest-environment-node` |
| Snapshots | `jest-snapshot`, `jest-serializer` |
| Mocks, espiões e controle de tempo | `jest-mock`, `spy`, `tinyspy`, `fake-timers`, `sinonjs__fake-timers` |
| Navegador real / E2E | `puppeteer`, `puppeteer-core`, `karma`, `karma-jasmine`, `karma-chrome-launcher`, `karma-typescript`, `jasmine-core`, `webdriverio`, `mocha`, `mocha-framework` |
| Cobertura de código | `istanbul-lib-coverage`, `istanbul-lib-instrument`, `istanbul-lib-report`, `istanbul-lib-source-maps`, `istanbul-reports`, `test-exclude` |

Observações:
- O Jest e o Puppeteer não servem só para testar o próprio Stencil. Eles também fazem parte da API de testes que o Stencil oferece a quem cria componentes com ele (`stencil test --spec` e `stencil test --e2e`).
- `evp_bytestokey` e `safe-regex-test` não são bibliotecas de teste. Provavelmente aparecem na lista porque o nome contém "test", ou seja, são falsos positivos da detecção por nome.

### Testes no CI

O TestMiner identificou workflows de teste separados por área, como `analysis`, `build`, `bundlers`, `component` e `copytask`, entre outros.

## Passo 3: Prática de teste: testes em várias camadas

A prática que mais chama atenção no Stencil é testar o software em **camadas diferentes**, cada uma com ferramentas próprias, em vez de depender de um único tipo de teste.

**1. Testes de unidade perto do código.** Os 391 arquivos de teste em `src/` ficam ao lado dos módulos que testam. Eles usam o Jest e verificam partes do compilador de forma isolada. Quando precisam de um DOM, usam o `jsdom`, que simula o navegador dentro do Node. Para isolar dependências ou controlar o tempo, usam mocks, spies e fake timers. São testes rápidos, que dão retorno imediato a quem está mexendo no código.

**2. Testes de integração e em navegador real numa pasta dedicada.** A pasta `test/`, com 1007 arquivos, concentra os testes que exercitam o compilador de ponta a ponta. Eles compilam componentes de exemplo e verificam o resultado em um navegador de verdade, com Karma + Jasmine, WebdriverIO + Mocha e Puppeteer. Isso é necessário porque o produto do Stencil é código que roda no navegador. Alguns problemas, como renderização, Shadow DOM, ciclo de vida dos componentes e integração com bundlers, só aparecem em ambiente real e passariam despercebidos num DOM simulado.

**3. CI dividido por área.** Com workflows separados (`bundlers`, `component`, `copytask`, `analysis`...), uma falha aponta direto para a parte do sistema que quebrou. Cada grupo de testes também pode rodar de forma independente.

**Por que isso é uma boa prática:** existe um compromisso entre velocidade e fidelidade. Testes de unidade são baratos e rápidos, mas não garantem que o sistema funciona como um todo. Testes em navegador são mais lentos e trabalhosos, mas reproduzem o uso real. Combinando os dois, o projeto tem retorno rápido no dia a dia e, ao mesmo tempo, confiança de que o código gerado funciona onde importa. A cobertura medida com Istanbul complementa isso, mostrando quais partes do código os testes realmente exercitam.

Um detalhe interessante é que Karma e WebdriverIO aparecem ao mesmo tempo. Como o Karma foi descontinuado, isso sugere que o projeto está migrando (ou já migrou) seus testes em navegador para o WebdriverIO. Ou seja, a infraestrutura de testes também evolui junto com o projeto. O histórico mostra esse crescimento: o número de testes praticamente dobrou entre as versões 1.11.0 e 4.45.1.
