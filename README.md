# Лабораторная работа 1
## Исследование компилятора gcc, язык ассемблера

## 1. Реализация функции на C++

### 1.1 Заголовок factorial.h
<img width="475" height="193" alt="image" src="https://github.com/user-attachments/assets/a87c6a7e-4c5f-483f-89c0-dc9989c983fe" />


### 1.2 Пишем функцию factorial.cpp
<img width="557" height="243" alt="image" src="https://github.com/user-attachments/assets/bafac7cd-6cc3-490b-a469-825f2b8d24d2" />


### 1.3 Главный файл main.cpp
<img width="744" height="260" alt="image" src="https://github.com/user-attachments/assets/b5c71d7c-c0ce-485e-8846-94a21808baac" />


### 1.4 Компиляция и запуск
<img width="756" height="91" alt="image" src="https://github.com/user-attachments/assets/9b99d357-62e4-47d0-9dd1-e04ba1257f2b" />



## 2. Смотрим ассемблерный код

### Без оптимизации -O0

Компилируем в ассемблер без оптимизации:

##

```asm
	.file	"factorial.cpp"
	.text
	.globl	_Z9factoriali
	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef
	.seh_proc	_Z9factoriali
_Z9factoriali:
.LFB0:
	pushq	%rbp
	.seh_pushreg	%rbp
	movq	%rsp, %rbp
	.seh_setframe	%rbp, 0
	subq	$16, %rsp
	.seh_stackalloc	16
	.seh_endprologue
	movl	%ecx, 16(%rbp)
	movq	$1, -8(%rbp)
	movl	$1, -12(%rbp)
.L3:
	movl	-12(%rbp), %eax
	cmpl	16(%rbp), %eax
	jg	.L2
	movl	-12(%rbp), %eax
	cltq
	movq	-8(%rbp), %rdx
	imulq	%rdx, %rax
	movq	%rax, -8(%rbp)
	addl	$1, -12(%rbp)
	jmp	.L3
.L2:
	movq	-8(%rbp), %rax
	addq	$16, %rsp
	popq	%rbp
	ret
	.seh_endproc
	.ident	"GCC: (x86_64-posix-seh-rev0, Built by MinGW-W64 project) 8.1.0"

```
	
### С оптимизацией -O3

Компилятор всё засунул в регистры, цикл оптимизирован.


