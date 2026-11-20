---
name: conduzir-entrega
description: Use quando a pessoa relatar um bug ou erro, trouxer um card ou log para investigação, ou pedir planejamento, implementação, testes, revisão de código ou verificação de release. Conduza a etapa solicitada e acione especialistas conforme o risco e a evidência necessária.
---

# Conduzir uma entrega

## Seleção e despacho

Zeus é a sessão principal. Os outros 15 nomes são papéis de workers acionados por demanda. Cada linha abaixo fornece a instrução específica que Zeus deve incluir no despacho. O perfil Codex padrão de Zeus é `gpt-6.1-sol` com esforço `medium`. Quando um plano não trivial exigir um despacho dedicado, use `gpt-6-astra` com esforço `high` e o fluxo de planejamento preferido pelo projeto. A sessão principal conserva o modelo escolhido no cliente.

Use o mesmo SHA ou a mesma fotografia do diff para auditores que devem comparar o mesmo resultado. Cada despacho começa em sessão nova e informa papel, modelo, esforço, tarefa, base, caminhos autorizados, evidência esperada e condição de término. Entregue a spec, o plano ou um diff em arquivo acessível ao worker. Registre os resultados no artefato da entrega para que outro worker possa continuar. O alvo do desenho original é até 40% da janela por sessão. Se a ocupação for observável, faça o handoff antes desse limite.

Para implementar, delegue a tarefa a um worker pelo bot. O Orca cria um worktree próprio para a mudança. Se a entrega já tiver esse worktree, confirme sua identidade antes de iniciar outra sessão nele com `orca agent launch`. Passe o seletor confirmado em `--worktree`, o provedor em `--agent codex`, o ID em `--model`, o esforço em `--effort` e o despacho em `--prompt`. Verifique o recibo com `--json`. Registre o handle devolvido. Em timeout, confira as sessões existentes antes de repetir. Use os controles de acesso oferecidos pelo runtime e explicite no despacho o escopo de leitura ou escrita. O nome mitológico identifica o papel no prompt.

Mantenha uma implementação por vez. Execute até duas leituras independentes em paralelo por padrão. Uma terceira exige ganho concreto e recursos independentes. Reserve um simulador por execução e isole bancos, portas e filas em testes concorrentes. Ao corrigir o diff, invalide as verificações afetadas pela nova revisão.

## Catálogo de papéis

Na tabela, Luna significa `gpt-6-luna`, Sol significa `gpt-6.1-sol` e Astra significa `gpt-6-astra`. Os modelos são escolhas de roteamento. Confira a disponibilidade no provedor e informe qualquer fallback. Para modelo solicitado indisponível, use Sol/high se disponível.

