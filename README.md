# Miniguia de estudos: modularização iOS com Swift Package Manager

Caderno temático criado com **NotebookLM** como ferramenta de aprendizagem ativa: curadoria de fontes, engenharia de prompts, registro de falhas (“cicatrizes”) e consolidação do conhecimento.

> Desafio de projeto — DIO  
> Objetivo da plataforma: aliar pensamento crítico, curadoria e organização do conhecimento em um repositório público.

---

## 1. Contexto e objetivos

### Assunto
Como **organizar um app iOS em módulos** usando **Swift Package Manager (SPM)** — pacotes locais, manifesto `Package.swift`, produtos, targets e o caminho até reuso entre apps — com base em documentação oficial da Apple e do Swift.org.

Escolhi este tema porque modularização e SPM são decisões de arquitetura em apps de grande escala: fronteiras de código mais claras, builds mais previsíveis, testes por módulo e evolução independente de features.

### Objetivos de estudo
- Distinguir **package, product, target e dependency**.
- Saber quando usar **pacote local** vs **pacote publicado** (repositório Git próprio).
- Entender o papel do **`Package.swift`** e da estrutura de pastas do SPM.
- Relacionar modularização com **manutenção, reuso e qualidade** (não só “separar pastas”).
- Sair com um **miniguia + prompts reutilizáveis** para revisar o tema sem recomeçar do zero.

### Como usei o NotebookLM
1. Criei um notebook só com as fontes da seção 2 (sem misturar blogs aleatórios).
2. Fiz perguntas de conceituação, depois de procedimento, depois de comparação.
3. Quando a resposta veio genérica ou misturou CocoaPods/Carthage, refinei o prompt e registrei a cicatriz.
4. Consolidei resumo, glossário e prompts finais neste README.

---

## 2. Curadoria de fontes

Critério: **fontes abertas e oficiais** (Apple / Swift.org), em texto, adequadas para upload no NotebookLM (copiar a página ou exportar PDF).

