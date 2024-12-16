org 100h

; Definiciones y constantes
MAX_CARTAS equ 8
CARTAS db 'A', 'A', 'B', 'B', 'C', 'C', 'D', 'D'
MOSTRAR_CARTAS db MAX_CARTAS dup ('*')
SELECCION1 db '?'
SELECCION2 db '?'
ENCONTRADAS db 0
FIN db 0
vidas db 5
FILA db 0
COLUMNA db 0


; Procedimiento principal
IM: call setinicio
    call mostrarTitulo
    call mostrarCartas
    mov vidas, 5
    mov FIN, 0

CM: cmp FIN, 1
    je FMG
    call ingresarSeleccion
    cmp FIN, 1
    je FMG
    call compararSeleccion
    call verificarFin
    cmp vidas, 0
    je FMP
    jmp CM

FMG: call mostrarGano
     jmp FM

FMP: call mostrarPerdio
     jmp FM

FM: ret

; Procedimientos y subrutinas

mostrarTitulo proc near
    mov dx, offset TITULO
    call mostrarmsg
    ret
mostrarTitulo endp

mostrarCartas proc near
    ; Mostrar cartas ocultas
    mov cx, MAX_CARTAS
    mov si, offset MOSTRAR_CARTAS
MC_LOOP: push cx
         mov ah, 02h
         mov dl, [si]
         int 21h
         inc si
         pop cx
         loop MC_LOOP
    ret
mostrarCartas endp

ingresarSeleccion proc near
    ; Ingresar primera seleccion
    mov dx, offset MSG_SELECCION
    call mostrarmsg
    call ingresar
    sub al, '0'          ; Convertir carácter a número
    cmp al, 0
    jb INVALIDO
    cmp al, MAX_CARTAS - 1
    ja INVALIDO
    mov SELECCION1, al
    jmp SELECCION_VALIDA

INVALIDO:
    mov FIN, 1           ; Terminar el juego si la entrada es inválida
    ret

SELECCION_VALIDA:
    ; Ingresar segunda seleccion
    mov dx, offset MSG_SELECCION
    call mostrarmsg
    call ingresar
    sub al, '0'          ; Convertir carácter a número
    cmp al, 0
    jb INVALIDO2
    cmp al, MAX_CARTAS - 1
    ja INVALIDO2
    mov SELECCION2, al
    ret

INVALIDO2:
    mov FIN, 1           ; Terminar el juego si la entrada es inválida
    ret
ingresarSeleccion endp

compararSeleccion proc near
    ; Comparar las dos selecciones
    mov al, SELECCION1
    mov bl, SELECCION2
    cmp al, bl
    je NO_PAR          ; Si las selecciones son iguales, no es un par válido

    ; Obtener cartas seleccionadas
    mov si, offset CARTAS
    mov dl, [si + SELECCION1]
    mov dh, [si + SELECCION2]
    cmp dl, dh
    jne NO_PAR

    ; Si son par, mostrar cartas
    mov si, offset MOSTRAR_CARTAS
    mov [si + SELECCION1], dl
    mov [si + SELECCION2], dh
    inc ENCONTRADAS
    jmp FIN_COMPARAR

NO_PAR: call quitarvida

FIN_COMPARAR: ret
compararSeleccion endp

verificarFin proc near
    cmp ENCONTRADAS, MAX_CARTAS / 2
    je JUEGO_TERMINADO
    ret

JUEGO_TERMINADO: mov FIN, 1
                 ret
verificarFin endp

mostrarGano proc near
    mov dx, offset MSG_GANO
    call mostrarmsg
    ret
mostrarGano endp

mostrarPerdio proc near
    mov dx, offset MSG_PERDIO
    call mostrarmsg
    ret
mostrarPerdio endp

; Mensajes y variables
TITULO db "Memorama", 13, 10, "$"
MSG_SELECCION db "Seleccione una carta (0-7): $"
MSG_GANO db "Felicitaciones, has ganado!$", 13, 10, "$"
MSG_PERDIO db "Perdiste, intente de nuevo.$", 13, 10, "$"
MSG_OCULTA db "Ingresa una palabra que no tenga numeros o caracteres especial y presiona enter.", 13, 10, "$"
MSG_ADIVINE db "Adivina la palabra!", 13, 10, "$"
MSG_INVALIDO db "Juego finalizado: La palabra contenia numeros o caracteres no validos.", 13, 10, "$"
MSG_VIDAS db "Vidas restantes: ", "$"
MAX_LARGO equ 10 ; Ajusta el valor según sea necesario
SECRETA db MAX_LARGO+1 dup ('$')
ADIVINO db MAX_LARGO+1 dup ('$')
LARGO_PALABRA db ? ; el largo de la palabra ingresada
CHAR db ?