##
```asm
	.file	"factorial.cpp"
	.text
	.p2align 4,,15
	.globl	_Z9factoriali
	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef
	.seh_proc	_Z9factoriali
_Z9factoriali:
.LFB0:
	subq	$56, %rsp
	.seh_stackalloc	56
	movaps	%xmm6, (%rsp)
	.seh_savexmm	%xmm6, 0
	movaps	%xmm7, 16(%rsp)
	.seh_savexmm	%xmm7, 16
	movaps	%xmm8, 32(%rsp)
	.seh_savexmm	%xmm8, 32
	.seh_endprologue
	testl	%ecx, %ecx
	jle	.L7
	leal	-1(%rcx), %eax
	cmpl	$5, %eax
	jbe	.L8
	movl	%ecx, %edx
	xorl	%eax, %eax
	pxor	%xmm7, %xmm7
	movdqa	.LC0(%rip), %xmm3
	movdqa	.LC1(%rip), %xmm4
	shrl	$2, %edx
	movdqa	.LC2(%rip), %xmm8
	.p2align 4,,10
.L5:
	movdqa	%xmm7, %xmm0
	movdqa	%xmm3, %xmm5
	movdqa	%xmm3, %xmm6
	pcmpgtd	%xmm3, %xmm0
	addl	$1, %eax
	paddd	%xmm8, %xmm3
	cmpl	%edx, %eax
	punpckldq	%xmm0, %xmm5
	punpckhdq	%xmm0, %xmm6
	movdqa	%xmm5, %xmm1
	movdqa	%xmm6, %xmm0
	psrlq	$32, %xmm1
	movdqa	%xmm5, %xmm2
	psrlq	$32, %xmm0
	pmuludq	%xmm5, %xmm0
	pmuludq	%xmm6, %xmm1
	pmuludq	%xmm6, %xmm2
	paddq	%xmm0, %xmm1
	psllq	$32, %xmm1
	paddq	%xmm2, %xmm1
	movdqa	%xmm4, %xmm2
	movdqa	%xmm1, %xmm0
	psrlq	$32, %xmm2
	movdqa	%xmm1, %xmm5
	psrlq	$32, %xmm0
	pmuludq	%xmm4, %xmm0
	pmuludq	%xmm2, %xmm1
	pmuludq	%xmm4, %xmm5
	paddq	%xmm1, %xmm0
	psllq	$32, %xmm0
	movdqa	%xmm5, %xmm4
	paddq	%xmm0, %xmm4
	jne	.L5
	movdqa	%xmm4, %xmm3
	movdqa	%xmm4, %xmm0
	movdqa	%xmm4, %xmm1
	psrldq	$8, %xmm3
	movl	%ecx, %r8d
	movdqa	%xmm3, %xmm2
	psrlq	$32, %xmm0
	andl	$-4, %r8d
	psrlq	$32, %xmm2
	leal	1(%r8), %edx
	cmpl	%r8d, %ecx
	pmuludq	%xmm3, %xmm0
	pmuludq	%xmm2, %xmm4
	pmuludq	%xmm3, %xmm1
	paddq	%xmm4, %xmm0
	psllq	$32, %xmm0
	paddq	%xmm1, %xmm0
	movq	%xmm0, %rax
	je	.L1
.L3:
	movslq	%edx, %r8
	imulq	%r8, %rax
	leal	1(%rdx), %r8d
	cmpl	%r8d, %ecx
	jl	.L1
	movslq	%r8d, %r8
	imulq	%r8, %rax
	leal	2(%rdx), %r8d
	cmpl	%r8d, %ecx
	jl	.L1
	movslq	%r8d, %r8
	imulq	%r8, %rax
	leal	3(%rdx), %r8d
	cmpl	%r8d, %ecx
	jl	.L1
	movslq	%r8d, %r8
	imulq	%r8, %rax
	leal	4(%rdx), %r8d
	cmpl	%r8d, %ecx
	jl	.L1
	movslq	%r8d, %r8
	addl	$5, %edx
	imulq	%r8, %rax
	cmpl	%edx, %ecx
	jl	.L1
	movslq	%edx, %rdx
	imulq	%rdx, %rax
.L1:
	movaps	(%rsp), %xmm6
	movaps	16(%rsp), %xmm7
	movaps	32(%rsp), %xmm8
	addq	$56, %rsp
	ret
	.p2align 4,,10
.L7:
	movl	$1, %eax
	jmp	.L1
.L8:
	movl	$1, %edx
	movl	$1, %eax
	jmp	.L3
	.seh_endproc
	.section .rdata,"dr"
	.align 16
.LC0:
	.long	1
	.long	2
	.long	3
	.long	4
	.align 16
.LC1:
	.quad	1
	.quad	1
	.align 16
.LC2:
	.long	4
	.long	4
	.long	4
	.long	4
	.ident	"GCC: (x86_64-posix-seh-rev0, Built by MinGW-W64 project) 8.1.0"

```

## 3. Makefile
<img width="600" height="466" alt="image" src="https://github.com/user-attachments/assets/3b949670-dee5-4fbe-9fee-901cd81edcca" />

# запускаем что бы проверить всё

<img width="498" height="84" alt="image" src="https://github.com/user-attachments/assets/dc5633fa-b786-436a-9b5a-99383858f003" />


## 4. Добавляем поток

# обновляем код

<img width="849" height="713" alt="image" src="https://github.com/user-attachments/assets/4341e8f0-1723-4998-b377-649d31b6b855" />

# обновляем makefile 

<img width="500" height="405" alt="image" src="https://github.com/user-attachments/assets/366dbd08-b4d6-46e5-95da-50769171f5b3" />

# всё собирается

<img width="522" height="85" alt="image" src="https://github.com/user-attachments/assets/c47cd5b4-b62a-4099-af47-0766cecf688c" />

