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
	pushq	%rbp              // сохраняем базовый указатель стека
	.seh_pushreg	%rbp
	movq	%rsp, %rbp        // устанавливаем новый кадр стека
	.seh_setframe	%rbp, 0
	subq	$16, %rsp         // выделяем 16 байт для локальных переменных
	.seh_stackalloc	16
	.seh_endprologue
	movl	%ecx, 16(%rbp)    // сохраняем аргумент n на стек
	movq	$1, -8(%rbp)      // res = 1
	movl	$1, -12(%rbp)     // i = 1
.L3:                          // начало проверки условия цикла
	movl	-12(%rbp), %eax   // загружаем i
	cmpl	16(%rbp), %eax    // сравниваем i с n
	jg	.L2               // если i > n, выходим из цикла
	movl	-12(%rbp), %eax   // загружаем i
	cltq                      // расширяем i до 64 бит
	movq	-8(%rbp), %rdx    // загружаем res
	imulq	%rdx, %rax        // res = res * i
	movq	%rax, -8(%rbp)    // сохраняем результат в res
	addl	$1, -12(%rbp)     // i++
	jmp	.L3               // возвращаемся к проверке условия
.L2:                          // выход из цикла
	movq	-8(%rbp), %rax    // помещаем результат в rax для возврата
	addq	$16, %rsp         // освобождаем стек
	popq	%rbp              // восстанавливаем базовый указатель
	ret                       // возвращаемся из функции
	.seh_endproc
	.ident	"GCC: (x86_64-posix-seh-rev0, Built by MinGW-W64 project) 8.1.0"
```
	
### С оптимизацией -O3

Компилятор всё засунул в регистры, цикл оптимизирован.

```asm
.file	"factorial.cpp"
	.text
	.p2align 4,,15            // выравнивание кода для быстрого выполнения
	.globl	_Z9factoriali
	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef
	.seh_proc	_Z9factoriali
_Z9factoriali:
.LFB0:
	subq	$56, %rsp         // выделяем место на стеке
	.seh_stackalloc	56
	.seh_endprologue
	testl	%ecx, %ecx        // проверяем n на ноль
	jle	.L7               // если n <= 0, возвращаем 1
	leal	-1(%rcx), %eax    // eax = n - 1
	cmpl	$5, %eax          // если n маленькое - идём без SIMD
	jbe	.L8               // переход к простому циклу

	// ОПТИМИЗИРОВАННЫЙ ЦИКЛ через SIMD (вычисление сразу нескольких чисел)
	movl	%ecx, %edx        // копируем n
	xorl	%eax, %eax        // обнуляем счётчик
	pxor	%xmm7, %xmm7     // обнуляем xmm7
	movdqa	.LC0(%rip), %xmm3 // загружаем константы
	movdqa	.LC1(%rip), %xmm4
	shrl	$2, %edx          // делим n на 4 (шаг SIMD)
	movdqa	.LC2(%rip), %xmm8
.L5:                          // основной SIMD цикл
	movdqa	%xmm7, %xmm0
	movdqa	%xmm3, %xmm5
	movdqa	%xmm3, %xmm6
	pcmpgtd	%xmm3, %xmm0
	addl	$1, %eax          // счётчик итераций
	paddd	%xmm8, %xmm3     // увеличиваем на шаг
	cmpl	%edx, %eax
	// умножение через SIMD регистры
	punpckldq	%xmm0, %xmm5
	punpckhdq	%xmm0, %xmm6
	pmuludq	%xmm5, %xmm0
	pmuludq	%xmm6, %xmm1
	paddq	%xmm0, %xmm1
	jne	.L5               // повторяем пока не дошли до n

.L3:                          // финальное умножение остатка
	movslq	%edx, %r8
	imulq	%r8, %rax         // перемножаем оставшиеся элементы

.L7:                          // возврат 1 если n <= 0
	movl	$1, %eax
	addq	$56, %rsp         // освобождаем стек
	ret                       // возврат из функции

.L8:                          // простой цикл для маленьких n
	movl	$1, %eax          // результат = 1
	movl	$1, %edx          // i = 1
.L9:
	imulq	%rdx, %rax        // res *= i
	addq	$1, %rdx          // i++
	cmpl	%ecx, %edx        // i <= n?
	jle	.L9               // если да - продолжаем
	addq	$56, %rsp
	ret
	.seh_endproc
	.ident	"GCC: (x86_64-posix-seh-rev0, Built by MinGW-W64 project) 8.1.0"
```
##

## 3. Makefile
<img width="600" height="466" alt="image" src="https://github.com/user-attachments/assets/3b949670-dee5-4fbe-9fee-901cd81edcca" />

# запускаем что бы проверить всё

<img width="498" height="84" alt="image" src="https://github.com/user-attachments/assets/dc5633fa-b786-436a-9b5a-99383858f003" />


## 4. Добавляем поток

## обновляем код

<img width="849" height="713" alt="image" src="https://github.com/user-attachments/assets/4341e8f0-1723-4998-b377-649d31b6b855" />

## обновляем makefile 

<img width="500" height="405" alt="image" src="https://github.com/user-attachments/assets/366dbd08-b4d6-46e5-95da-50769171f5b3" />

## всё собирается

<img width="522" height="85" alt="image" src="https://github.com/user-attachments/assets/c47cd5b4-b62a-4099-af47-0766cecf688c" />

#### Лабораторная работа 2: Установка Linux

https://disk.yandex.ru/d/eX_zqxEc24wHBA

## Лабораторная работа 3a — Реализация скрипта Bash
Вариант 4: Поиск строки в текстовых файлах
##### Скрипт принимает слово как аргумент командной строки, обходит все .txt файлы в текущей директории, находит файлы содержащие заданное слово и сохраняет их список в list.txt

### Шаг 1: Создание рабочей папки и файлов

<img width="1080" height="121" alt="image" src="https://github.com/user-attachments/assets/f241c10b-64bf-4ac0-ab2a-6c32dc1d1582" />

### Шаг 2: Проверка файлов

<img width="590" height="42" alt="image" src="https://github.com/user-attachments/assets/85d3cb7c-c940-439e-9ae8-588e8aac161b" />

### Шаг 3 Скрипт script.sh

<img width="757" height="371" alt="image" src="https://github.com/user-attachments/assets/f528eec5-2745-4fab-a90b-120f38477dfd" />

### Шаг 4: Запуск и результат

<img width="434" height="86" alt="image" src="https://github.com/user-attachments/assets/0a06b17d-90a8-460c-9c5e-1ffea4374b7c" />

## Лабораторная работа 3b
### PowerShell скрипт — поиск строки в .txt файлах

**Вариант 4:** то же задание но на Windows PowerShell.

### 1. Создаём папку и тестовые файлы
<img width="469" height="109" alt="image" src="https://github.com/user-attachments/assets/d55474e2-2c1b-45dc-a1f8-8b32dc30b433" />


### 2. Создаём скрипт
<img width="625" height="179" alt="image" src="https://github.com/user-attachments/assets/48d71f8d-aa44-4d85-842b-0949bfb16a47" />


### 3. Разрешаем выполнение скриптов


<img width="1024" height="124" alt="image" src="https://github.com/user-attachments/assets/cae9bc5d-c54c-4599-8a74-775c39e883da" />




### 4. Запускаем скрипт и смотрим результат

<img width="476" height="86" alt="image" src="https://github.com/user-attachments/assets/0734fcc4-c70b-4bc8-a10e-18676c55d281" />


<img width="936" height="219" alt="image" src="https://github.com/user-attachments/assets/335b7af2-d265-4e9c-84b0-28e41130479c" />