; Subrutinas de entrada/salida y otros

setinicio proc near
    mov FILA, 0
    mov COLUMNA, 0
    call setcursor
    ret
setinicio endp

ingresar proc near
    push cx
    push di
    mov cl, MAX_LARGO
    mov di, offset SECRETA
    mov ch, 0
IC: mov ah, 07h
    int 21h
    cmp cl, ch
    je FI
    cmp al, 13
    je FI
    call convertir_a_mayuscula
    cmp al, 'A'
    jb INVALIDO
    cmp al, 'Z'
    ja INVALIDO
    mov [di], al
    inc di
    inc ch
    jmp IC
INVALIDO:
    call finalizar
FI: inc di
    mov [di], '$'
    mov LARGO_PALABRA, ch
    pop di
    pop cx
    ret
ingresar endp

crearoculta proc near
    push di
    push cx
    mov ch, 0
    mov cl, LARGO_PALABRA
    mov di, offset ADIVINO
CC: cmp cx, 0
    je FC
    mov [di], 'X'
    dec cx
    inc di
    jmp CC
FC: inc di
    mov [di], '$'
    pop cx
    pop di
    ret
crearoculta endp

ingresarletra proc near
    push ax
    mov ah, 07h
    int 21h
    call convertir_a_mayuscula
    mov CHAR, al
    pop ax
    ret
ingresarletra endp

convertir_a_mayuscula proc near
    cmp al, 'a'
    jb FIN_CONVERTIR
    cmp al, 'z'
    ja FIN_CONVERTIR
    sub al, 32
FIN_CONVERTIR:
    ret
convertir_a_mayuscula endp

comparar proc near
    push ax
    push di
    push si
    mov cx, 0
    mov di, offset SECRETA
    mov si, offset ADIVINO
CCMP: mov ah, [di]
      mov al, [si]
      cmp ah, '$'
      je FCMP
      cmp ah, al
      je CSIG
      jmp CPRO
CSIG: inc di
      inc si
      jmp CCMP
CPRO: mov cx, 1
FCMP: pop si
      pop di
      pop ax
      ret
comparar endp

reemplazar proc near
    push si
    push di
    push ax
    mov cx, 0
    mov si, offset ADIVINO
    mov di, offset SECRETA
    mov al, CHAR
CR: cmp [di], '$'
    je FR
    cmp [di], al
    je CPR
    inc di
    inc si
    jmp CR
CPR: cmp [si], 'X'
    je RMP
    mov cx, 1
    jmp FR
RMP: mov [si], al
    inc si
    inc di
    jmp CR
FR: pop ax
    pop di
    pop si
    ret
reemplazar endp

buscar proc near
    push di
    push ax
    mov cx, 0
    mov di, offset SECRETA
    mov al, CHAR
CB: cmp [di], '$'
    je FB
    cmp [di], al
    je PB
    inc di
    jmp CB
PB: mov cx, 1
FB: pop ax
    pop di
    ret
buscar endp 

mostrarvidas proc near
    push ax
    push dx
    push cx
    mov al, vidas
    add al, '0'
    mov ah, 02h
    mov dl, al
    int 21h
    mov dl, ' '
    int 21h
    mov dx, offset MSG_VIDAS
    call mostrarmsg
    pop cx
    pop dx
    pop ax
    ret
mostrarvidas endp

quitarvida proc near
    dec vidas
    call mostrarvidas
    cmp vidas, 0
    je FMP
    ret
quitarvida endp

mostrarmsg proc near
    push ax
    mov ah, 09h
    int 21h
    pop ax
    ret
mostrarmsg endp

mostrarchar proc near
    push ax
    push dx
    mov dl, CHAR
    mov ah, 02h
    int 21h
    pop dx
    pop ax
    ret
mostrarchar endp

setcursor proc near
    push dx
    push bx
    push ax
    mov dh, FILA
    mov dl, COLUMNA
    mov bh, 0
    mov ah, 02h
    int 10h
    pop ax
    pop bx
    pop dx
    ret
setcursor endp

saltarlinea proc near
    add FILA, 1
    mov COLUMNA, 0
    call setcursor
    ret
saltarlinea endp

finalizar proc near
    add FILA, 2
    call setcursor
    mov dx, offset MSG_INVALIDO
    call mostrarmsg
    mov ah, 4Ch
    int 21h
    ret
finalizar endp

; Aquí incluirías las subrutinas de mostrarmsg, ingresar, setinicio, etc.

; Subrutina para mostrar un mensaje

; Subrutina para ingresar un caracter
