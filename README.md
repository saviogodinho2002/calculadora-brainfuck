# Calculadora Brainfuck (operadores `+` e `-`, dígitos 0-9)

Pequeno experimento em **Brainfuck** que lê expressões muito simples com **dois operandos de um único dígito** e um operador (`+` ou `-`) — tudo em ASCII — e devolve o resultado como um único dígito (0-9).

```
Exemplos de entrada          Saída
-------------------          -----
2+3                          5
5-2                          3
9-9                          0
0+9                          9
```

> Qualquer caractere fora desse padrão cancela o cálculo ou gera resultado incorreto.

---

## 📄 Código-fonte

```
,.                       
[>+>+<<-]>>>[<<+>>-]

>>>>,.                        

<<<<,.                         
[>+>+<<-]>>>[<<+>>-]

<<[<++>-]<
>>[<<<+>>>-]
+++++[>+++++++++<-]>--
[>-<-]
>
[ <<<<[<->-] >> ++++++++++[>++++++++++<-]>----[<<+>>-] <<[<<+>>-]>> >--]
>+++++++++[<+++++>-]<+++
[<<<<<->>>>>-]
++++++[>++++++++++<-]>+.
<<<<<<.
```

Essa sequência Brainfuck faz:

1. **Lê** primeiro dígito (`'0'`–`'9'`) → converte para valor numérico (ASCII – 48).
2. **Armazena** em célula A.
3. **Lê** o operador (`'+'` ou `'-'`).
4. **Lê** o segundo dígito → converte → célula B.
5. Executa **ADIÇÃO** se operador = `'+'` ou **SUBTRAÇÃO** se `'-'`.

   * A lógica usa comparação do byte do operador para desviar o fluxo e
     aplicar `B` a `A` com sinais opostos.
6. **Converte** o resultado de volta para ASCII (`+48`) e imprime.

O algoritmo pressupõe que o resultado também cabe em 0-9 (ex.: não testado para “9+9” que deveria dar 18).

---

## ▶️ Como executar

1. Instale um interpretador Brainfuck. Exemplos:

   ```bash
   # Ubuntu / Debian
   sudo apt install bf         # pacote 'bf'
   # ou instale via pip:
   pip install brainfuck
   ```

2. Salve o código acima em `calc.bf`.

3. Execute:

   ```bash
   # Usando bf
   echo "2+3" | bf calc.bf
   # → 5
   ```

   A maioria dos intérpretes lê de **STDIN** e escreve em **STDOUT**.
   Cada execução resolve **uma** expressão.

---
**Anotação detalhada – célula por célula e ciclo por ciclo**
(pense a fita do Brainfuck indexada a partir de `0`, ponteiro inicia em `0`)

```
, .                            ╮ 1)  lê 1º dígito (ASCII ‘0’–‘9’) em C0
                               ╰ 2)  ecoa o próprio dígito

[>+>+<<-]                      copia C0 → C1 e C2, zera C0
                               └─ padrão clássico “copy-twice”

>>>[<<+>>-]                    salta p/ C3; nada a copiar (C3==0)

>>>> , .                       lê **operador** (‘+’=43 ou ‘-’=45) em C7, ecoa
<<<< , .                       volta a C3; lê 2º dígito em C3, ecoa

[>+>+<<-]>>>[<<+>>-]           copia 2º dígito: C3 → C4 e C5, zera C3

=========================================================
Até aqui a fita está assim (valores em decimal):

 idx : 0   1    2    3  4    5    6   7
 val : 0  d1  d1   0  d2   d2    0  op
=========================================================

● d1 = ASCII do 1º dígito = 48+N1  
● d2 = ASCII do 2º dígito = 48+N2  
● op = 43 (’+’)  **ou** 45 (’-’)

------------------------------------------------------------------
### 1.  Converte **2º dígito** de ASCII → valor e guarda a soma

<<[<++>-]<                     ; ponteiro → C4
                               ; laço: move 2×d2 de C4 → C3
                               ; resultado: C3 = 2·d2, C4 = 0
>>[<<<+>>>-]                   ; ponteiro → C5
                               ; laço: move d2 de C5 → C2
                               ; agora C2 = d1 + d2, C5 = 0

------------------------------------------------------------------
### 2.  Constrói a constante **43** em C6
+++++                          ; C5 = 5
[>+++++++++<-]                 ; 5×(-9 em C6) → C6 = 45
>--                            ; C6 = 43
                               ; (C5 volta a 0)

------------------------------------------------------------------
### 3.  Opções ‘+’ ou ‘-’:  testa o operador

[>-<-]                         ; transfere 43 de C6 para C7 (op)
                               ; C7 = op – 43  →  0 se ‘+’, 2 se ‘-’

>                              ; ponteiro agora em C7
[                              ; === executa **apenas se C7==2** (isto é, ‘-’) ===
 <<<<[<->-]                    ;   C3 (2·d2) é subtraído de C2
                               ;   ⇒ C2 = d1 + d2 – 2·d2 = d1 – d2
 >> ++++++++++[>++++++++++<-]  ;   cria 100 em C6
 >----[<<+>>-]                 ;   move 96 de C6 → C4
 <<[<<+>>-]                    ;   move 96 de C4 → C2
 >> >--                        ;   C7 -= 2  → sai do laço
]                              ; ===============================================

Se o operador era ‘-’, **C2 = d1 - d2 + 96**  
Se era ‘+’, o bloco é pulado, então **C2 = d1 + d2**

------------------------------------------------------------------
### 4.  Subtrai 48 (ASCII ‘0’) para obter ASCII do resultado

>+++++++++[<+++++>-]<+++       ; gera a constante 48 em C7
[<<<<<->>>>>-]                 ; transfere −48 de C7 → C2
                               ; C2 agora guarda **ASCII( N1 ± N2 )**

------------------------------------------------------------------
### 5.  Imprime “=“ e o resultado

++++++[>++++++++++<-]>+.       ; escreve '='     (61)
<<<<<<.                        ; escreve C2      (dígito resultado)

Programa encerra: a saída final é  
```

