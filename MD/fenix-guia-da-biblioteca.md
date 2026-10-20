# GUIA DA BIBLIOTECA FÊNIX (v3.6)

Este guia explica, em português claro, tudo o que existe na biblioteca Fênix: o que cada função do Fênix Core faz e para que serve cada regra do Livro das Regras. Ele também funciona como o **Changelog da biblioteca** (Parte 4). Acompanha todo pacote novo e é atualizado a cada versão, junto com o livro, o core, os templates e os documentos legais, que andam sempre com o mesmo número (regra 7).

Endereço base de tudo: https://fenicris-dev.github.io/Fenix-Core/

## PARTE 1. MAPA DA BIBLIOTECA

| Arquivo | Onde fica | Para que serve |
|----------------------------------------|------------------|------------------------------------------|
| `fenix-core.js` | raiz | Biblioteca de funções que todo app carrega (módulo ES, exporta `FenixCore`). |
| `index.html` | raiz | Painel da biblioteca: testa cada endereço oficial e confere se tudo está na mesma versão. |
| `fenix-template-login.html` | raiz | Layout oficial de login de todo app com senha. |
| `fenix-template-onboarding.html` | raiz | Layout oficial da tela de primeiro acesso. |
| `Icon/Fenix_Icone_64, 128, 192, 512.png` | pasta `Icon/` | Ícone da águia (branca sobre o degradê): favicon, cabeçalho, PWA, splash e ícone provisório. |
| `Logo/Fenix_Logo_Azul*.png`, `Fenix_Logo_Branca*.png` | pasta `Logo/` | Logo oficial (Azul) e segunda opção (Branca), também em 320 e 640 px. |
| `Logo/Fenix_Logo_Black.png`, `Fênix-Logo-Silver.png`, `Fênix-Logo-Gray.png` | pasta `Logo/` | Logos auxiliares, para escolher o melhor contraste. |
| `Termos/fenix-termos-de-uso.html`, `Termos/fenix-politica-de-privacidade.html` | pasta `Termos/` | Textos legais oficiais, linkados no login e no onboarding. |
| `MD/fenix-livro-das-regras.md`, `DOC/fenix-livro-das-regras.docx` | pastas `MD/` e `DOC/` | O Livro das Regras. |
| `MD/fenix-guia-da-biblioteca.md`, `DOC/fenix-guia-da-biblioteca.docx` | pastas `MD/` e `DOC/` | Este guia. |

Todo material com versão tem duas cópias (regra 11): a **neutra**, sem versão no nome (é a que os apps usam), e a **com versão no nome** (`-v3.6`), só para controle interno.

## PARTE 2. O FÊNIX CORE, FUNÇÃO POR FUNÇÃO

### 2.1 Como carregar e iniciar

O core é um módulo ES: não funciona colado como `<script>` comum. Use o carregador oficial da regra 2, que guarda uma cópia no aparelho e permite abrir o app sem internet.

- **`FenixCore.init(nomeApp)`**: liga a conexão com o banco do core (projeto Firebase "fenix-core"). Chame uma vez só, ao abrir o app, com o mesmo identificador do nome do arquivo (ex.: `fenix-controle-obras`). O nome é gravado nos registros como "criado por" e "atualizado por".
- **`FenixCore.initFirebase(config)`**: alternativa ao `init` que devolve `{ app, db }`. Serve para apps que também precisam do objeto `app` para montar o login. Reaproveita a mesma conexão, nunca cria duas.
- Não é preciso colocar credenciais: o core já traz a configuração do projeto.

### 2.2 Dados fixos da Fênix

- **`FenixCore.EMPRESA`**: nome fantasia, razão social (60292 CRISTIANO RODRIGUES), CNPJ (só números e formatado) e e-mail da empresa. Use em rodapés, documentos e na tela Sobre.
- **`FenixCore.CPF_CNPJ_DISCLOSURE`**: o texto fixo "O CPF/CNPJ é usado apenas para emissão de Notas Fiscais e/ou Recibos." (regra 37). Também está em `FenixCore.pessoas.disclosure`.

