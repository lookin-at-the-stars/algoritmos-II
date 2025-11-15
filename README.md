# Algoritmo de Giro de Letras

Este documento explica o problema e a solução usando C, com detalhes sobre a estratégia e análise do código.

---

## 1️⃣ Contexto do Problema

O problema original é um clássico de **“giro de letras” (letter rotation)**.  

Dada uma string de letras, você pode girar cada letra de `'a'` a `'z'` (circularmente).  
O objetivo é transformar a string de forma a satisfazer certas condições **minimizando o número total de movimentos**.

O ponto crucial é perceber que **a ordem relativa das diferenças entre letras importa**, e podemos reduzir o problema a **diferenças mod 26**.

---

## 2️⃣ Estratégia do Código

O algoritmo utiliza uma abordagem **gulosa (greedy)** sobre **deltas**. A ideia geral é:

1. **Transformar a string em diferenças entre letras consecutivas (mod 26):**  
   - Isso reduz o problema: ao invés de olhar para cada letra individualmente, olhamos quanto cada letra precisa girar em relação à anterior.  
   - Exemplo:  
     ```
     a -> c -> b
     diferenças: 2, 25
     (porque 'c'-'a'=2 e 'b'-'c'=-1 ≡ 25 mod 26)
     ```

2. **Adicionar `'a'` no início e no fim da string:**  
   - Garante que cada letra tenha um "vizinho" inicial e final para o cálculo de diferença.  
   - Transforma a string original em algo simétrico, permitindo tratar extremos de forma uniforme.

3. **Ordenar as diferenças:**  
   - Permite aplicar o algoritmo guloso, focando em igualar deltas extremos primeiro.

4. **Aplicar dois ponteiros (`pos1` e `pos2`):**  
   - `pos1` aponta para os deltas menores ou zerados.  
   - `pos2` aponta para os deltas maiores.  
   - A cada passo:  
     - Incrementa `pos2` (gira o maior delta para frente)  
     - Decrementa `pos1` (gira o menor delta para trás)  
   - Isso minimiza o total de movimentos, porque cada giro reduz simultaneamente um delta grande e um delta pequeno.

5. **Terminar o loop quando todos os deltas são iguais ou zero:**  
   - Se ainda sobra algo no maior delta (`v[pos2]`), adiciona `26 - v[pos2]` ao total de movimentos, fechando a rotação.

---

## 3️⃣ Código C Completo

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// Função de comparação usada pelo qsort para ordenar inteiros
int cmp(const void *a, const void *b) {
    return (*(int*)a - *(int*)b);
}

int main() {
    char s[1010], aux[1012];
    int v[1010];

    // Lê linhas até encontrar '*'
    while (fgets(s, sizeof(s), stdin)) {

        int len = strlen(s);

        // Remove o '\n' se existir
        if (s[len-1] == '\n') 
            s[--len] = 0;

        // Condição de parada
        if (len == 1 && s[0] == '*') 
            break;

        /*
            A ideia aqui é transformar a string original "s"
            em uma cadeia circular, começando e terminando
            com a letra 'a'. Por isso o programa coloca 'a'
            no início e no fim. Isso facilita calcular
            os "incrementos" entre caracteres.
        */

        aux[0] = 'a';
        memcpy(aux + 1, s, len);
        aux[len + 1] = 'a';
        aux[len + 2] = 0;

        int n = len + 2;

        /*
            Agora calculamos as diferenças mod 26 entre
            cada par de caracteres consecutivos da nova string "aux":

            v[i] = (aux[i+1] - aux[i]) mod 26

            Isso cria um vetor de "incrementos" entre as letras,
            representando quanto cada salto no colar circular avança.
        */

        for (int i = 1; i < n; ++i) {
            v[i-1] = (aux[i] - aux[i-1] + 26) % 26;
        }

        // Ordena os incrementos
        qsort(v, n - 1, sizeof(int), cmp);

        int ans = 0;

        /*
            pos1 aponta para o primeiro valor diferente de zero.
            pos2 aponta para o último valor (o maior).
            
            O objetivo é "nivelar" todos os incrementos realizando
            operações de:
                - diminuir 1 do menor incremento não-zero
                - aumentar 1 do maior incremento
            Cada operação conta como 1 passo.
        */

        int pos1 = 0;
        while (pos1 < n - 1 && v[pos1] == 0) 
            ++pos1;

        // Se todos são 0, já está nivelado
        if (pos1 == n - 1) {
            printf("0\n");
            continue;
        }

        int pos2 = n - 2;

        /*
            Loop principal de redistribuição dos incrementos.
            Ele vai empurrando valor do maior para o menor
            até que todos possam ser zerados.
        */
        while (pos2) {

            ++ans;  // conta uma operação

            // aumenta o maior incremento
            v[pos2] = (v[pos2] + 1) % 26;

            // diminui o menor incremento não-zero
            v[pos1] = (v[pos1] - 1 + 26) % 26;

            // avança pos1 até achar um novo não-zero válido
            while (pos1 < n - 1 && (v[pos1] == 0 || v[pos1] == v[pos2])) 
                ++pos1;

            // se percorreu tudo, acabou
            if (pos1 == n - 1) 
                break;

            // recua pos2 até achar valor não-zero
            while (pos2 && v[pos2] == 0) 
                --pos2;
        }

        /*
            Se sobrou apenas um incremento não-zero,
            falta apenas completar até que ele vire 0.
            Exemplo: se for 5, faltam 21 operações (26 - 5).
        */
        if (v[pos2]) 
            ans += 26 - v[pos2];

        // imprime o total de operações
        printf("%d\n", ans);
    }

    return 0;
}

```

## 4️⃣ Análise Linha a Linha

- **`aux[0] = 'a'; memcpy(aux+1, s, len); aux[len+1] = 'a';`**  
  Insere `'a'` no começo e no fim da string.

- **`for (int i = 1; i < n; ++i) v[i-1] = (aux[i]-aux[i-1]+26)%26;`**  
  Converte a string em vetor de **deltas** (diferenças módulo 26).

- **`qsort(v, n-1, sizeof(int), cmp);`**  
  Ordena os deltas para aplicar a estratégia gulosa.

- **`int pos1 = 0; while (pos1 < n-1 && v[pos1] == 0) ++pos1;`**  
  Inicializa o ponteiro `pos1` para o primeiro delta não-zero.

- **Loop guloso (`pos2 = n-2; while(pos2){...}`)**  
  - Incrementa o delta maior (`pos2`) e decrementa o menor (`pos1`).  
  - Ajusta os ponteiros até que todos os deltas estejam equilibrados.

- **`if (v[pos2]) ans += 26 - v[pos2];`**  
  Adiciona o restante dos movimentos se o maior delta não for zero.

- **`printf("%d\n", ans);`**  
  Imprime o número mínimo de movimentos.

---

## 5️⃣ Tipo de Algoritmo e Complexidade

- **Transformação → Diferença → Ordenação → Guloso**

- **Complexidade:**  
  - `O(n log n)` por causa do `qsort`.  
  - Loop guloso é praticamente `O(n)` porque cada movimento aproxima os ponteiros de uma solução equilibrada.

---

## 6️⃣ Por Que Funciona

- Transformar em deltas simplifica o problema, pois gira cada letra de forma **relativa à anterior**, eliminando redundância.  
- Ordenar permite sempre reduzir o delta maior enquanto aumenta o menor, **minimizando movimentos**.  
- Essência do **algoritmo guloso** aplicado após transformação do problema.
