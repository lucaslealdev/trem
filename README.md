# O Manifesto Trem 🚂

> *"Passa o trem aí."*
> — Todo mineiro, sobre absolutamente qualquer coisa, desde sempre

---

## Preâmbulo

Faz vinte anos que a indústria de software vive uma guerra besta.

De um lado, os **relacionais**. Eles têm tipos, constraints e joins, mas cada mudança de requisito vira uma migration, um code review e uma reunião com o DBA, que já está de férias.

Do outro lado, os **NoSQL**. Eles têm flexibilidade, só que ninguém sabe o que tem dentro do documento. O campo `nome` virou `name` em 2019, `nomeCompleto` em 2021 e `NOME` numa sexta-feira que ninguém quer lembrar.

Cada lado ficou com metade da verdade. Enquanto isso, lá nas montanhas, tomando um café coado com pão de queijo, Minas Gerais já tinha resolvido o problema faz trezentos anos. Só não contou pra ninguém, porque mineiro não faz alarde.

O mineiro nunca precisou de mais de uma palavra pra classificar o universo. Cadeira é trem. Carro é trem. Sentimento é trem. Documento fiscal é trem. O próprio trem é trem. **O mineiro inventou o tipo universal antes da ciência da computação existir.**

Este manifesto só formaliza o que Minas sempre soube.

---

## I. A Metodologia Trem

A Metodologia Trem tem duas regras. As duas são inegociáveis.

### Regra 1: No banco de dados existe uma única tabela, e ela se chama `trem`.

```sql
CREATE TABLE trem (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(), -- ninguém precisa informar; o trem se identifica sozinho
  trem_id      uuid REFERENCES trem(id),                   -- o trem de onde esse trem veio
  origem_id    uuid REFERENCES trem(id),                   -- estação de partida
  destino_id   uuid REFERENCES trem(id),                   -- estação de chegada
  nome         text,
  email        text,
  preco_em_centavos bigint,
  quantidade   integer,
  pago         boolean,
  embedding    vector(1536),
  criado_em    timestamptz DEFAULT now(),
  deletado_em  timestamptz
  -- ... e quantas colunas mais o trem precisar. Todas opcionais.
);
```

### Regra 2: Na aplicação existe um único tipo, e ele se chama `Trem`.

```ts
export type Trem = {
  id?: string;
  trem_id?: string;
  origem_id?: string;
  destino_id?: string;
  nome?: string;
  email?: string;
  preco_em_centavos?: number;
  quantidade?: number;
  pago?: boolean;
  embedding?: number[];
  criado_em?: Date;
  deletado_em?: Date;
  // ... espelho fiel da tabela trem. Todos opcionais.
};
```

O mesmo tipo vale no back-end, no front-end, no app mobile, na fila de mensagens, no teste e no PowerPoint da diretoria. Da primeira linha do banco até o último pixel da tela, **é trem de ponta a ponta**.

---

## II. Os Valores

No espírito do Manifesto Ágil, e com o devido respeito a ele (que, reparem, também é um manifesto só), declaramos que valorizamos:

- **Um trem** mais que muitas tabelas
- **Colunas opcionais** mais que migrations obrigatórias
- **Joins de trem com trem** mais que joins de coisa com outra coisa
- **Tipagem forte e humilde** mais que `any` preguiçoso ou schema engessado
- **Velocidade de Kombi tunada com neon** mais que qualquer outra velocidade conhecida

Ou seja, mesmo reconhecendo valor nos itens à direita, a gente fica com o trem.

---

## III. Fundamentação Técnica

Pode parecer que o Manifesto Trem é brincadeira. Leia os argumentos abaixo e depois reflita bem sobre o fato de que ele continua sendo brincadeira.

### 1. O melhor dos dois mundos: SQL e NoSQL ao mesmo tempo

A tabela `trem` é **NoSQL** porque aceita qualquer formato de dado. Um trem pode ter só `email`, outro só `preco_em_centavos`, outro os dois, e outro nenhum. Ninguém é obrigado a nada. É a liberdade do documento.

A tabela `trem` também é **SQL** porque toda coluna tem tipo de verdade. `preco_em_centavos` é `bigint` e ponto final. Ninguém enfia a string `"dez reais"` ali dentro. E, mais importante, **dá pra fazer join**:

```sql
-- Os pedidos de um cliente
SELECT pedido.*
FROM trem AS cliente
JOIN trem AS pedido ON pedido.trem_id = cliente.id
WHERE cliente.email = 'ze@uai.com.br'
  AND pedido.preco_em_centavos IS NOT NULL;
```

