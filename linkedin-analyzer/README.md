# Analisador de Perfil LinkedIn

App de página única que diagnostica um perfil do LinkedIn: score por seção, radar de
oito dimensões de posicionamento e plano de ação ordenado por urgência e impacto.

Não tem build, não tem backend, não tem dependência instalada. É um `index.html`
que roda direto no navegador — ou servido por GitHub Pages.

## Como rodar

Abra `index.html` no navegador, ou sirva a pasta:

```bash
python3 -m http.server 8080
# http://localhost:8080/linkedin-analyzer/
```

O botão **Carregar exemplo** preenche um perfil fictício para demonstração.

## Como o score é calculado

Tudo é determinístico e roda no navegador. Nenhum dado do formulário sai da máquina.

**Nove seções**, cada uma pontuada de 0 a 10 e ponderada pelo impacto no perfil:

| Seção | Peso | Seção | Peso |
|---|---|---|---|
| Experiências | 20% | Skills | 9% |
| About / Resumo | 16% | Foto de perfil | 8% |
| Atividade / Conteúdo | 16% | Recomendações | 8% |
| Headline | 14% | Banner / Capa | 6% |
| | | URL personalizada | 3% |

O **score geral** é a média das nove seções ponderada por esses pesos.

**Completude** é medida à parte: 12 itens obrigatórios do perfil, contados como
preenchidos ou não. Um perfil pode ter 100% de completude e score baixo — completo
não é o mesmo que bom, e o relatório mostra exatamente essa diferença.

**As oito dimensões do radar** são combinações ponderadas das seções: posicionamento
e clareza vêm de headline e about; credibilidade de experiências, recomendações e
certificações; prova social e autoridade de recomendações, rede e atividade;
visibilidade e conversão de atividade, skills, URL e chamada para ação.

**A ordem das melhorias** combina a urgência da regra com o prejuízo real —
peso da seção multiplicado pelo quanto falta para o máximo. Uma seção de peso alto
com score zero sobe no topo da lista.

**A referência de rede** varia com o objetivo declarado no formulário:
500 conexões para especialista, 800 para gestão, 1.500 para executivo,
2.000 para comercial, 700 para transição de carreira.

## Enriquecimento opcional com Claude

Com uma chave da API da Anthropic, o Claude reescreve os textos qualitativos do
diagnóstico e redige uma headline e um About sob medida a partir dos dados do perfil.

O que a chave **não** faz: mudar um único número. Score, radar, barras e ordenação
continuam vindo do motor local. O modelo recebe os scores já calculados e é instruído
a nunca contradizê-los. Sem chave — ou se a chamada falhar — o app entrega a análise
local completa, com sugestões geradas por template.

A chamada usa o SDK oficial `@anthropic-ai/sdk`, carregado sob demanda via ESM, com
`dangerouslyAllowBrowser: true` e saída estruturada por JSON Schema.

> **Sobre a chave:** ela vai direto do navegador para a API da Anthropic e fica só na
> máquina do usuário — em memória, ou no `localStorage` se ele marcar a opção. Nenhum
> servidor deste app a recebe. Ainda assim, chave em navegador é chave exposta: para
> demonstração pública use uma chave dedicada, com limite de gasto, e rotacione depois.
> Para uso aberto ao público, o certo é um proxy no servidor guardando a chave.

## Exportar

- **Exportar PDF** usa a impressão do navegador, com folha de estilo própria que
  remove os controles e evita quebra dentro dos cards.
- **Baixar JSON** entrega o resultado completo — perfil, scores, dimensões, achados
  e plano — para arquivar ou comparar duas análises ao longo do tempo.

## Estrutura

Arquivo único, dividido em blocos comentados: tokens do design system, CSS do
formulário, CSS do relatório, markup das duas telas, motor de scoring, geradores de
narrativa, renderizadores de gráfico (SVG inline, sem biblioteca) e a integração
opcional com Claude.
