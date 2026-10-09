# protesis_helper — protótipo de visualização 3D

Protótipo Flutter para visualizar peças protéticas em 3D no celular.

## Tipos de usuário

- **Professor**: cria casos, peças e importa modelos. Os casos e as peças
  ficam salvos no banco (Firestore) e aparecem para todos os alunos.
- **Padrão** (aluno): só vê a lista de casos para executar, o ranking e o
  perfil.

Toda conta nasce como Padrão. Para virar Professor, o usuário vai em
**Perfil → Solicitar acesso**, escolhe o documento (RG, CNH ou passaporte) e
envia uma foto da frente e outra do verso. As fotos são comprimidas em JPEG
(até 1600 px, cerca de 600 KB) e gravadas em base64 em
`teacherRequests/{uid}/files/front` e `.../back`; o pedido fica em
`teacherRequests/{uid}` com `status: pending`. Só o próprio usuário lê o que
enviou. A aprovação é feita no console do Firebase:

1. Confira o pedido em `teacherRequests/{uid}` e as fotos em `files`.
2. Em `users/{uid}`, mude `role` para `teacher`.
3. Em `teacherRequests/{uid}`, mude `status` para `approved` (ou `rejected`,
   com o motivo em `reviewNote`; o usuário pode então enviar de novo).

O app percebe a mudança de papel na hora e troca a navegação.

## Navegação

Professor: a barra de baixo tem **Dashboard**, **Ranking**, **Perfil** e
**Menu**. O Menu abre o menu lateral com as ferramentas, em dois grupos:
*Peças* (Desenhar, Implante, Modelos) e *Casos* (Arcada, Montagem, Executar).

Padrão: a barra tem **Dashboard**, **Casos**, **Ranking** e **Perfil**, sem
menu de ferramentas.

- **Dashboard**: saudação, pontos, montagens concluídas, posição no ranking,
  casos (os seus, para o professor; os disponíveis, para o aluno) e as
  últimas montagens concluídas. O professor vê também o caso em edição e
  atalhos para as ferramentas.
- **Ranking**: pódio e lista por pontos, com você destacado. Pontos vêm das
  montagens concluídas na aba Executar: 10 por peça (15 se o caso exige
  acertar o ângulo), menos 2 por tentativa de pegar peça fora da ordem, e
  nunca menos que a metade. Testar o caso em edição não conta. O botão ⓘ
  explica as regras.
- **Perfil**: foto, nome, e-mail, tipo de usuário, método de login, pontos,
  montagens e peças encaixadas; dados do cadastro (nascimento, telefone,
  instituição, curso, cidade), com edição; pedido de acesso de professor (para o
  usuário Padrão); editar nome, reenviar a confirmação de e-mail, redefinir
  senha e sair.

Sem login (testes, ou "continuar sem login" em modo debug), os pontos ficam
só na memória e o ranking mostra só você.

## Ferramentas

- **Implante**: implante com rosca helicoidal na própria malha ou pilar tipo
  munhão. Sliders de diâmetro, comprimento, conicidade, passo e profundidade da
  rosca e colar (implante); plataforma, altura, conicidade e transmucoso (pilar).
- **Desenhar**: peças desenhadas com o dedo, em cinco tipos:
  - *Revolução*: meio contorno a partir do eixo, girado 360° (cicatrizador,
    implante próprio), com rosca opcional nos trechos verticais.
  - *Barra*: trajeto varrido por uma seção meia-pera, meia-cana, redonda ou
    plana (barra lingual/palatina).
  - *Grampo*: um traço por braço, afinando do início até a ponta.
  - *Placa*: cada contorno fechado vira uma placa com espessura, que pode ser
    curvada (placa lingual em segmentos).
  - *Grade*: contorno com moldura e treliça em X (grade de retenção), com
    largura das hastes, espaçamento, ângulo e curvatura ajustáveis.

  O traço vira pontos de controle. No modo *Editar*, arraste um ponto para
  movê-lo, toque na linha para inserir um ponto e segure sobre um ponto para
  apagá-lo. Tem *Desfazer*, zoom da área de desenho e um *Exemplo* de cada tipo.
- **Modelos**: abre `.stl` (binário ou ASCII), `.obj`, `.glb` e `.gltf` com
  buffers embutidos. No STL as arestas vivas continuam nítidas. Malhas acima de
  60 mil triângulos são simplificadas para exibição. Tem uma coroa de exemplo.
  O modelo aberto pode ir para o caso ou para a biblioteca (é reduzido a 12
  mil triângulos e guardado compactado dentro do caso).