### 2.3 Ferramentas de texto e de documento (não usam banco)

| Função | O que faz | Exemplo |
|------------------------------|---------------------------------------------|-------------------------|
| `limparDocumento(valor)` | Deixa só os números. | `"123.456.789-09"` vira `"12345678909"` |
| `tipoDocumento(valor)` | Diz se é CPF (11 dígitos, `"fisica"`) ou CNPJ (14 dígitos, `"juridica"`), ou `null`. | |
| `validarCPF(cpf)` / `validarCNPJ(cnpj)` | Confere os dígitos verificadores, não só o formato (regra 38). Recusa sequências repetidas. | `true` ou `false` |
| `formatarCPF`, `formatarCNPJ`, `formatarDocumento` | Colocam a máscara. `formatarDocumento` escolhe sozinha entre CPF e CNPJ. | `"12345678000199"` vira `"12.345.678/0001-99"` |
| `formatarNomeMaiusculo(texto)` | Nome de pessoa ou empresa: tudo em MAIÚSCULAS, sem espaços sobrando (regra 42). | `"joão da silva"` vira `"JOÃO DA SILVA"` |
| `formatarTitulo(texto)` | Demais textos: primeira letra de cada palavra em maiúscula, menos preposições e artigos. Palavras com número ficam como digitadas. | `"RUA DA PAZ"` vira `"Rua da Paz"` |
| `normalizarBusca(texto)` | Maiúsculas e sem acento, para busca. | `"  João "` vira `"JOAO"` |

### 2.4 Pessoas (cadastro nacional de PF e PJ)

Guarda as pessoas em `pessoas_fisicas` (chave = CPF) e `pessoas_juridicas` (chave = CNPJ). Segue a regra de ouro (regra 41): nenhum app cria cadastro próprio de clientes. **Estas funções usam o banco: só com o banco ativo**, exceto as indicadas.

- **`pessoas.buscar(cpfOuCnpj)`**: acha a pessoa pelo documento (detecta PF ou PJ). Devolve os dados ou `null`.
- **`pessoas.salvarFisica(dados)`**: cria ou atualiza uma PF. Valida o CPF, padroniza a escrita e grava `nomeBusca` para a busca por nome. Campos: `cpf`, `nome`, `telefone`, `email`, `endereco`.
- **`pessoas.salvarJuridica(dados)`**: igual, para PJ. Campos: `cnpj`, `razaoSocial`, `nomeFantasia`, `telefone`, `email`, `endereco`.
- **`pessoas.salvar(dados)`**: escolhe PF ou PJ pelo tamanho do documento.
- **`pessoas.buscarPorNome(termo, tipo, max)`**: autocomplete por início do nome, sem diferenciar maiúsculas e acentos. `tipo` é `"fisica"` (padrão) ou `"juridica"`; `max` padrão 10. Cadastros anteriores à v2.0 só aparecem depois de salvos de novo.
- **`pessoas.buscarNaReceita(cnpj)`** (não usa banco): consulta o CNPJ na BrasilAPI e devolve razão social, nome fantasia, situação cadastral, data de abertura, atividade principal, telefone, e-mail e endereço. Lança erro se o CNPJ for inválido e devolve `null` se não achar.
- **`pessoas.consultarCNPJ(cnpj)`** (não usa banco; é a que a regra 35 manda usar): faz o mesmo que a anterior e, se o CNPJ vier só com o CEP, completa logradouro, bairro, cidade e UF pela BrasilAPI de CEP, sem o usuário redigitar. Só preenche o que faltou. Devolve também `origemEndereco` (`"receita"` ou `"receita+brasilapi-cep"`) e `enderecoCompleto` (verdadeiro se logradouro, bairro, cidade e UF vieram). Use dentro de `try/catch`.

### 2.5 Produtos, serviços e soluções (catálogo comum)

Coleção `produtos_servicos`. **Usa o banco.**