<eco>   <eco>   <eco>  =  <resultado>
d1      op      d2

```
(Num interpretador que **não ecoa** a entrada, você verá apenas `=<dígito>`.)

------------------------------------------------------------------
## Resumo rápido do que acontece em cada célula

| Célula | Uso final                           |
|--------|-------------------------------------|
| **C0** | zerada após cópia                   |
| **C1** | cópia extra do 1º dígito (não usada depois) |
| **C2** | **ASCII do resultado**              |
| **C3** | trabalho (2 × d2, depois zerada)    |
| **C4** | trabalho (96 durante subtração)     |
| **C5** | trabalho (contador/zero)            |
| **C6** | gera 43 e depois 100 → 0            |
| **C7** | controla ramo ‘+’/’-’, depois gera 48 |
| **C8** | gera '=' (61)                       |

------------------------------------------------------------------
## Por que tantas “mágicas” de 43, 48 e 96?

* **43** = ASCII de ‘+’. Subtraí-lo do próprio operador faz `‘+’→0`, `‘-’→2`.  
  O valor resultante em C7 é usado como *contador* para decidir se o trecho de
  subtração executa.
* **96** é 2 × 48.  No caso de subtração, primeiro fazemos `d1 – d2`
  (ficando abaixo de 48).  Somar 96 garante que continuamos **acima**
  de 48 para, em seguida, subtrair exatamente 48 e cair no intervalo `'0'–'9'`.
* **48** é somado a (N1 ± N2) para voltar ao código ASCII do dígito.

Com apenas 30-e-poucos comandos “funcionais”, o programa:

1. **Copia** valores sem destruir dados (padrões `[>+>+<<-]`).  
2. **Constrói** constantes 43, 48, 96 e 100 “do zero” (incrementos + loops).  
3. **Seleciona** o bloco de cálculo pela contagem deixada em C7.  
4. **Faz aritmética** usando apenas operações de incremento/decremento e *move*.  
5. **Converte** entre ASCII e valor numérico de forma aritmética, não por tabela.  

Um belo exemplo de quão longe dá para ir mesmo com o minimalismo extremo do Brainfuck.
::contentReference[oaicite:0]{index=0}
```
## 🛑 Limitações conhecidas

* Aceita **apenas um operador** (`+` ou `-`) e **dois dígitos simples**.
* Não verifica overflow/underflow; resultados fora de 0-9 não são válidos.
* Não há suporte a espaço em branco; a expressão deve vir sem espaços e sem caracteres extras.
* Entrada com newline no fim é necessária em alguns intérpretes (use `echo "2+3"` que já inclui `\n`).

---

## ✨ Possíveis melhorias

* Suportar números multi-dígito (exige loop para parser decimal).
* Implementar multiplicação/divisão (logicamente mais complexas em Brainfuck).
* Permitir múltiplas expressões na mesma execução, separadas por `\n`.
* Tratar erro de sintaxe e imprimir mensagem em vez de caractere lixo.

---

## 📚 Referências

* [Especificação Brainfuck — Esolang](https://esolangs.org/wiki/Brainfuck)
* [Intepretes Brainfuck em várias linguagens](https://github.com/search?q=brainfuck+interpreter)

---

> Projeto de estudo — fique à vontade para clonar, experimentar e propor alterações!