Nas abas Implante, Desenhar e Modelos, o botão de marcador (Salvar na
biblioteca) guarda a peça atual com um nome na **biblioteca de peças**, no
banco. Na Montagem, "Adicionar peça" lista as peças abertas nas abas e a
biblioteca, separada por categoria; cada professor pode apagar as próprias
peças da biblioteca.

- **Arcada**: modelo completo da arcada superior ou inferior (uma de cada
  vez), com os 16 dentes numerados no padrão FDI, gengiva nas laterais e o
  meio preenchido (palato na superior, assoalho na inferior), sobre uma base
  plana. As duas aparecem com a base embaixo e os dentes para cima; a superior
  fica espelhada no plano oclusal, como no espelho oclusal, e o STL exportado
  volta à posição anatômica. Toque num dente no 3D ou no
  odontograma para selecioná-lo; os selecionados podem ser ocultados e
  mostrados de novo (a gengiva continua no lugar, como espaço desdentado).
  Tem vistas prontas (frente, lado, oclusal), opção de esconder a gengiva e
  exportação em STL só do que está visível. As coroas são procedurais, com
  tamanhos médios por tipo de dente: ilustrativas, não a anatomia de um
  paciente.

- **Montagem**: monta o caso com as peças criadas. Nas abas Implante e
  Desenhar, o botão ➕ (Adicionar ao caso) envia a peça atual como próximo
  passo; também dá para usar "Adicionar peça" na própria montagem. Cada peça
  entra já encaixada num dente (um pilar vai sobre o último implante; implante
  e pilar marcam o dente como ausente). Para posicionar: toque na peça para
  selecioná-la e toque num dente (no 3D ou no odontograma) para encaixá-la;
  no modo "Mover", a peça selecionada ganha setas de eixo: arraste X
  (vermelha, lateral), Y (verde, altura) ou Z (azul, frente/trás) para mover
  só naquele eixo. No modo "Girar", ganha anéis de rotação: Y (verde) gira em
  torno da vertical, X (vermelho) inclina no sentido vestíbulo-lingual e Z
  (azul) no mésio-distal; a peça gira em torno do próprio centro. Fora das
  setas e dos anéis, arrastar gira a vista. Os sliders fazem o ajuste fino de
  posição e rotação. A ordem dos passos é a ordem de
  montagem: arraste pela alça para reordenar e use "Ver a montagem até o
  passo" para conferir a sequência. "Base transparente" mostra implantes
  dentro da gengiva.

  O caso em edição fica salvo no aparelho automaticamente (um rascunho por
  conta). "Salvar caso" (ou "Finalizar e salvar caso") publica o caso com
  nome no banco, onde os alunos podem executá-lo; em "Casos salvos" o
  professor reabre ou apaga os próprios casos. O caso também pode ser
  exportado e importado como arquivo `.caso.json`, e a montagem exportada em
  STL. O caso guarda a receita de cada peça (parâmetros e pontos de
  controle); modelos importados vão com a malha compactada.

- **Executar** (ou **Casos**, para o aluno): lista os casos publicados pelos
  professores; quem executa só escolhe um caso e monta. As peças aparecem em
  miniatura 3D no menu lateral, separadas por categoria (Implantes, Pilares,
  Peças de revolução, Barras, Grampos, Placas, Grades e Modelos importados),
  e o usuário escolhe a peça pela miniatura; o menu abre sozinho ao escolher
  o caso e depois de cada encaixe. Tocar na peça da vez
  a traz para o centro da tela, já com a rotação certa; o usuário a leva até
  o lugar arrastando a própria peça (desliza sobre a arcada) ou as setas
  X, Y, Z. Solta dentro da margem de erro, ela se encaixa na posição exata, o
  passo fica marcado com ✓ e o próximo é liberado. Tocar numa peça fora de
  ordem avisa qual é a vez. Base transparente, sombra do alvo, margem de
  erro e "Destacar a próxima peça" são definidas por quem monta o caso
  (seção "Na execução" da aba Montagem) e ficam salvas nele. Sem o destaque,
  o menu não indica qual peça é a da vez: o aluno precisa reconhecê-la pela
  miniatura, e cada escolha errada conta como fora de ordem. Com "Exigir rotação", a peça chega girada e
  é preciso acertar também o ângulo (margem em graus), usando os anéis do
  modo Girar. A execução trabalha numa cópia: não altera
  o caso. O professor tem também um atalho para testar o caso em edição na
  Montagem (não conta pontos).