| # | Fonte | Por que entrou |
|---|--------|----------------|
| 1 | [Introducing Packages (Swift.org)](https://docs.swift.org/latest/documentation/packagemanagerdocs/introducingpackages/) | Vocabulário canônico: package, products, targets, modules, dependências. |
| 2 | [Getting Started — Package Manager (Swift.org)](https://docs.swift.org/latest/documentation/packagemanagerdocs/gettingstarted/) | Fluxo prático: `swift package init`, library, dependência, testes. |
| 3 | [Organizing your code with local packages (Apple)](https://developer.apple.com/documentation/xcode/organizing-your-code-with-local-packages) | Modularizar o **app** com pacotes locais no mesmo repositório. |
| 4 | [Creating a standalone Swift package with Xcode (Apple)](https://developer.apple.com/documentation/xcode/creating-a-standalone-swift-package-with-xcode) | Manifesto, produtos library/executable, schemes e testes no Xcode. |
| 5 | [Creating Swift Packages — WWDC19 (Apple)](https://developer.apple.com/videos/play/wwdc2019/410/) | Narrativa de refatorar código compartilhado (iOS/watchOS) para pacote local e publicar depois. |

**O que ficou de fora (de propósito):** tutoriais de CocoaPods, comparativos antigos SPM vs Carthage e posts sem data. O caderno precisa de uma base estável para a IA não misturar gerenciadores.

**Como subir no NotebookLM:** abra cada URL → salve PDF (imprimir → salvar como PDF) ou cole o texto em um `.md`/`.txt` → **Add source**.

---

## 3. Engenharia de prompts e cicatrizes

Regra: **só usar o conteúdo das fontes**. Se a IA inventar, o prompt falhou.

### Rodada 1 — conceituação (fraca → melhor)

**Prompt v1 (fraco)**  
> Explique Swift Package Manager.

**O que aconteceu:** resposta ampla, misturou “gerenciador de dependências” com história do SPM e pouco sobre *local packages* no Xcode. Pouca âncora nas fontes 3 e 4.

**Prompt v2 (melhor)**  
> Com base apenas nas fontes deste notebook, explique o que é um Swift package e a relação entre Package, Product, Target e Dependency. Use uma analogia curta e cite de qual fonte veio cada ideia. Não fale de CocoaPods nem Carthage.

**Aprendizado:** restringir o vocabulário e pedir **citação da fonte** reduz alucinação.

### Rodada 2 — procedimento no Xcode

**Prompt v1**  
> Como modularizar um app iOS?

**Cicatriz:** a IA descreveu pastas no projeto (`Networking/`, `Features/`) **sem** criar pacote SPM. Tecnicamente “modular”, mas **fora do recorte das fontes**.

**Prompt v2**  
> Siga apenas a documentação da Apple sobre local packages. Liste o passo a passo para: (1) identificar código candidato; (2) criar um pacote local no Xcode; (3) mover o código; (4) ajustar o Package.swift se necessário; (5) adicionar o library product no target do app em Frameworks, Libraries, and Embedded Content. Numere os passos. Se algo não estiver nas fontes, diga “não encontrado nas fontes”.

**Resultado esperado (síntese das fontes 3 e 4):** File > New > Package → template Library → mover código → manifesto (product + targets) → ligar o produto no target do app. Pacote local vive no **mesmo Git do app**; para reuso entre apps, extrair para outro repositório e adicionar como package dependency.

### Rodada 3 — “quando NÃO modularizar”

**Prompt**  
> As fontes dizem para criar o máximo de módulos possível. Extraia a recomendação exata e os riscos que *não* estão escritos (marque como inferência minha, não da fonte).

**Cicatriz:** o Swift.org sugere que “more modules are probably better than fewer”, no contexto do *package manager*. Aplicar isso cegamente a um app bancário (dezenas de pacotes, grafo cíclico, tempo de CI) **não está** nas fontes.  
**Troubleshooting:** pedi para separar **citação** vs **inferência**. Isso é o que o desafio chama de pensamento crítico.

### Rodada 4 — manifesto vs projeto Xcode

**Prompt**  
> Compare Package.swift com um .xcodeproj: o que o pacote usa no lugar do projeto? O que o Xcode gera automaticamente (schemes, testes)?

**Ganho:** a fonte 4 deixa claro que pacotes **não** usam `.xcodeproj`/`.xcworkspace` como fonte da verdade: pasta + manifesto. O Xcode cria scheme por product e um scheme `[nome]-Package` quando há vários produtos.

### Rodada 5 — troubleshooting de extração

| Problema | Causa | Ajuste de prompt |
|----------|--------|------------------|
| Resposta “Wikipedia” do SPM | Prompt aberto demais | “Apenas estas fontes” + “cite a fonte” |
| Mistura CocoaPods | Treino genérico do modelo | “Proibido mencionar outros gerenciadores” |
| Passos de UI desatualizados (Xcode 14 vs 15+) | Fonte fala “Swift Package” em versões antigas | Pedir para **transcrever o nome do menu da fonte** e não inventar o Xcode 16 |
| Resumo sem glossário | Pedi “explique tudo” | Separar tarefas: resumo / glossário / cheatsheet |
| Citação inventada | Pedi “com citações” sem trecho | “Para cada afirmação, indique o título da fonte; se não achar, diga que não achou” |

### Perguntas estratégicas que funcionaram
1. Qual a diferença entre *product* e *target*?
2. Quando um pacote local deve virar repositório separado?
3. O que é um bom candidato a módulo (rede, utilitários) segundo a Apple?
4. Como o SPM resolve versões (ideia geral da WWDC / Swift.org)?
5. Como testes se encaixam (`testTarget` / schemes)?

---

## 4. Miniguia de estudo (entrega final)

### 4.1 Resumos estruturados

#### O que é um Swift package
Um package é um diretório com **`Package.swift`** (manifesto) + código (e opcionalmente recursos). O manifesto, via `PackageDescription`, declara nome, **products**, **targets** e **dependencies**.

#### Product vs Target vs Module
- **Target:** bloco de build (biblioteca, testes, executável…). Pode depender de outros targets do mesmo package e de products de outros packages.
- **Product:** o que o package **exporta** (library, executable, plugin) para apps e outros packages.
- **Module:** unidade de distribuição/namespace; na prática, um target de código Swift vira um módulo.

Regra de ouro do Swift.org: o gerenciador existe para **baratear** criar vários módulos — mas “mais módulos” não é desculpa para grafo bagunçado no app.

#### Pacote local no app (Xcode)
1. Identificar código extraível (rede, utilitários, modelo compartilhado).
2. File > New > Package (template Library / Multiplatform).
3. Mover arquivos e ajustar o manifesto (library product + targets).
4. No target do app: **Frameworks, Libraries, and Embedded Content** → adicionar o product do pacote.
5. O código do pacote local permanece no **mesmo repositório** do app.

#### Pacote standalone e reuso
Pacote independente: pasta + manifesto, sem `.xcodeproj` como definição. Dá para abrir o `Package.swift` no Xcode, buildar pelo scheme do product e ter `testTarget`.  
Quando o mesmo módulo servir a **vários apps**, a Apple sugere promover o pacote local a **repositório Git** e consumir como *package dependency*.

#### Dependências
Declarar o package **e** ligar o product no target que vai `import`. Resolver versões é trabalho do SPM (requisitos compatíveis → `Package.resolved` no fluxo clássico). Builds isolados e manifesto em Swift são parte do desenho do SwiftPM.

#### Ponte para o dia a dia (inferência, não fonte)
Em app grande, SPM local ajuda a: limitar `import` entre features, testar módulo isolado, e preparar extração de um kit interno. Não resolve sozinho CI lento, acoplamento de domínio nem WebView/legado — isso é desenho de fronteiras.

---

### 4.2 Glossário

| Termo | Significado curto |
|--------|-------------------|
| **Swift Package Manager (SPM / SwiftPM)** | Ferramenta para criar, resolver e buildar packages Swift. |
| **Package.swift** | Manifesto do package (nome, products, targets, dependencies). |
| **Product** | Artefato visível para quem depende do package (ex.: library). |
| **Target** | Unidade compilável (código, testes, executable). |
| **Library product** | Código para `import` em app ou outro package. |
| **Executable product** | Binário de linha de comando. |
| **Local package** | Package dentro do projeto/repositório do app, desenvolvido em conjunto. |
| **Package dependency** | Package remoto (Git) ou publicado, versionado, reutilizado em vários apps. |
| **Module** | Namespace / unidade de distribuição; targets de código viram módulos. |
| **`swift package init`** | Scaffold de library ou executable. |
| **testTarget** | Target de testes do package. |
| **Scheme `[nome]-Package`** | No Xcode, scheme extra para buildar/testar todos os products quando há vários. |
| **Dependency resolution** | Escolher versões compatíveis de todos os packages do grafo. |
| **Modularidade (Apple)** | Organizar código para manutenção, reuso e fronteiras claras — pacotes locais são o mecanismo oficial no Xcode. |

---

### 4.3 Prompts reutilizáveis (revisão futura)

Use no mesmo notebook (ou em um novo, com as mesmas fontes).

**Revisão em 10 minutos**  
> Faça um resumo em 12 bullets só com o que está nas fontes: package, product, target, local package, quando publicar. Uma linha de “não coberto pelas fontes” no final.

**Simulado de entrevista**  
> Me faça 8 perguntas de entrevista iOS sênior sobre SPM e pacotes locais. Depois da minha resposta, corrija usando apenas as fontes. Se eu inventar CocoaPods, interrompa.

**Checagem de alucinação**  
> Para cada afirmação do seu último resumo, marque: [FONTE: título] ou [NÃO ESTÁ NAS FONTES].

**Procedimento Xcode**  
> Recoloque o passo a passo de criar um local package e ligar no app, só com a doc da Apple. Sem atalhos de teclado inventados.

**Decisão de arquitetura**  
> Com base nas fontes, quando um pacote local deve virar repositório separado? Responda em 5 linhas. O que for opinião, rotule como opinião.

**Glossário relâmpago**  
> Defina em uma frase: Product, Target, Module, Local package, Package.resolved (se não estiver nas fontes, diga que não está).

**Mapa mental**  
> Estruture: Conceitos → Passos no Xcode → Publicar/reuso → Testes. Sem introdução.

---

## 5. Como reproduzir este caderno

1. Criar notebook no [NotebookLM](https://notebooklm.google.com).
2. Adicionar as 5 fontes da seção 2.
3. Rodar as rodadas da seção 3 e os prompts da 4.3.
4. Comparar a saída da IA com este README — atualizar se a doc da Apple mudar nomes de menu.

---

## Licença e uso

Material de estudo pessoal para o desafio DIO. As fontes pertencem à Apple e ao Swift.org; aqui há apenas curadoria, síntese e prompts.