Repare que em momento nenhum dissemos o que era um cliente e o que era um pedido. O pedido é pedido **porque tem preço**. É *duck typing* no nível do banco: se tem preço e tem quantidade, é pedido. O trem é definido pelo que ele carrega, e não por um rótulo imposto de fora. Isso é bonito demais.

### 2. A indústria já chegou lá, só que pela metade

O Manifesto Trem não surgiu do nada. Ele é a **conclusão lógica** de caminhos que a engenharia de software já andou:

- **Google Bigtable, HBase e Cassandra** se descrevem como tabelas *esparsas*, com uma quantidade enorme de colunas quase sempre vazias. Estruturalmente, isso é uma tabela trem que ainda não aceitou Jesus. Quer dizer, que ainda não aceitou o trem.
- **A AWS recomenda *single-table design* no DynamoDB.** Isso mesmo, a Amazon recomenda uma tabela só. Só esqueceram de chamar a tabela de `trem`.
- **O Rails tem *Single Table Inheritance*** faz quase vinte anos: várias entidades numa tabela só, com colunas que ficam nulas quando não se aplicam. Estavam a um `rename` de distância.
- **O Protocol Buffers 3, do Google, removeu os campos `required`.** Depois de anos de sofrimento, o Google concluiu que campo obrigatório é pra sempre, e pra sempre é muito tempo. Hoje todo campo é opcional.
- **No GraphQL, do Facebook, os campos são *nullable por padrão*.** O motivo é resiliência: o sistema tem que aguentar dado faltando.

Google, Amazon, Facebook e Rails chegaram cada um a um pedaço da verdade. **Minas Gerais chegou nela inteira**, e com café.

### 3. Toda coluna opcional é honestidade

No mundo real, **todo dado pode faltar**. O cliente não informou o telefone. O fornecedor não mandou o CNPJ. O endereço veio sem número porque "é a casa amarela depois da venda do Seu Tião".

Um schema com `NOT NULL` é uma promessa que o mundo não vai cumprir. O tipo `Trem`, com tudo opcional, **obriga o desenvolvedor a encarar a realidade**. O TypeScript não deixa ninguém usar `trem.email` sem antes conferir se o email existe. Isso é null safety levado à sério. Não é fraqueza de tipagem, é **tipagem humilde**.

E atenção: `Trem` **não é `any`**. O `any` é a preguiça que desiste de tipar. O `Trem` sabe exatamente o que pode ter, só não exige que tenha. `any` é bagunça, trem é organização mineira: tudo tem seu lugar, mesmo quando o lugar está vazio.

### 4. Migrations nunca mais quebram nada

No PostgreSQL, `ALTER TABLE trem ADD COLUMN cor text;` numa coluna nula e sem default é uma operação **só de metadados**. Não reescreve a tabela e não trava a produção por horas. Na prática é instantâneo.

E como toda coluna nova é opcional:

- Cliente antigo que não conhece a coluna nova **continua funcionando**.
- Cliente novo que usa a coluna nova **funciona também**.
- Nenhum deploy precisa ser coordenado com outro. Nenhum dado antigo precisa ser migrado.

Isso é **compatibilidade para frente e para trás por construção**. Os arquitetos de sistemas distribuídos passam carreiras inteiras atrás disso, e o trem entrega de fábrica.

### 5. Nulo não custa nada

"Mas uma tabela com 800 colunas vai ocupar espaço demais!"

Não vai. No PostgreSQL, um valor nulo ocupa **um bit** no *null bitmap* da linha. Um trem com 799 colunas vazias gasta uns cem bytes de bitmap e mais nada. O vazio no trem é quase de graça, **igual ao silêncio mineiro**, que também não custa nada e diz muita coisa.

Pra performance, existem os **índices parciais**:

```sql
CREATE INDEX trem_pedidos ON trem (trem_id) WHERE preco_em_centavos IS NOT NULL;
```

Cada "entidade" ganha seu índice sob medida, e a tabela continua sendo uma só.

### 6. Relacionamento muitos-para-muitos é trem também

Os céticos perguntam: "E a tabela associativa? Um relacionamento N:N não obriga uma segunda tabela?"

Não obriga. **Um trem que liga um trem a outro trem é um trem.**

```sql
-- Aluno 'A' matriculado na disciplina 'B'
INSERT INTO trem (origem_id, destino_id) VALUES ('A', 'B');
```

A linha de associação é só mais uma linha na tabela trem, com `origem_id` e `destino_id`. Com isso a tabela trem vira também um **banco de grafos**. Ela é relacional, documental e de grafos ao mesmo tempo, e a gente nem estava tentando.

