# Calculadora Brainfuck — dígitos 0-9 com `+` e `-`

Pequeno experimento em **Brainfuck** que lê **dois** operandos de um único dígito (0-9) e um operador (`+` ou `-`) em ASCII e devolve o resultado como **um** dígito (0-9).

```
Exemplos
========
Entrada  Saída
-------  -----
2+3      5
5-2      3
9-9      0
0+9      9
```

> Qualquer caractere fora desse padrão cancela o cálculo ou gera caractere lixo.

---

## Código-fonte

```brainfuck
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

---

## Como executar

1. Instale um interpretador Brainfuck (ex.: `bf`, `bfi`, `brainfuck` via pip).
2. Salve o código acima em `calc.bf`.
3. Execute:

```bash
echo "2+3" | bf calc.bf      # → 5
```

*A maioria dos intérpretes lê de **STDIN** e escreve em **STDOUT**. Cada execução resolve **uma** expressão.*

---

## Limitações

* Aceita **apenas um operador** (`+` ou `-`) e **dois dígitos** simples.
* O resultado também deve caber em `0-9` (não testado para “9+9” → 18).
* Sem tratamento de erro/overflow; expressão fora do padrão gera byte indefinido.
* Não aceita espaços — a string deve vir **sem** whitespace.

---

## Possíveis melhorias

* Suportar números multi-dígito (parser decimal em loop).
* Implementar multiplicação, divisão, potenciação.
* Permitir múltiplas expressões (uma por linha).
* Mensagens de erro legíveis em vez de caractere lixo.

---

## Anotação detalhada – célula por célula e ciclo por ciclo

(pense a fita do Brainfuck indexada a partir de `0`, ponteiro inicia em `0`)


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
[>+++++++++<-]                 ; 5×(+9 em C6) → C6 = 45  
>--                            ; C6 = 43  
                               ; (C5 volta a 0)

------------------------------------------------------------------
### 3.  Determina ‘+’ ou ‘-’

[>-<-]                         ; transfere 43 de C6 para C7 (op)  
                               ; C7 = op – 43  → 0 se ‘+’, 2 se ‘-’

>                              ; ponteiro agora em C7  
[                              ; =========== executa **apenas se C7==2** (ou seja, ‘-’) ===========
 <<<<[<->-]                    ;   C3 (2·d2) é subtraído de C2  
                               ;   ⇒ C2 = d1 + d2 – 2·d2 = d1 – d2
 >> ++++++++++[>++++++++++<-]  ;   cria 100 em C6
 >----[<<+>>-]                 ;   move 96 de C6 → C4
 <<[<<+>>-]                    ;   move 96 de C4 → C2
 >> >--                        ;   C7 -= 2  → sai do laço
]                              ; ==================================================================

Se o operador era ‘-’, **C2 = d1 - d2 + 96**  
Se era ‘+’, o bloco é pulado, então **C2 = d1 + d2**

------------------------------------------------------------------
### 4.  Subtrai 48 (ASCII ‘0’) para obter ASCII do resultado

>+++++++++[<+++++>-]<+++       ; gera a constante 48 em C7  
[<<<<<->>>>>-]                 ; transfere −48 de C7 → C2  
                               ; C2 agora guarda **ASCII( N1 ± N2 )**

------------------------------------------------------------------
### 5.  Imprime “=” e o resultado

++++++[>++++++++++<-]>+.       ; escreve '=' (61)  
<<<<<<.                        ; escreve C2 (dígito resultado)

A saída final é:

<eco>   <eco>   <eco>  =  <resultado>
  d1      op      d2

(Num interpretador que **não ecoa** a entrada, você verá apenas `=<dígito>`.)

------------------------------------------------------------------
## Resumo do estado final das células

| Célula | Uso final                                    |
|--------|----------------------------------------------|
| **C0** | 0 (zerada após cópia)                        |
| **C1** | cópia extra do 1º dígito (não usada depois)  |
| **C2** | **ASCII do resultado**                       |
| **C3** | trabalho (zerada)                            |
| **C4** | trabalho (zerada)                            |
| **C5** | trabalho (zerada)                            |
| **C6** | gera 43 e depois 100 → 0                     |
| **C7** | decide ramo ‘+’/‘-’, depois gera 48 → 0      |
| **C8** | 61 (`'='`)                                   |

------------------------------------------------------------------
## Por que tantas “mágicas” de 43, 48 e 96?

* **43** = ASCII de `'+'`. Subtraí-lo do próprio operador faz `‘+’→0`, `‘-’→2`, valor usado como contador para o loop condicional.  
* **96** = 2 × 48. Ao subtrair, `d1 – d2` ficaria abaixo de 48; somar 96 traz de volta acima de 48 antes do ajuste final.  
* **48** é adicionado ao valor numérico para voltar ao código ASCII do dígito resultado.

Com cerca de 30 comandos efetivos, o programa:

1. **Copia** bytes sem destruí-los (`[>+>+<<-]`).  
2. **Constrói** as constantes 43, 48, 96 e 100 do zero, usando loops.  
3. **Seleciona** dinamicamente o bloco `+` ou `-` através do contador em C7.  
4. **Executa aritmética** só com incrementos, decrementos e movimentação.  
5. **Converte** entre ASCII ↔ valor numérico sem tabelas, apenas aritmeticamente.

Um exemplo de como o minimalismo extremo do Brainfuck ainda permite lógica não trivial.

---

## Referências

* [Brainfuck — Esolang](https://esolangs.org/wiki/Brainfuck)  
* Repositórios com intérpretes Brainfuck no GitHub: https://github.com/search?q=brainfuck+interpreter

---

> Projeto de estudo — clone, experimente e, se quiser, abra *pull requests*!
::contentReference[oaicite:0]{index=0}