- **`produtos.salvar(dados)`**: cria (ID automático) ou atualiza (se vier `id`). Campos: `nome`, `tipo` (`produto`, `servico` ou `solucao`), `categoria`, `descricao`, `precoBase`, `unidade`, `ativo`. Nome e categoria são padronizados.
- **`produtos.buscar(id)`**: devolve um item ou `null`.
- **`produtos.listar({ tipo, apenasAtivos })`**: lista, podendo filtrar por tipo e só ativos (padrão: só ativos).
- **`produtos.excluir(id)`**: apaga o item. Em tela, use a confirmação por palavra (regra 47).

### 2.6 Localidades (CEP, cidades e bairros) e qual função usar

| Função | Fonte | Usa banco? | Quando usar |
|--------------------------------------|------------|--------------------|------------------------------|
| `localidades.buscarPorCEPSemBanco(cep)` | ViaCEP | Não | Padrão enquanto o banco do app estiver desligado (regra 33). |
| `localidades.buscarPorCEP(cep)` | ViaCEP | Sim: grava cidade e bairro | Só com o banco ativo; vai aprendendo cidades e bairros. |
| `localidades.buscarPorCEPBrasilAPI(cep)` | BrasilAPI | Não | Usada por dentro do `consultarCNPJ`; serve de alternativa. |

Todas devolvem `{ cep, logradouro, bairro, cidade, uf, ... }` ou `null` se o CEP não existir, e lançam erro se o CEP não tiver 8 dígitos. O bairro pode vir vazio em cidade pequena: deixe o campo livre para digitar.

- **`localidades.buscarCidadesPorUF(uf)`**: cidades já conhecidas de um estado, para preencher uma lista de escolha. **Usa o banco.**
- **`localidades.buscarBairros(codigoIBGE)`**: bairros já conhecidos de uma cidade, para autocomplete. **Usa o banco.**
- **`localidades.registrarBairro(codigoIBGE, nomeBairro, origem)`**: guarda um bairro novo. `origem` é `"cep"` (oficial) ou `"manual"` (digitado). **Usa o banco.**

### 2.7 Módulos para apps com várias empresas (usados pelo Fênix Ativos e semelhantes)

Todos usam o banco e recebem o `app` ou o `db` que vem do `initFirebase`.

- **`criarModuloAuth(app)`**: login por e-mail e senha. Devolve `aoMudarEstado(callback)`, `entrar(email, senha)`, `criarConta(email, senha)`, `sair()` e `usuarioAtual()`.
- **`criarModuloEmpresas(db)`**: `criarEmpresa(uid, email, nome)` (cria a empresa e faz o usuário dono), `obterEmpresa(empresaId)`, `obterEmpresaDoUsuario(uid)` e `repararVinculo(uid, dadosParciais)`.
- **`criarModuloConvites(db, baseUrl)`**: convidar técnicos por link. `criarConvite`, `obterConvite`, `aceitarConvite`, `listarConvitesPendentes` e `cancelarConvite`. Cada convite é uma chave aleatória de uso único.
- **`criarModuloEquipe(db)`**: `listarMembros(empresaId)` e `removerMembro(empresaId, uid)`.
- **`criarModuloAppData(db, empresaId, nomeApp)`**: guarda listas inteiras do app na nuvem, um documento por chave. `salvar(chave, lista)` e `carregar(chave)`.
- **`criarModuloCatalogoEmpresa(db, empresaId, nomeColecao, campoChave)`**: cria uma coleção própria do app, isolada dentro da empresa, sem mexer no core. Devolve `listar()`, `salvar(obj)` (cria ou atualiza pela chave) e `excluir(chave)`. Atalhos prontos: `criarModuloMateriais`, `criarModuloProdutosServicos` e `criarModuloFamiliasMateriais`.
- **`criarModuloPessoas(db, empresaId)`**: clientes de um app dentro da empresa (diferente do cadastro nacional da seção 2.4). `listar`, `salvar`, `arquivar(obj, arquivado)` e `excluir`.

### 2.8 Backup (não usa o banco: guarda no aparelho)

