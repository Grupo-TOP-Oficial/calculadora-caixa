# Calculadora de Caixa

Calculadora de fluxo de caixa para projetos de formatura/baile — simula entradas (Adesão, Convites, Mesas), Fee da Agência, custos de Eventos e financiamento via empréstimo quando falta caixa.

## Deploy

- App real: **`index.html`** (single-file, vanilla JS, sem build — Chart.js + xlsx.js via CDN).
- Repositório mora na conta **`grupotopoficial`** (conta pessoal dedicada à empresa, não uma Organization — passou por `theochacon` → `Grupo-TOP-Oficial` (org) → `grupotopoficial` em set/2026. A Organization foi desfeita porque o plano Hobby da Vercel não conecta repositórios privados de organizações, só de contas pessoais. Se algum link antigo com `github.com/theochacon/...` ou `github.com/Grupo-TOP-Oficial/...` aparecer, está desatualizado).
- Publicado ao vivo via GitHub Pages: **https://grupotopoficial.github.io/calculadora-caixa/**, a partir da branch `main`.
- O usuário testa recarregando essa URL pública, **não** um arquivo local. Uma alteração que fica só no working tree é invisível pra ele.
- **Sempre `git commit` + `git push origin main` logo após editar** — não pare para perguntar "posso enviar?". Trate isso como parte normal de terminar a tarefa.

## Cuidado: arquivo obsoleto no repo

`calculadora-caixa.html` na raiz é um rascunho antigo (usa Plotly, não tem import de xlsx) — não é o app. Se algo "não funciona" e deveria, confira se não é esse arquivo (ou uma edição local ainda não commitada) antes de assumir bug de código.

## Repositório é público — cuidado com PII

O repo GitHub é **público**. Nunca commitar planilhas `.xlsx` reais de alunos/clientes (nome, CPF, e-mail, telefone) — use-as só localmente para testar e mantenha fora do git (ver `.gitignore`).

## Importador de planilhas (.xlsx)

`processarArquivo()` detecta o formato pelo cabeçalho e roteia para o parser certo. Formatos suportados hoje:

1. **Legado** — colunas `Valor do Plano · 1º Pagamento · Nº Parc. Adesão`.
2. **BT** — cabeçalho começa com `Contrato Id`; exige seleção de turma na UI; agrupa parcelas não pagas por contrato.
3. **Formandos** — cabeçalho `Contrato` / `Nome` / ... `Aguardando Pagamento (líquido)`. Um formando por linha, com totais já agregados. **Importante:** as colunas `Em Atraso` e `Aguardando Pagamento` são só o *status atual* da parcela daquele contrato — não usar para calcular saldo. A lógica correta reconstrói o cronograma **completo** do plano, ignorando status de pagamento:
   - `1ª parcela = Última Parcela − (Qtd de Parcelas − 1)`
   - `parcela mensal = (Total − Lançamento sem parcela) / Qtd de Parcelas`
   - Aplicado de forma cumulativa (todas as parcelas, passadas e futuras), sem filtrar por status.
   - Essa versão foi validada contra o total mensal real reportado pelo sistema de origem do usuário (ficou a ~14% de diferença, contra ~4x de erro da versão anterior baseada em saldo restante).

Se aparecer um novo formato de exportação, seguir o mesmo padrão: ler o arquivo, achar a assinatura do cabeçalho, escrever um par `detectarFormatoX` + `parsarX`, plugar em `processarArquivo`.
