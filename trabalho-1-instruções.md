[Simulador](https://www.kvakil.me/venus/)

[Interpretador](https://www.cs.cornell.edu/courses/cs3410/2019sp/riscv/interpreter/#)

- Referências consultadas

    [https://github.com/riscv/riscv-asm-manual/blob/master/riscv-asm.md](https://github.com/riscv/riscv-asm-manual/blob/master/riscv-asm.md)

    [https://github.com/mortbopet/Ripes/wiki/RISC-V-Assembly-Programmer's-Manual-(Adapted-for-Ripes)](https://github.com/mortbopet/Ripes/wiki/RISC-V-Assembly-Programmer%27s-Manual-(Adapted-for-Ripes))

    [https://virtual.ufmg.br/20211/pluginfile.php/440586/mod_resource/content/1/RISC-V-Reference-Data.pdf](https://virtual.ufmg.br/20211/pluginfile.php/440586/mod_resource/content/1/RISC-V-Reference-Data.pdf)

    [https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf)

    [https://medium.com/swlh/risc-v-assembly-for-beginners-387c6cd02c49](https://medium.com/swlh/risc-v-assembly-for-beginners-387c6cd02c49)

    [https://erik-engheim.medium.com/risc-v-assembly-code-examples-7bca0e7ebaa3](https://erik-engheim.medium.com/risc-v-assembly-code-examples-7bca0e7ebaa3)

    [https://rosettacode.org/wiki/Sorting_algorithms/Insertion_sort#360_Assembly](https://rosettacode.org/wiki/Sorting_algorithms/Insertion_sort#360_Assembly)

    [https://riscv.org/news/2020/10/bubble-sort-in-risc-v-assembly-video-learn-risc-v/](https://riscv.org/news/2020/10/bubble-sort-in-risc-v-assembly-video-learn-risc-v/)

    [https://hackmd.io/@_01X9rimQmWH33Djf8QhoA/HJ1QiTzYB](https://hackmd.io/@_01X9rimQmWH33Djf8QhoA/HJ1QiTzYB)

    [https://github.com/mish24/Assembly-step-by-step/blob/master/insertion-sort.asm](https://github.com/mish24/Assembly-step-by-step/blob/master/insertion-sort.asm)

## Problema 1: Lei de Ohm

Na questão 1 do TP1, considerem o registrador de retorno do valor calculado como sendo o registrador x10. Nessa mesma questão, não precisam considerar cálculos com ponto flutuante, apenas números inteiros.

$V = R \cdot I$

V → x10

R → x11

I → x12

retornar em x10

Escreva um programa assembly que, dadas duas dessas grandezas quaisquer, o programa possa calcular e retornar o valor da terceira grandeza. Considere x10, x11 e x12 como tensão (V ), resistência (R) e (I) respectivamente e a variável com valor zero é a que deve ser calculada. Caso mais de uma variável tenha valor zero, seu programa deve retornar também zero.

```wasm
# setando valores iniciais (mudá-los para fins de teste)
li x10, 0
li x11, 2
li x12, 3

# tensão = resistência x corrente
# resultado armazenado em x10

# validação de qual variável queremos calcular
beq x10, x0, tensao # V -> x10
beq x11, x0, resistencia # R -> x11
beq x12, x0, corrente # I -> x12

tensao:
		# se mais de uma variável for zero, retorna zero no x10
		beq x11, x0, retorna
		beq x12, x0, retorna

		# V = R x I
		mul x10, x11, x12

		jal x0 retorna

resistencia:
		beq x10, x0, retorna
		beq x12, x0, zeraRetorno

		# R = V/I
		div x10, x10, x12

		jal x0 retorna

corrente:
		beq x10, x0, retorna
		beq x11, x0, zeraRetorno

		# I = V/R
		div x10, x10, x11

		jal x0 retorna

zeraRetorno:
		# zera o valor do registrador x10
		mv x10, x0

		jal x0 retorna

retorna:
```

## Problema 2: Primos

```c
#define FALSE 0
#define TRUE 1

int primo (int n) {
	int d;
	if (n <= 1) return FALSE;
	for (d = 2; d < n; d++) {
		if (n%d == 0) return FALSE;
	}
	return TRUE;
}
```

```c
# N está no registrador x10
# retorno no registrador x11

li x10, 2 # valor inicial (mudar para fins de teste)
li x28, 1 # variável para retorno verdadeiro

primo:
	ble x10, x28, retornaFalso # if (n <= 1) return FALSE
	li x29, 2 # variável d = 2

	loop:
		beq x29, x10, retornaVerdadeiro # atingiu condição, sai do loop (return TRUE)
		rem x31, x10, x29 # n % d
		beq x31, x0, retornaFalso # if (n%d == 0) return FALSE;
		addi x29, x29, 1 # d++
		jal x0 loop
		
retornaVerdadeiro:
	mv x11, x28
	jal x0 retorno

retornaFalso:
	mv x11, x0
	jal x0 retorno

retorno:

```

## Problema 3: Ordenação

Implemente o insertion sort em assembly, considere que o começo do array está em x10 e o tamanho do array está em x11. Para esta atividade você pode utilizar arrays ou ponteiros.

```c
#include <stdio.h>

void insertionSort (int arr[], int n)
{
    int i, key, j;
    for (i = 1; i<n; i++) {
        key = arr[i];
        j = i-1;

        while (j >= 0 && arr[j] > key) {
            arr[j+1] = arr[j];
            j = j-1;
        }

        arr[j+1] = key;
    }
    
    for (i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
}

int main () {
    int arr[] = {27, 16, 20, 49, 39, 48, 38, 7, 5, 19};
    int n = sizeof(arr) / sizeof(arr[0]);

    insertionSort(arr, n);

    return 0;
}
```

```wasm
setup:
	# i -> x5
	li x5, 0 # i = 0
    # int arr[] = {27, 16, 20, 49, 39, 48, 38, 7, 5, 19};
    # o início do array está em x10
    # n -> x11
    li x11, 10
    
    # valor a ser salvo no array -> x12
    li x12, 27 # temp = 27
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++
    
    li x12, 16 # temp = 16
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 20 # temp = 20
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 49 # temp = 49
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 39 # temp = 39
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 48 # temp = 48
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 38 # temp = 38
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 7 # temp = 7
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 5 # temp = 5
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++    
    
    li x12, 19 # temp = 19
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x12, 0(x7) # array[i] = temp
    addi x5, x5, 1 # i++
    #j retorna

main:
	li x5, 1 # i = 1
    li x15, 1 # x15 = 1 (fixo)
    j for

for: 
    # i -> x5
    # key -> x29
    # j -> x30
    # n -> x11
    beq x5, x11, retorna # deve rodar enquando i < n, então se for igual ele sai
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    lw x29, 0(x7) # key = arr[i];
    sub x30, x5, x15 # j = i-1;
    j goWhile

goWhile:
 	blt x30, x0, backWhile # while j >= 0
    # arr[j] > key
    slli x6, x30, 3 # x6 = j * 8
    add x7, x10, x6 # x7 = endereço array[j]
    lw x28, 0(x7) # x12 = array[j]
    ble x28, x29, backWhile # se arr[j] <= key sai do while
    
    # arr[j+1] = arr[j]
    # o arr[j] ja está no x28
    addi x16, x30, 1 # b = j + 1
    slli x6, x16, 3 # x6 = b * 8
    add x7, x10, x6 # x7 = endereço array[i]
    sw x28, 0(x7) # x12 = array[i]

	# j = j-1;
    sub x30, x30, x15
    
    j goWhile
backWhile:
    # arr[j+1] = key;
    addi x28, x30, 1 # a = j + 1
    slli x6, x28, 3 # x6 = a * 8
    add x7, x10, x6 # x7 = endereço array[a]
    sw x29, 0(x7) # array[j+1] = key

	addi x5, x5, 1 # i++
    j for # continua rodando!
retorna:
 	li x5, 0 # i = 0
 	j imprimeArray
 
imprimeArray:
	# deveria ser: 5, 7, 16, 19, 20, 27, 38, 39, 48, 49
 	beq x5, x11, sair
    slli x6, x5, 3 # x6 = i * 8
    add x7, x10, x6 # x7 = endereço array[i]
    lw x12, 0(x7) # x12 = array[i]
    addi x5, x5, 1 # i++
    j imprimeArray
    
 sair:
```