- **`backup.verificarLembrete(nomeApp, { dias, aoFazerBackup, aoRecusar })`**: chame uma vez depois de abrir o app. Se passaram mais de 7 dias (ou o número que você informar) sem backup, abre um aviso com "Agora não" e "Fazer agora". Quem exporta os dados é o app, na função `aoFazerBackup`. "Agora não" só volta a perguntar na próxima abertura. Devolve `"nao-necessario"`, `"adiado"`, `"feito"` ou `"erro"`.
- **`backup.baixarJSON(nomeApp, dados)`**: baixa o arquivo `nomeApp-backup-aaaa-mm-dd.json` e registra o backup. Ponha a versão do app dentro de `dados`.
- **`backup.registrar(nomeApp)`**: marca que um backup acabou de ser feito (zera a contagem).
- **`backup.ultimo(nomeApp)`** e **`backup.diasSemBackup(nomeApp)`**: data do último backup e quantos dias se passaram (desde a primeira abertura, se nunca fez).
- O core não tem importação: o app oferece o botão Importar (regra 44).

### 2.9 Tela Sobre e Changelog (não usa o banco: guarda no aparelho)

- **`sobre.registrarAcesso(nomeApp, versao)`**: chame uma vez ao abrir. Conta 1 acesso por abertura (recarregar a página não conta), guarda o primeiro acesso e a data em que a versão atual foi instalada. Devolve `acessos`, `primeiroAcessoTexto`, `ultimaAtualizacaoTexto` e o histórico das últimas 20 versões instaladas.
- **`sobre.obter(nomeApp)`**: lê esses dados sem contar acesso.
- **`sobre.formatarData(iso, comHora)`**: formata como `02/10/2026 14:30`.
- **`sobre.montarTela(container, { nomeApp, titulo, versao, changelog })`**: monta a tela Sobre completa: versão, acessos, primeiro acesso, última atualização, data de criação e de última modificação do app, dados da Fênix e o Changelog com botão de PDF.
- **`sobre.changelogHTML(changelog)`**, **`sobre.ordenarChangelog(changelog)`** e **`sobre.datasDoApp(changelog)`**: peças usadas pela tela (HTML do Changelog, lista ordenada da versão mais nova para a mais antiga, e as datas de criação e modificação).
- **`sobre.baixarChangelogPDF(nomeApp, changelog, opcoes)`**: baixa `nomeApp-changelog-vX.X.pdf`. Usa o jsPDF que o app já embute.
- **Formato do changelog:** lista de `{ versao: "1.1", criadaEm: "2026-10-02", modificadaEm: "2026-10-05", mudancas: ["...", "..."] }`. `modificadaEm` é opcional (vale a data de criação). Datas aparecem como dd/mm/aaaa.

### 2.10 Manual do usuário (não usa o banco)