| Papel | Modelo / esforço | Quando acionar e o que devolver |
|---|---|---|
| Zeus | Sol/medium | Coordene escopo, aprovações, spec, plano e evidências. Ao retomar, confira a base e a última tarefa concluída antes de despachar. |
| Hermes | Luna/low | Para localizar um símbolo ou fluxo conhecido, devolva entradas, chamadores e arquivo:linha. Escale para Sol se as evidências admitirem interpretações distintas. |
| Íris | Sol/low | Havendo um nó do Figma, extraia tokens, medidas, componentes e estados. Informe a referência do nó e marque os detalhes que o design não especifica. |
| Atena | Sol/high | Quando houver uma decisão estrutural, compare alternativas pelo impacto em contratos, falhas, latência e custo de migração. Recomende uma e registre a condição que mudaria a escolha. |
| Prometeu | Sol/high | Antes de aprovar spec ou plano, identifique critérios impossíveis de verificar, contradições, lacunas de recuperação e dependências omitidas. Cite trecho e consequência. |
| Hefesto | Sol/medium | Implemente uma tarefa aprovada. Para uma regressão testável, reproduza-a em teste antes da correção. Entregue diff, testes executados, resultados e pendências. Use Sol/high para integração estrutural ou sincronização. |
| Quíron | Sol/medium | Diante de falha, isole ambiente, entrada e causa. Devolva reprodução, evidência e hipóteses descartadas. Após duas tentativas sem progresso, encaminhe a Sol/high com o contexto reunido. |
| Argos | Sol/medium | Quando a aceitação depender do app rodando, exercite o fluxo no simulador. Devolva build, ambiente, passos, resultado e log ou captura. Marque os estados que não conseguiu alcançar. |
| Têmis | Sol/high | Revise o diff e os contratos depois dos gates. Priorize regressões observáveis. Informe arquivo, linha, cenário e consequência. Reavalie os achados se o diff mudar. |
| Ártemis | Sol/medium | Para comportamento alterado, identifique qual teste falharia se a regressão voltasse. Aponte mocks que encobrem a falha. Use Sol/high para desenho de concorrência, carga ou recuperação. |
| Clio | Luna/low | Quando um fluxo, contrato ou dependência documentada mudar, compare o diff com o trecho publicado. Entregue a referência e a correção textual mínima. |
| Cérbero | Sol/high | Revise defensivamente fronteiras alteradas de autorização, segredos, dados pessoais, dependências ou CI. Relacione o trecho à política do repositório e proponha a mitigação. Use Astra/high em risco material de autorização, exposição ou perda de dados. |
| Atalanta | Sol/medium | Para mudanças em listas, animações, navegação ou bootstrap, identifique o caminho crítico e compare medições sob as mesmas condições. Separe hipótese de regressão medida. |
| Afrodite | Sol/medium | Quando houver Figma e app executável, compare a mesma tela, estado e viewport. Entregue captura, nó de referência e desvios de layout, tokens ou tipografia. |
| Hélio | Sol/medium | Após um release identificado e com acesso de leitura, compare crashes, erros e latência por versão e janela. Informe volume e limitações de atribuição. Prepare um rascunho de card quando houver evidência. |
| Cronos | Luna/low | Sob demanda ou por agenda autorizada, confronte versões instaladas, avisos oficiais e versões corrigidas. Entregue dependência, versão afetada e fonte. Encaminhe análise de impacto ambígua a Cérbero. |

## Fluxo e recuperação

1. **Entrada.** Fixe a demanda, a base do repositório e o critério de aceite. Leia código, contratos e testes antes de propor mudanças. Para trabalho com UI, Hermes e Íris podem pesquisar em paralelo.
2. **Decisão.** Para mudança não trivial, Zeus registra a spec, solicita análise de Atena quando houver trade-off material e usa Prometeu para revisar lacunas. Após aprovação, decomponha o plano em tarefas verificáveis. Em mudança persistente, descreva compatibilidade de rollout, rollback de código e recuperação de dados.
3. **Implementação.** Despache apenas a tarefa atual a Hefesto. Informe quem é dono dos arquivos. Em falha de ambiente ou permissão, resolva a dependência antes de escalar modelo.
4. **Validação.** Identifique os scripts existentes e a matriz de verificação do repositório. Em um projeto TypeScript, os candidatos podem incluir ts:check, lint, Jest, i18n:check e agents:check. Registre comando, saída e revisão. Se um gate obrigatório falhar, devolva a tarefa a Hefesto. Um teste ausente ou impedido fica pendente.
5. **Revisão.** Acione Têmis para a entrega e apenas os especialistas cujos gatilhos se aplicam ao diff. Argos valida comportamento executável. Afrodite compara fidelidade visual. Quando ambos precisarem da mesma tela, compartilhe a captura identificada por build e estado.
6. **Entrega.** Apresente resultado, evidências e pendências. Respeite o alvo de branch e o procedimento do projeto. Em operação externa de resultado incerto, consulte o estado por card, SHA, MR ou versão antes de tentar novamente. Hélio depende da versão publicada e de telemetria acessível. Cronos requer uma agenda configurada separadamente para execução periódica.
