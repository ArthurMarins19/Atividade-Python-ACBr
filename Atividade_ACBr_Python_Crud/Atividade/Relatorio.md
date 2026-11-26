1. Como o projeto funciona
Explique, com suas palavras, o funcionamento geral do projeto:
Qual é o objetivo da aplicação e o que é um CRUD.
O objetivo da aplicação seria cadastrar e gerenciar vagas. Cada vaga teria que ter título, empresa e telefone.
C: Create (Criar): insere novos registros ou dados no sistema
R: Read (Ler): consulta, busca ou exibe dados já armazenados
U: Update (Atualizar): modifica informações de um registro existente
D: Delete (Excluir): remove registros do sistema ou do banco de dados
O caminho que um dado percorre quando o usuário cadastra uma vaga: do menu no terminal até o arquivo vagas.db.
1.Eu escolho no menu a opção de cadastro
2.Aí eu insiro os dados de título, empresa e telefone
3.Os dados são encaminhados pro lugar que cria as vagas
4.O banco confere se tá tudo dentro das regras
5.Se tiver, ele grava e salva no arquivo vagas.db
 menu -> app.py -> crud.py -> SQLite -> vagas.db.
O papel de cada arquivo: app.py, crud.py e os scripts em sql/.
O app.py está responsável por mostrar para o usuário as opções e executar algumas funções que respondem de acordo com a opção escolhida, utilizando as funções do crud.py.
O crud.py tem dentro dele as funções responsáveis pela realização de operações no banco de dados, como cadastrar vagas novas, listá-las, consultá-las, atualizar e remover as vagas, utilizando as operações do CRUD.
Os scripts do SQL são responsáveis pela definição da estrutura do banco de dados, a criação de tabelas, podendo estabelecer restrições com os dados inseridos, e fornecendo comandos dentro do SQL com a manipulação de dados.
    
2. Exemplo de execução
Rode a aplicação (`python src/app.py`) e registre um exemplo real de uso:
- Cadastre pelo menos duas vagas, liste, atualize uma e remova outra.
- Cole no relatório a saída do terminal (ou capturas de tela) de cada passo.
- Descreva em uma frase o que cada passo fez.

![Passo 1](<img/WhatsApp Image 2026-11-25 at 20.47.52.jpeg>)
1. Cadastrei a primeira vaga, que ficou com o id 1.
    
![Passo 2](<img/WhatsApp Image 2026-11-25 at 20.49.54.jpeg>)
2. Cadastrei a segunda vaga, que ficou com o id 2.

![Passo 3](<img/WhatsApp Image 2026-11-25 at 20.51.57.jpeg>)
3. Listei as duas vagas cadastradas

![Passo 4](<img/WhatsApp Image 2026-11-25 at 21.03.17.jpeg>)
4. Atualizei os dados da vaga 1.

![Passo 5](<img/WhatsApp Image 2026-11-25 at 21.04.55.jpeg>)
5. Removi a vaga 2.

![Passo 6](<img/WhatsApp Image 2026-11-25 at 21.06.15.jpeg>)
6. Listei de novo e só restou a vaga 1.

![Sair](<img/WhatsApp Image 2026-11-25 at 21.09.32.jpeg>)
Sair: encerrei o programa com a opção 0.


3. Explicação de uma função: remover_vaga()

```python
def remover_vaga(conexao, id):
    sql = 'DELETE FROM vaga WHERE id = ?'
    cursor = conexao.execute(sql, (id,))
    conexao.commit()
    return cursor.rowcount
```

O comando SQL

DELETE FROM vaga WHERE id = ?

- DELETE FROM vaga: apaga uma linha da tabela vaga.
- WHERE id = ?: diz qual vaga apagar, a que tem o id informado. Sem o WHERE, apagaria todas.

O resto da função:

- conexao.execute(sql, (id,)): manda o banco apagar a vaga.
- conexao.commit(): salva a mudança. Sem isso, a vaga não some de verdade.
- cursor.rowcount: diz quantas vagas foram apagadas.

Por que usar ? e não colar o valor no SQL

Para evitar SQL injection. Se o id fosse colado no comando, alguém poderia digitar um comando no lugar do id e o banco executaria. Com o ?, o banco trata o que foi digitado só como um valor.

O que a função devolve e como o app.py usa

Ela devolve 1 se apagou a vaga e 0 se o id não existia. O app.py olha esse número: se for 1, mostra "Vaga removida."; se for 0, mostra "Vaga não encontrada.".


4. Encontre o easter egg 🥚

Durante a aula vimos o CRUD funcionando com dados "bem-comportados".
Mas o projeto tem uma regra escondida que não foi comentada em aula: em
certas situações o banco simplesmente se recusa a gravar uma vaga.

Sua missão:

- Provoque esse comportamento. Tente cadastrar vagas com dados fora do comum
  e observe quando a gravação é recusada. (Dica: preste atenção no campo empresa e no campo telefone.)
- Descubra onde no projeto essa regra está definida e como ela é escrita.
- Explique o que exatamente é exigido de cada campo e o que acontece com a aplicação quando a regra é violada (qual erro aparece e de onde ele vem).
- Comente: por que vale a pena ter esse tipo de regra no próprio banco, e não só no código da aplicação?

Como provoquei

Tentei cadastrar uma vaga com o telefone sem o 19 no começo e o banco não deixou:

```
Título: Base
Empresa: Corinthians
Telefone (DDD 19): 3333-1000
O banco recusou os dados: CHECK constraint failed: telefone LIKE '19%'
```

O banco tem duas regras escondidas:
- A empresa precisa ter pelo menos 3 letras.
- O telefone precisa começar com 19, que é o DDD.

O título só é obrigatório, não tem outra regra.

As regras estão no arquivo schema.sql, escritas com CHECK:

```sql
empresa TEXT NOT NULL CHECK (length(empresa) >= 3),
telefone TEXT NOT NULL CHECK (telefone LIKE '19%')
```

A vaga não é gravada e aparece uma mensagem de erro.

Se a empresa for curta demais, aparece:

```
O banco recusou os dados: CHECK constraint failed: length(empresa) >= 3
```

Se o telefone não começar com 19, aparece:

```
O banco recusou os dados: CHECK constraint failed: telefone LIKE '19%'
```

O erro se chama sqlite3.IntegrityError e vem do próprio banco (SQLite), que confere as regras do schema.sql na hora de gravar. O crud.py não trata o erro, então ele chega no app.py, que o captura com try/except, mostra a mensagem e volta para o menu.

Por que vale a pena ter a regra no banco e não só no código

Porque o banco continua protegendo os dados mesmo se o código da aplicação tiver um erro ou se os dados vierem de outro programa. Assim, dados fora das regras nunca são gravados, e os dados do banco ficam sempre corretos e confiáveis.