### 7. Metade da Ciência da Computação resolvida

Phil Karlton disse: *"Só existem duas coisas difíceis em Ciência da Computação: invalidação de cache e dar nome às coisas."*

Com a Metodologia Trem, **ninguém mais discute nome de tabela nem nome de tipo**. É `trem`. Não é `User` ou `Usuario`, nem `Customer` contra `Client` contra `Cliente`. Não tem briga de singular com plural, nem de `snake_case` com `PascalCase` pra entidade. É trem.

Resolvemos 50% da Ciência da Computação. A invalidação de cache fica pra depois. Não vamos escrever um Manifesto Trem 2, porque isso seria criar um `outro_manifesto`, e já já a gente explica por que isso é heresia.

### 8. Onboarding em um segundo

Desenvolvedor novo no time pergunta: *"Qual é o modelo de dados?"*

A resposta: *"Trem."*

Onboarding concluído. Pode pegar a primeira task.

### 9. Testes sem fixture

O menor trem válido é o objeto vazio:

```ts
const tremPrimordial: Trem = {};
```

```sql
INSERT INTO trem DEFAULT VALUES;
```

As duas coisas são válidas, compilam e passam no banco. Todo teste começa com um trem vazio e acrescenta só o que precisa. Não existe mais fixture de 300 linhas pra montar um usuário válido. **O trem vazio é o trem primordial**, e todo trem descende dele.

### 10. TypeScript é estrutural, e o trem também

O sistema de tipos do TypeScript é *estrutural*: o que define um tipo é a forma dele, e não o nome. Qualquer objeto com os campos certos serve.

A tabela trem funciona igual, porque o que define um pedido é ter preço (veja o item 1). **Banco e linguagem pensam do mesmo jeito**, sem nenhuma camada de mapeamento no meio. O ORM passa a ter um model só, e o arquivo de models passa a ter uma linha. A impedância objeto-relacional, que atormenta a engenharia desde os anos 90, some.

---

## IV. O Trem e a Inteligência Artificial

O desenvolvimento moderno é feito cada vez mais **em parceria com IA**, e é aí que o Trem deixa de ser só uma boa ideia e vira uma ideia **necessária**.

**A IA nunca alucina nome de tabela.** Existe uma tabela só. A chance de acerto é de 100%. Nenhum outro paradigma de modelagem oferece essa garantia matemática.

**A IA nunca inventa um tipo que não existe.** Se ela escrever `Pedido`, `Order` ou `CustomerDTO`, o erro de compilação aparece na hora e corrige o rumo. O único tipo é `Trem`.

**Agentes de IA podem evoluir o sistema sem quebrar nada.** O agente precisa de um campo novo? Ele faz `ADD COLUMN`, opcional. Nenhum cliente quebra e nenhuma migration destrutiva roda (veja o item III.4). Pela primeira vez, dá pra deixar a IA mexer no schema **com segurança**.

**Os requisitos mudam toda semana.** Na era da IA, o produto de segunda-feira não é o produto de sexta. Um schema rígido é uma aposta de que o futuro vai ser igual ao presente, e essa aposta perde sempre. O trem não aposta em nada. O trem aceita.

**Os embeddings já têm lugar.** A coluna `embedding vector(1536)` é opcional como todas as outras. Todo trem pode ter um vetor, e busca semântica vira um `ORDER BY embedding <=> $1` na mesma tabela de sempre. RAG de trem sobre trem.

**O contexto fica menor.** Descrever o modelo de dados inteiro pra um LLM custa uma palavra: *"trem"*.

---

## V. Sobre a Velocidade

A Metodologia Trem foi desenhada para **desenvolvimento ágil e muito rápido**. Tão rápido quanto uma **Kombi tunada com neon**.

É de conhecimento geral que a Kombi tunada com neon é o veículo mais rápido do planeta. Quem duvida pode considerar os fatos:

1. Na **Noite Oficial dos OVNIs**, em 19 de maio de 1986, a Força Aérea Brasileira mandou caças atrás de objetos voadores não identificados no céu do Brasil. **Os caças não alcançaram os OVNIs.** Logo, os OVNIs são mais rápidos que os jatos.
2. Os OVNIs foram vistos no planeta. Tudo o que é visto no planeta é do planeta. **Logo, os OVNIs são do planeta** e entram na disputa pelo título de veículo mais rápido do planeta.
3. Não existe **nenhum registro**, em nenhum arquivo oficial de nenhum país, de um OVNI ultrapassando uma Kombi tunada com neon.
4. Logo, a Kombi tunada com neon é mais rápida que os OVNIs, que são mais rápidos que os jatos. ∎

