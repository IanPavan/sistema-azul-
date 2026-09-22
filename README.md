# Simulador de Portabilidade INSS

Sistema para simular portabilidade pura, portabilidade + refinanciamento e margem livre
de empréstimos consignados do INSS, a partir do extrato em PDF do beneficiário.

## Arquitetura: 100% local, sem custo

Este projeto **não tem backend, não usa IA e não precisa de nenhuma chave de API.**
Tudo roda no navegador de quem usa:

- A leitura do PDF usa a biblioteca [pdf.js](https://mozilla.github.io/pdf.js/) (carregada
  via CDN) para extrair o texto com as posições exatas de cada palavra na página.
- Como o extrato do INSS segue sempre o mesmo layout (mesmas colunas, mesmas posições),
  o sistema reconstrói a tabela de contratos e os campos do benefício usando essas posições
  — sem depender de nenhuma IA para "entender" o PDF.
- Todo o motor de cálculo (Tabela Price, prazo máximo por idade, regras por banco) também
  roda inteiramente no navegador.

Isso significa: **um único arquivo `index.html`**, sem servidor, sem variável de ambiente,
sem custo por PDF processado. Pode ser hospedado em qualquer lugar que sirva arquivos
estáticos — GitHub Pages, Vercel, Netlify, ou até aberto localmente no navegador.

## Estrutura

- `index.html` — front-end completo: upload do PDF, extração local, tela de conferência
  editável, os 3 fluxos de simulação e o comparativo entre bancos.

## Publicando no GitHub Pages

1. Suba este repositório para o GitHub (pode ser público ou privado — GitHub Pages funciona
   com os dois, mas em repositório privado o site publicado também fica privado se a conta
   for de plano pago; em conta gratuita, publicar a partir de um repo privado exige o plano Pro).
2. No repositório: **Settings → Pages → Source** → escolha a branch `main` e a pasta raiz (`/`).
3. Em alguns minutos o GitHub gera um link tipo `seu-usuario.github.io/nome-do-repo/`.

## Limitação conhecida da extração automática

Como a leitura é baseada nas posições fixas do modelo atual do extrato do INSS, ela depende
de esse modelo continuar igual. Se o INSS mudar o layout do PDF no futuro, a extração
automática pode parar de funcionar corretamente — nesse caso, os dados sempre podem ser
digitados manualmente na tela de conferência, que já é editável.

## Bancos parametrizados

`bancosConfig` dentro de `index.html` concentra as regras de cada banco (idade máxima,
taxas, troco mínimo, `naoPorta`, acordos comerciais). BMG não está incluído. BRB está
listado como inativo até ser reativado.

## Observações de projeto

- A extração nunca sobrepõe os valores de margem já impressos no extrato do INSS — eles são
  a fonte da verdade; a constante `REGRAS_MARGEM_INSS` serve só como conferência/projeção.
- `naoPorta` e `acordoBanco` nunca bloqueiam silenciosamente: o banco correspondente some
  da lista de opções válidas com uma mensagem explicando o motivo.
- A comparação de nomes de banco (`bancoCasa`) ignora espaços internos, porque o extrato às
  vezes quebra o nome do banco no meio da palavra ao virar linha na célula do PDF (ex.:
  "INBURS A" em vez de "INBURSA").
- Datas de contrato: o extrato traz duas informações diferentes — "INÍCIO DE DESCONTO"
  (competência, só mês/ano) e "PRIMEIRO DESCONTO" (data exata, dia/mês/ano). O cálculo de
  parcelas pagas usa a data exata (`dataPrimeiroDesconto`) sempre que disponível.
- A interpretação de "idade máxima" em `bancosConfig` (seção 6.1 da especificação original)
  compara a idade atual do cliente contra o limite de cada faixa. Se a fonte de dados
  (bevi) na verdade quiser dizer "idade ao final do contrato", ajustar a função
  `prazoMaximoPorIdade` em `index.html` (está comentada no código).
