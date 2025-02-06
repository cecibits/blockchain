# blockchain

De acordo com o menu principal, temos as seguintes opções:

1. Criar bloco gênesis
2. Adicionar um novo bloco
3. Exibir blockchain completa
4. Validar a blockchain
5. Ataque
6. Proof of inclusion
7. Sair

## Criar bloco gênesis

Para inicializar uma blockchain, é necessário a criação do bloco inicial, chamado gênesis. Nele é utilizado a função *create_block*, do tipo Block, que recebe como parâmetro o **index** (nesse caso, o 0), a **hash anterior** (também 0), a **transação** (NULL), **número de transações** (0) e, por último, a **dificuldade** (quantidade de zeros adicionados).

## Exibir blockchain completa
Na main, é verificado se a blockchain é nula. Caso seja, o retorno é de uma blockchain vazia. Não sendo, ao menos temos o bloco gênesis, que foi o teste do print.
A impressão é relativamente simples, visto que apenas pega os dados da própria struct e printa até que o atual seja nulo. Ou seja, até que haja blocos a serem printados.

![Screenshot from 2025-02-05 19-29-58](https://github.com/user-attachments/assets/e18d5c4c-87a4-4405-a077-7d641dbbef10)

## Adicionar um novo bloco
Após a criação do bloco gênesis, podemos inserir um novo **bloco, onde o mesmo é** declarado como uma struct.
Como mencionado anteriormente, temos a função create_block, também do tipo Block. Recebe os parâmetros citados anteriormente e, após isso, aloca-se um trecho de memória com o tamanho da struct Block, indicando seu novo índice, uma cópia da hash anterior, uma segunda alocação de memória para a quantidade de transações e as transações em específico, copiando, assim, para a memória das transactions, carregando seu contador. É definido o nonce e o timestamp, que são valores contidos na struct bloco.
Após isso, temos a construção da merkle tree, obtendo o hash root. Caso o contador de transações seja 0, é feita uma cópia para o hashroot de ‘0’. Caso não, temos o cálculo das hashes de transações (*tx_hashes[block→transaction_count), onde para cada transação, teremos a chamada da função calculate_transaction_hash, que recebe o endereço das transações e seus respectivos espaços de memória em tx_hashes.
Entrando em *calculate_transaction_hash*, encontramos como parâmetro um ponteiro para struct do tipo Transaction e um ponteiro de char para output (anteriormente sendo preenchidos, respectivamente, pelos endereços das transações e o contador das transações). Com isso, calcula-se a hash por meio do SHA256, importado da biblioteca *#include <openssl/sha.h>.*
O output é convertido para um hexadecimal e então finalizado com um caracter sem valor.
Com os hashes de transação calculados, podemos retornar a build_merkle_tree definindo uma constante para o número de transações, no intuito de começar a construção da Merkle Tree.  A cada iteração, o número de hashes é diminuído pela metade, combinando pares de hashes em um único hash e gerando novos até que reste apenas 1 (o hashroot).
Finalizando a Merkle Root, temos a chamada da função proof_of_work, que recebe o bloco e sua dificuldade para encontrar uma hash válida. Enquanto não se encontrar uma hash válida, a função permanece buscando, incrementando o nonce e calculando o hash.
Similar ao cálculo de hash para transação, temos calculo do hash do bloco. É utilizado, novamente, o SHA256 para cálculo do hash do bloco.
Com isso, é inserido o novo bloco:

![Screenshot from 2025-02-05 19-31-09](https://github.com/user-attachments/assets/0ca17c7e-63c0-4a26-a612-086d1735b406)
![Screenshot from 2025-02-05 19-32-22](https://github.com/user-attachments/assets/bc521c21-5f45-4959-afd9-44ace6dca1ea)
![Screenshot from 2025-02-05 19-32-43](https://github.com/user-attachments/assets/5d46c004-aff4-486a-9c10-bb1c00fc3e1b)

Caso não haja um bloco gênesis:

![image](https://github.com/user-attachments/assets/117a2b02-dade-4ff2-ad48-4d1415ed2350)

Para cada opção na main, como dito anteriormente, é verificado se a blockchain está ou não vazia:
![image](https://github.com/user-attachments/assets/07e613cc-4bcd-43a0-8591-ebc873e8b7c6)

## Validar a blockchain

Após a inserção de um bloco, podemos validar o mesmo. Primeiro verificamos se o bloco existe. Caso não exista, é finalizado. Então temos a tratativa de bloco gênesis. Se existir um hash anterior no gênesis, ou seja, diferente de zero, a blockchain é invalidada, já que se trata de um bloco gênesis.
Caso não, enquanto os ponteiros não forem nulos, é verificado se o hash do bloco anterior é o mesmo hash anterior do novo bloco. Não sendo, temos falha na validação.
Por último, podemos validar a merkle root. Pra isso, utilizamos a função calcultate_merkle_root com o bloco atual. Foi necessário a criação de uma função específica para calcular a raiz, pois a build_merkle_tree atualiza a Merkle Root dentro do bloco, o que não queremos nesse caso.
A função calcula a Merkle Root de um bloco sem modificar aestrutura, garantindo a integridade das transações. Então verificamos se o bloco possui transações e, se não houver, define-se a raiz como ‘0’. Em seguida, calculamos os hashes de cada transação, armazenando em um array. Então constrói a árvore de Merkle combinando os hashes em pares, gerando um novo hash para cada par, repetindo o processo até restar apenas um hash, que se torna a Merkle Root. Se o número de hashes for ímpar, o último hash é copiado para a próxima rodada. Por fim, copia a raiz calculada pro output e libera a memória alocada. Essa função é essencial para validar, pois permite recalcular a Merkle Root sem modificar o bloco, garantindo que as transações não foram alteradas.
Por último, verificamos que se trata de uma blockchain válida.

![image](https://github.com/user-attachments/assets/28e3c8ba-bad3-4cc6-8993-fd81d1eae622)

## Ataque

Na main temos a chamada da função *ataque*, seguida da blockchain inteira pós ataque e a validação da blockchain pós ataque, o que resultará em um erro devido à inconsistência nos hashes dos blocos alterados.
![image](https://github.com/user-attachments/assets/12f8438e-333c-4b7f-91dd-8f9d1429f254)

A função ataque simula um ataque à blockchain ao modificar os dados do segundo bloco da cadeia. Primeiro, verifica se a blockchain possui pelo menos dois blocos e, caso positivo, acessa o segundo bloco (em blockchain→next). Então, altera os dados da primeira transação desse bloco, substituindo por “Transação manipulada’. Como a alteração impacta a integridade do bloco, a função recalcula a Merkle Root e, posteriormente, gera um novo hash para o bloco, substituindo o hash antigo pelo novo. 

## Proof of Inclusion

Com o proof of inclusion, verificamos se uma transação é ou não válida, se está presente em algum bloco da blockchain. Inicialmente é calculado o primeiro hash da transação (*target_hash).* Percorremos cada bloco da blockchain comparando o *target_hash* com os hashes de transações já armazenadas. Encontrando uma correpondência, verificamos se a transação realmente faz parte tentando gerar uma Merkle Proof. Se for válido, a transação é válida e está incluída. Caso não, continua buscando em outros blocos até que encontre a transação ou um bloco nulo. Sendo nulo, a transação não foi encontrada e a prova de inclusão é falsa.
![image](https://github.com/user-attachments/assets/a87e4955-e848-4240-a430-2474105ca3a9)
![image](https://github.com/user-attachments/assets/ada053cb-c7f7-48e1-b536-5bb13461e17f)
![image](https://github.com/user-attachments/assets/56919b15-6987-47e5-a81c-32284ee28524)

## Saída
Ao optar em *sair*, temos a função *free_blockchain*, criada apenas para não deixarmos memória alocada sem necessidade.