Em todas as abas, o botão de paleta escolhe o material (titânio usinado,
jateado e anodizado, cromo-cobalto, ouro, zircônia, dissilicato, esmalte, PMMA
e gengiva), o tom (escala tipo VITA) e a textura: grade UV ou imagem própria,
com repetição e mapeamento (UV da peça, caixa ou cilíndrico). O botão de
download exporta a peça em STL (mm).

As cores são aproximações visuais, não servem para seleção clínica. As formas
de implante e pilar são ilustrativas: para peças reais, use o STL da biblioteca
CAD do fabricante.

## Login (Firebase)

Ao abrir, o app pede login com e-mail e senha (entrar, criar conta, esqueci
a senha) ou com o Google (web, Android e iOS; no desktop o Firebase não
oferece). A cada entrada, o documento `users/{uid}` do Firestore é criado ou
atualizado com `uid`, `name`, `email`, `photoUrl`, `emailVerified`,
`providers`, `createdAt` (criação da conta) e `lastLoginAt` (último acesso).
Na criação ele recebe também `role: student`; o app nunca altera o papel.

O cadastro pede também **data de nascimento** (dd/mm/aaaa ou calendário),
**telefone** (com DDD), **instituição de ensino** e **curso** (Odontologia,
Técnico em Prótese Dentária, Técnico ou Auxiliar em Saúde Bucal, Outro); o
período, a cidade e a UF são opcionais. Eles vão para o mesmo `users/{uid}`
(`birthDate` como `AAAA-MM-DD`, `phone` só com dígitos, `institution`,
`course`, `period`, `city`, `state`). Quem entra sem esses dados (login com o
Google ou conta antiga) passa por uma tela "Complete seu cadastro" antes do
app. No Perfil, o card "Dados do cadastro" mostra e edita tudo.
Ao criar conta, o app envia o e-mail de confirmação. A conta fica na aba
Perfil.

Pontos e ranking também ficam no Firestore:
- `users/{uid}/executions/{id}`: cada montagem concluída (caso, peças,
  tentativas fora de ordem, tempo, pontos, data); só o dono lê.
- `ranking/{uid}`: nome, foto e totais (pontos, montagens, peças); todo
  usuário logado lê, para montar o ranking.

Casos e peças:
- `cases/{id}`: resumo (nome, arcada, número de peças, autor, data);
  `cases/{id}/content/main` guarda o caso completo. Todo usuário logado lê.
- `pieces/{id}` e `pieces/{id}/content/main`: biblioteca de peças, no mesmo
  formato.

As regras em `firestore.rules` deixam cada usuário gravar só os próprios
dados; só professores (`users/{uid}.role == 'teacher'`) criam casos e peças,
e cada um altera ou apaga só os seus; ninguém muda o próprio papel; o pedido
de professor e as fotos do documento só são lidos pelo dono; e os pontos só
sobem (no máximo 500 por montagem). Como é o próprio
app que grava os pontos, alguém mal-intencionado ainda poderia inflá-los
dentro desse limite; para um ranking à prova de fraude, a pontuação teria de
ser calculada no servidor (Cloud Functions).

Enquanto o Firebase não estiver configurado, o app mostra uma tela com os
passos (em modo debug dá para continuar sem login).

### Configurar o Firebase

1. Entre no Firebase CLI e instale o FlutterFire CLI:

   ```bash
   firebase login --reauth
   dart pub global activate flutterfire_cli
   ```

2. Na pasta do projeto, gere `lib/firebase_options.dart` (escolha ou crie o
   projeto e marque android, ios, web e, se quiser, windows):

   ```bash
   flutterfire configure
   ```