- **`manual.montarTela(container, md, { nomeApp, versao, titulo })`**: mostra o manual em Markdown, só para ler. Com `nomeApp`, mostra o botão "Baixar manual (PDF)".
- **`manual.baixarPDF(nomeApp, versao, md, opcoes)`**: gera `nomeApp-manual-vX.X.pdf` a partir do mesmo Markdown, com os prints em base64.
- **`manual.paraHTML(md)`**: converte o Markdown em HTML seguro. Aceita títulos (#, ##, ###), parágrafos, listas, citações, código, linha, negrito, itálico, links e imagens (base64 de PNG, JPEG, WebP ou GIF, ou endereço https).

### 2.11 Dashboard clicável (não usa o banco)

- **`dashboard.montar(container, indicadores, opcoes)`**: desenha os cartões de indicador. Cada indicador é `{ id, rotulo, registros (lista ou função), valor, moeda }`. Clicar abre a lista dos registros que formam o número. Indicador zerado não abre e mostra "Nenhum registro". Tem botão Fechar, a tecla Esc fecha e a tela volta à posição anterior. Funciona pelo teclado. Opções: `rotuloRegistro`, `detalheRegistro` e `aoAbrirRegistro` (para ir ao detalhe do registro).
- Devolve um objeto com **`abrir(id)`**, **`fechar()`**, **`abrirLista(titulo, registros)`** (para fatias e barras de gráfico) e **`atualizar(novos)`** (redesenha depois de mudar os dados).

### 2.12 O que funciona com o banco desligado

Todo app nasce com o banco desligado (regra 1). Enquanto `FENIX_BANCO_ATIVO` for `false`, use só:

- **Sem banco, pode usar:** `EMPRESA`, `CPF_CNPJ_DISCLOSURE`, as ferramentas de texto e documento (2.3), `pessoas.buscarNaReceita`, `pessoas.consultarCNPJ`, `localidades.buscarPorCEPSemBanco`, `localidades.buscarPorCEPBrasilAPI`, `backup`, `sobre`, `manual` e `dashboard`.
- **Precisam do banco ativo:** `pessoas.buscar`, `salvar`, `salvarFisica`, `salvarJuridica`, `buscarPorNome`; `produtos.*`; `localidades.buscarPorCEP`, `buscarCidadesPorUF`, `buscarBairros`, `registrarBairro`; e todos os módulos `criarModulo...`.

## PARTE 3. O LIVRO DAS REGRAS, REGRA POR REGRA

### Seção 1. Arquitetura
1. **Core centralizado.** Todo app carrega o core do endereço oficial e nunca guarda cópia dentro de si; assim, uma correção no core vale para todos. O banco vem pré-configurado e desligado (`FENIX_BANCO_ATIVO = false`) e só é ligado quando você der o sinal. O app nasce pronto: ativar é só trocar a chave para `true`.
2. **Carregador oficial.** Código fixo no `<head>` que baixa o core, guarda uma cópia no aparelho e usa essa cópia sem internet. A primeira abertura precisa de internet.
3. **Comportamento offline.** O app nunca trava sem internet: trabalha com dados locais e tenta reconectar a cada 5 minutos e quando a internet voltar.
4. **Indicador de conexão.** Ícone discreto no cabeçalho: online, offline ou sincronizando.
5. **Stack.** HTML, CSS e JS (arquivo único ou vários), PWA quando fizer sentido, Firebase, localStorage, jsPDF embutido, offline-first.
6. **Hospedagem.** GitHub Pages.

### Seção 2. Versionamento e entrega
7. **Sequência de versões.** 1.0, 1.1, ... 1.9, 2.0, sem pular e com uma casa decimal. Livro, core, templates e documentos legais andam juntos com o mesmo número. O salto da v3.0 foi a única exceção.
8. **Toda alteração sobe a versão**, inclusive no arquivo baixável.
9. **Dizer a versão** no texto da resposta de toda entrega.
10. **Nome do arquivo** `fenix-nome-do-app-vX.X.html`, sempre com a cópia neutra.
11. **Duas cópias** de tudo que tem versão: com versão no nome (controle interno) e neutra (referência).
12. **Changelog e Manual obrigatórios**, dentro do app, só para ver. O Changelog tem as datas de criação e modificação de cada versão e PDF para baixar; o Manual é passo a passo, com print e exemplo de cada campo e botão, em MD (no app) e PDF (download). Prints refeitos a cada alteração que mude o uso.
13. **Tela Sobre** com versão, acessos, primeiro acesso, última atualização, datas de criação e modificação e os dados da Fênix.

### Seção 3. Identidade visual
14. **Paleta:** degradê `#040798` para `#0076CE`; marca `#041273`; botões `#0076CE`; destaque, alerta e hover `#ffcb00`; texto de input `#000000`; texto de menu e rodapé `#ffffff`.
15. **Degradê** em menu e rodapé.
16. **Fundo** `#F3F4F9`, cartões brancos com sombra suave, texto `#020733`.
17. **Identidade gráfica:** a águia. Ícones em `Icon/`, logos em `Logo/`, embutidos em base64 no app. Escolha o contraste que melhor combina: Azul (oficial) em fundo claro, Branca (segunda opção) em fundo de cor, e as auxiliares Black, Silver e Gray quando convier. Azul nunca sobre o degradê. App sem ícone próprio, ou com o ícone da letra F, usa a águia como ícone provisório, inclusive no splash.
18. **Layout** com menu lateral ou superior mais cartões; seções colapsáveis sempre fechadas.
19. **Mobile:** barra de abas fixa embaixo, com o ícone central em destaque.
20. **Áreas de toque** de pelo menos 44 × 44 px.
21. **Mensagens em português claro**, sem jargão, dizendo o que houve e o que fazer; estados de carregando, vazio e erro iguais em todos os apps, alertas em amarelo.
22. **Rodapé padrão:** degradê, texto branco, `{ANO} <Cristiano Fênix/> | © Fênix Soluções Tecnológicas`, com ano automático, links do WhatsApp e do Instagram e a versão do app visível.

### Seção 4. Acesso e login
23. **Layout de login único:** use o `fenix-template-login.html`, sem criar outro.
24. **Endereço do template de login** e como versioná-lo.
25. **O que muda em cada app:** só o nome do app e a ligação da autenticação ao core.
26. **Autenticação** por e-mail ou Google, sem PIN.
27. **Termos de Uso e Política de Privacidade** em `Termos/`, com endereço fixo; a versão acompanha o livro e a data só muda quando o texto muda.
28. **Onboarding** no primeiro acesso: primeiro nome, e-mail e CNPJ opcional (nunca CPF). O CNPJ traz razão social e endereço para conferir. Aparece uma única vez.
29. **Template de onboarding** único, com `FENIX_PERFIL_INICIAL`, `salvarPerfil()` e o evento `fenix:onboarding-concluido`.
30. **Nome do usuário** no lado direito, junto ao avatar.
31. **Avatar** com a inicial do primeiro nome (sem foto); menu com nome, Ajuda e Sair.
32. **Splash de abertura em todos os apps:** tela azul com o ícone do app, nome do sistema, "Fênix Soluções Tecnológicas" e barra animada; some quando a tela certa estiver pronta, com teto de 1,2 s.

### Seção 5. Cadastros e dados
33. **Cadastro PF** completo; o CEP preenche o endereço (`buscarPorCEPSemBanco` com o banco desligado, `buscarPorCEP` com ele ativo).
34. **Cadastro PJ** completo, com responsável ou contato.
35. **Fluxo do CNPJ** em qualquer tela que capture CNPJ: use `consultarCNPJ` e liste Razão Social, Endereço, Número, Bairro, CEP, Cidade e UF; tudo continua editável.
36. **Alternância PF/PJ** mantendo os campos comuns.
37. **Aviso de uso do CPF/CNPJ** sempre abaixo dos campos.
38. **Máscaras** de CPF, CNPJ, CEP, celular e fixo enquanto digita, com validação dos dígitos.
39. **Moeda** no padrão `R$ 0.000,00`, na exibição e na digitação.
40. **Datas** em dd/mm/aaaa e horas em 24 h; guardar em formato ISO.
41. **Pessoas só pelo core:** nenhum app cria coleção própria de clientes; guarda a referência e, no máximo, um cache leve.
42. **Padronização de escrita** ao salvar: nomes em MAIÚSCULAS, demais textos com iniciais maiúsculas, e-mail em minúsculas, UF em maiúsculas.
43. **Ficha de produto ou equipamento:** base sempre Marca, Modelo e Série; campos extras por categoria.
44. **Backup:** alerta após mais de 7 dias, arquivo JSON datado, botão Importar no app (que segue a regra 47).
45. **PDF** com cabeçalho padrão: logo Azul e dados da empresa do perfil.

### Seção 6. Segurança e qualidade
46. **Regras do Firestore:** nenhuma coleção aberta; cada usuário só lê e grava os próprios dados; as regras seguem junto com a entrega.
47. **Ações destrutivas** exigem digitar uma palavra (ex.: APAGAR). Nunca o `confirm()` do navegador.
48. **Validar o JS** com `node --check` (retorno 0) e chaves balanceadas.
49. **Testar** em tela de 360 px e sem internet.
50. **Funções dos botões** no HTML declaradas com `var` ou `function`, nunca `const` nem seta.
51. **Evitar regressão:** conferir o escape do Python em template literals e blocos órfãos depois de refatorar.

### Seção 7. Processo
52. **Livro vivo e Ficha de aplicação:** em app novo, ou existente trazido para alteração, preencher a Ficha com o Cristiano antes de começar.
53. **Selo do Livro:** o app declara `fenix-livro` (versão aplicada) e `fenix-ficha` (grupos marcados) no `<head>` e mostra "Padrão: Livro das Regras Fênix vX.Y" na Sobre. Selo igual à versão do livro significa que já está aplicado; selo antigo, aplicar só o que o Histórico indica; sem selo, aplicar tudo.

### Seção 8. Dashboard
54. **Dashboard clicável:** todo indicador com dados abre os registros que o formam, com o mesmo número; zerado mostra "Nenhum registro"; vale também para gráficos e valores em R$. Use `dashboard.montar`.

### Seções finais do livro
- **Ficha de aplicação:** sempre valem as regras 1 a 3, 5 a 22, 39, 40 e 47 a 53. Grupos marcáveis: A banco ativo (4, 46), B login (23 a 32), C PF/PJ (33 a 38, 41, 42), D produto (43), E backup (44), F PDF (45), G dashboard (54).
- **Histórico do livro:** diz o que cada versão muda para os apps e se afeta ou não.
- **Endereços oficiais:** a lista única de endereços; só use os dela.
- **Como entregar:** declarar a versão; nome padrão; selo e Ficha no Changelog; manual MD e PDF; duas cópias; rodar o Painel e tocar em "Testar tudo"; zip com `LEIAME.txt` quando houver mais de um arquivo; avisar se um pedido contrariar o livro.
- **Apps já no padrão:** anotação numerada dos apps que já seguem o livro.

## PARTE 4. CHANGELOG DA BIBLIOTECA

Cada linha diz o que mudou e o que isso exige dos apps. As versões v1.x e v2.x do core vieram antes do livro atual; as versões v3.x são niveladas entre livro, core, templates e documentos legais.

### Versões v3.x (niveladas)
- **v3.6**: entrega com mais de um arquivo leva também um `.zip` com `LEIAME.txt`. Este guia nasce nesta versão. Apps: só na entrega.
- **v3.5**: nova função `localidades.buscarPorCEPSemBanco` (ViaCEP sem gravar no banco). A data dos Termos e da Política só muda quando o texto muda. Entrega termina com o Painel e "Testar tudo". Apps: troque para `buscarPorCEPSemBanco` se consultam CEP com o banco desligado.
- **v3.4**: `index.html` vira o Painel da biblioteca. O template de login fica só em `fenix-template-login.html`. Apps: nada.
- **v3.3**: endereços conferidos: repositório `Fenix-Core` (com hífen), pastas `Icon/`, `Logo/`, `Termos/`, `MD/` e `DOC/`; logos oficial e auxiliares; criado o `Fenix_Icone_128.png`. A v3.2 (Fenix_Core com sublinhado) foi cancelada. Apps: apontar para os endereços novos na próxima alteração.
- **v3.2**: cancelada (endereço com sublinhado que não existe).
- **v3.1**: ícone provisório para app sem ícone ou com o "F"; splash em todos os apps. Apps: aplicar na próxima alteração.
- **v3.0**: nivelamento: livro, core, templates e documentos legais passam a ter o mesmo número; duas cópias de tudo que tem versão. Apps: passam a sair também com a cópia neutra.

### Livro das Regras, antes do nivelamento
- **v2.9**: consulta de CNPJ com endereço completo; ícone provisório; app nasce pronto para o banco. Apps: sim.
- **v2.8**: regras 12, 13 e 54 passam a indicar as funções prontas do core. Apps: não.
- **v2.7**: Manual em MD e PDF com prints; Changelog com datas na tela Sobre; dashboard clicável (regra 54). Apps: sim.
- **v2.6**: banco pré-configurado e desligado em todo app (chave `FENIX_BANCO_ATIVO`). Apps: sim.
- **v2.5**: Ficha de aplicação das regras e Selo do Livro. Apps: sim.
- **v2.4**: seção Endereços oficiais. Apps: não.
- **v2.3**: templates sem versão no nome. Apps: não.
- **v2.2**: removidas as regras do aviso de nova versão e do template de layout com menu. Apps: não.
- **v2.1**: anotação dos apps já no padrão. Apps: não.

### Fênix Core, funções acrescentadas (nada foi removido)
- **v2.5**: `pessoas.consultarCNPJ` e `localidades.buscarPorCEPBrasilAPI`.
- **v2.4**: `sobre.montarTela`, `changelogHTML`, `ordenarChangelog`, `datasDoApp`, `baixarChangelogPDF`; módulo `manual` (`paraHTML`, `montarTela`, `baixarPDF`); `dashboard.montar`.
- **v2.3**: `EMPRESA.razaoSocial` e e-mail corrigido para `trampodofenix@gmail.com`; texto de uso atualizado para o carregador oficial.
- **v2.2**: módulo `sobre` (`registrarAcesso`, `obter`, `formatarData`).
- **v2.1**: `EMPRESA`; módulo `backup`; padronização de escrita ao salvar (`formatarTitulo`, `formatarNomeMaiusculo`).
- **v2.0**: `pessoas.buscarPorNome` corrigido com o campo `nomeBusca`; documentação corrigida.
- **v1.9**: correção grave: o vínculo usuário e empresa passou a usar a coleção `usuarios_empresa`, a mesma das regras do banco.
- **v1.8**: `criarModuloCatalogoEmpresa` e `excluir` no catálogo por empresa.
- **v1.7**: reconstrução dos módulos multiempresa, alinhada ao uso real do Fênix Ativos.
- **v1.3 a v1.6**: módulos de login, empresas, convites, equipe, dados do app, catálogos e pessoas por empresa.
- **v1.2**: `pessoas.buscarNaReceita` (consulta de CNPJ).
- **v1.1**: `localidades.buscarPorCEP` (ViaCEP).
- **v1.0**: módulo base: pessoas, produtos e serviços, localidades.

## PARTE 5. RECEITA RÁPIDA: APP NOVO EM 10 PASSOS

1. Preencha a Ficha de aplicação com o Cristiano (regra 52) e escolha o nome do arquivo `fenix-nome-do-app` (regra 10).
2. Ponha o carregador oficial do core no `<head>` (regra 2) e chame `FenixCore.init("fenix-nome-do-app")`.
3. Crie a chave `FENIX_BANCO_ATIVO = false` e escreva as leituras e gravações do banco atrás dela (regra 1).
4. Monte o login e o onboarding a partir dos templates (regras 23 e 29), o splash (32), o avatar (30 e 31) e o rodapé padrão (22).
5. Use a paleta, o ícone e os logos oficiais; se não houver ícone próprio, use a águia (regras 14 a 17).
6. Cadastros: CPF e CNPJ com máscara e validação, `consultarCNPJ`, CEP com `buscarPorCEPSemBanco` (regras 33 a 38).
7. Moeda e datas no padrão brasileiro (39 e 40); ações destrutivas com palavra de confirmação (47).
8. Backup com `verificarLembrete` e `baixarJSON` (44); Sobre com `registrarAcesso` e `montarTela` (13); dashboard com `dashboard.montar` (54).
9. Escreva o Changelog e o Manual (com prints a 360 px), declare o selo e a ficha no `<head>` (12 e 53) e valide o JS e o app sem internet (48 e 49).
10. Entregue o app nas duas cópias, o manual em MD e PDF e o zip; depois rode o Painel e toque em "Testar tudo".