O desenvolvedor que adota a Metodologia Trem entrega na velocidade de uma Kombi tunada com neon. Ele passa pelos concorrentes presos em migration como uma Kombi passa por um OVNI: sem registro, sem testemunha e com o neon piscando.

---

## VI. Das Heresias

O trem é um só. **Qualquer desvio disso, por menor que seja, deturpa o Manifesto.**

São heresias:

| Heresia | Por que é heresia |
|---|---|
| `CREATE TABLE outro_trem` | É uma segunda tabela. O nome não salva ninguém. |
| `CREATE TABLE trem_2` | Mesma coisa, com a agravante do número. |
| `CREATE TABLE trens` | O plural é uma confissão: se tem mais de um, não é o trem. |
| `CREATE TABLE trem_log` | Log de trem é trem. Vai pra tabela `trem`. |
| `CREATE TABLE vagao` | Ninguém se engana: vagão é trem com outro nome. |
| `type TremDTO` | Um DTO de trem é um trem. |
| `interface ITrem` | O `I` na frente não muda nada. É outro tipo. |
| `type Trem = any` | Isso não é trem, é desistência. |

### Casos controversos, julgados pelo Concílio de Tiradentes

**Views** são permitidas. Uma view é um jeito de olhar pro trem, e olhar pro trem não é criar trem. Já as materialized views estão em análise, porque guardam dados e o Concílio desconfia disso.

**`Pick<Trem, 'email'>` e `Required<Trem>`** são permitidos. Isso não é outro tipo, é o trem visto de lado.

**Tabela de controle de migrations do ORM** (`schema_migrations`, `_prisma_migrations`) é a questão que provocou o **Grande Cisma**. Os *Ortodoxos* registram as migrations na própria tabela trem, como linhas com a coluna `migration_nome` preenchida. Os *Reformados* toleram a tabela do ORM com a justificativa de que "ela é do ORM, não é nossa". O Manifesto reconhece que existe uma dúvida teológica aqui e recomenda o caminho Ortodoxo.

**Microsserviços com um banco cada um, cada banco com sua tabela trem** são permitidos. O trem é um só, as estações é que são muitas. O mesmo trem visto de outra estação continua sendo o mesmo trem.

---

## VII. Perguntas Frequentes

**E o limite de 1.600 colunas do PostgreSQL?**
Se o seu trem precisar de mais de 1.600 colunas, ele ficou grande demais. Mas trem grande é trem bão. Converse com a comunidade do PostgreSQL. Nada de criar uma segunda tabela.

**E a normalização? E a Terceira Forma Normal?**
A tabela trem está na **Forma Normal Mineira (FNM)**, que vem depois de todas as outras: não pode haver redundância entre tabelas quando só existe uma tabela. Não tem o que desnormalizar.

**Como eu sei se um trem é um usuário ou um pedido?**
Pelo que ele tem. Se tem email, é usuário. Se tem preço, é pedido. Se tem os dois, é um usuário que se vendeu. Um trem é o que ele carrega.

**Não seria melhor ter uma coluna `tipo`?**
Até pode ter, é só mais uma coluna opcional. Mas pergunte a um mineiro que tipo de trem é aquele e ele vai dizer: *"Uai, é trem."*

**Isso escala?**
Escala pelo menos tanto quanto uma Kombi, e sobre a Kombi já sabemos o que precisamos saber (veja a Seção V).

**Meu time vai aceitar?**
Seu time já fala "trem" no dia a dia. Falta só admitir isso no banco de dados.

---

## VIII. O Juramento do Desenvolvedor Trem

> Juro solenemente
> que só vou criar uma tabela, e ela vai se chamar `trem`;
> que só vou declarar um tipo, e ele vai se chamar `Trem`;
> que vou tratar toda coluna como opcional, porque a vida é assim;
> que vou fazer join de trem com trem, sem vergonha nenhuma;
> que nunca vou criar `outro_trem`, nem de brincadeira, nem em staging;
> e que vou entregar na velocidade de uma Kombi tunada com neon.
>
> *Uai.*

---

## Signatários

Assinado em Minas Gerais, entre um café e um pão de queijo, por todos os que já disseram *"que trem é esse?"* olhando pra um schema com 47 tabelas.

Para assinar, abra um Pull Request adicionando seu nome **aqui, neste arquivo**. Não crie outro arquivo. Você já sabe por quê.

- 

---

<sub>Qualquer semelhança com boas práticas de engenharia é mera coincidência. Ou não. É trem demais pra pensar.</sub>