3. No [console do Firebase](https://console.firebase.google.com):
   - *Authentication → Sign-in method*: ative **E-mail/senha** e **Google**.
   - *Firestore Database*: crie o banco.
   - *Authentication → Settings → Authorized domains*: para a web, confira
     se o domínio onde o app roda está na lista (`localhost` já vem).
4. Publique as regras do Firestore:

   ```bash
   firebase deploy --only firestore:rules --project SEU_PROJETO
   ```

5. Android, para o Google: cadastre o SHA-1 e o SHA-256 do app nas
   configurações do projeto (app Android) e rode `flutterfire configure` de
   novo. Para ver as chaves: `cd android` e `./gradlew signingReport`.
6. iOS, para o Google: adicione o `REVERSED_CLIENT_ID` do
   `GoogleService-Info.plist` como URL Scheme no `Info.plist`.

## Visual

As cores e o estilo dos componentes (botões, abas, campos, chips, cards,
menus) ficam em [lib/app/theme/](lib/app/theme/) (`app_theme.dart` e
`app_color_schemes.dart`):
paleta em vermelho suave (rosé queimado) com superfícies em branco quente,
nos modos claro e escuro. O verde de "encaixado" na execução também é suave
e vem do tema (`StatusColors`).

## Arquitetura

O código segue Clean Architecture, organizado por funcionalidade:

```
lib/
  main.dart            inicia o Firebase e o app
  app/                 composição do app
    di/                monta repositórios, casos de uso e controllers
    navigation/        barra de baixo, menu lateral e seções
    theme/             tema, paleta e estilos dos componentes
  core/                código compartilhado, sem regra de negócio
    geometry/          malhas, revolução, varredura, placas
    io/                STL, OBJ, glTF, exportar e escolher arquivos
    rendering/         câmera, projeção, iluminação, renderizador, gizmos
    theme/             extensões de tema (cores de status)
    utils/             formatação, ids, leitura de JSON
    widgets/           componentes reutilizáveis (avatar, banners,
                       métricas, diálogos, sliders, barra de ferramentas...)
  features/<nome>/
    domain/            entidades, contratos de repositório, casos de uso
                       e serviços de domínio (Dart puro, sem Flutter)
    data/              implementações (Firebase, Firestore,
                       SharedPreferences, memória) e conversões
    presentation/      controllers (ChangeNotifier), páginas e widgets
```

Funcionalidades: `auth` (login e papel do usuário), `teacher_access`
(pedido de acesso de professor), `profile` (perfil), `progress` (pontos e
ranking), `dashboard`, `cases` (caso, montagem e casos no banco),
`piece_library` (biblioteca de peças), `execution` (executar o caso),
`pieces` (implante e desenho), `dental_arch` (arcada) e `model_import`
(abrir modelos 3D).

Regras seguidas:
- O domínio não depende de Flutter, Firebase nem das outras camadas; as
  páginas dependem de casos de uso e controllers, nunca dos repositórios
  concretos (inversão de dependência). Só `app/di` conhece Firebase,
  Firestore e SharedPreferences.
- Cada caso de uso faz uma coisa (`SignInWithEmail`, `RecordExecution`,
  `WatchRanking`...), e a regra de pontuação fica em `ScoringPolicy`.
- As telas são compostas de widgets pequenos; o estado e a lógica de cada
  tela ficam no controller dela (ex.: `SketchController`,
  `ArchViewController`, `ExecutionController` sobre `ExecutionSession`).
- O código não tem comentários: nomes de classes, métodos e constantes
  explicam o que fazem.

## Como rodar

```bash
flutter pub get
flutter run --release
```

Funciona com o Flutter estável atual (testado no 3.32), em Android, iOS, web e
desktop. Não precisa de Flutter GPU nem de flags extras. Use `--release` ou
`--profile` no celular: em modo debug a rotação fica bem mais lenta.

## Como funciona o 3D

O renderizador ([lib/core/rendering/](lib/core/rendering/)) é
escrito em Dart puro sobre `Canvas.drawVertices`. A cada quadro ele projeta os
vértices, calcula a luz por vértice (difusa, especular com Fresnel, reflexo de
estúdio, verniz e translucidez), descarta as faces de trás e ordena os
triângulos de trás para frente. Para peças de até algumas dezenas de milhares
de triângulos isso roda fluido no celular.

Se um dia precisar de cenas maiores, sombras ou PBR por pixel, o caminho é
migrar para o `flutter_scene` (exige Flutter 3.47+ com Flutter GPU). A geometria
e os leitores de arquivo (`lib/core/geometry`, `lib/core/io`) independem do
renderizador.

## Testes

```bash
flutter test
```

Cobrem malhas fechadas e orientadas (implante, pilar, desenho, coroa, barra,
grampo, placa, grade e arcadas), curvas pelos pontos de controle, leitura e
escrita de STL, OBJ e GLB, simplificação, arestas vivas, descarte de faces de
trás, seleção por toque, a montagem do caso (receitas, encaixe no dente,
ordem dos passos, salvar/abrir/exportar, casos por autor), a execução e o
menu de peças por categoria, a rotação, a compactação das malhas, a tela de
login e os dados gravados em `users` (com um login de mentira, sem Firebase),
os papéis (navegação de professor e de aluno), o pedido de acesso de
professor (validação, compressão das fotos, estados no perfil) e a navegação
entre as abas.
